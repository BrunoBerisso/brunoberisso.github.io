---
title:  "Thought Process: split database schema"
date:   2024-03-15 19:00:35 +0000
---

This post is my thought process while trying to analyze the problem so it will not be a perfect straight piece of text. I have been tasked with the refactoring of a graph database schema supporting a B2B product to make it db-agnostic and split the operational and client-specific data stores. This implies that data stores might be different which means: distributed transactions 😐

Now, as far as I can tell (and search in Google) any solution would be tailored to the specific case, but some concrete questions might be worth asking - answering:
1. How much referential integrity is enough?
2. How often do references become invalid?
3. What life cycles are these references tight to?

_(At this point I already have an implementation in my head that I'm playing with, each of those questions is my rotating this idea and trying to observe it from different angles. If that question sheds some light over an invisible edge the shape of the solution will change to accommodate for that)_

### Enough Referential Integrity

There are a couple of specific features that require cross-database references, some would not suffer much if a remote reference becomes invalid but there is one use case in particular, with many cross-references that will become useless if the reference were invalid. Even more, this feature takes advantage of this by-reference relation to get the latest updates automatically when something changes. So, choosing the maximum common denominator I think we will need to keep it at least _"eventual referential integrity"_ and I mean _eventual_ because it doesn't need to be in the same online transaction. The integrity of the references can be handled in the next couple of minutes and it will be fine.

Answer: need at least eventual consistency in a minute time window

### How Fast Data Become Stale

This is hard to answer, not because it's a hard question but because I can not access any currently deployed system to run some counts... Given there is no real data let's make some safe assumptions: data is loaded once a day, an ETL pipeline will get records from Kafka or something and run some sort of UPSERT, at the end there is a step that checks for stale data to be deleted. Given this scenario the worst case would be that all references become invalid every time the dataset is refreshed, in this situation I think we are good because it means our product would not be usable 😅 so no need to worry about it. On the other side of the spectrum, there is the "no reference is ever deleted" that we would also ignore, so we are left with the 50-50 case for this exercise.

Answer: half the data becomes stale every day

### References Lifecycles On The Operational Database

There are a couple of features that create remote references but the one driving the analysis has a manual lifecycle, meaning that the user creates and destroys the references at will. They can last 5 mins or 5 years, in any case the standard life span of the references is at the order of weeks if not more.

Answer: life-cycles are arbitrary depending on user desire

## Approach

My initial idea is to have a table on the operational datastore side which mirrors the references on the remote database. This means that every fetch to the database will require a remote call, kind of a remote LEFT JOIN. It might be a good idea to cache this record in the same table and evict it on every reload?, probably... what's for sure is that this table will need to know the keys, basically the remote reference, to build an efficient query and because we are talking about one table holding many potential remote references we can't use a fixed set of columns so I'm thinking on a JSON payload.

![remote_ref table schema](/assets/images/remote_ref.png)

### Use Cases

#### 1. Add a remote reference

When the user adds a new remote reference, remember these have user-controller lifecycles, two things can happen: the reference was not in `remote_ref` so we need to add it or it was already there because it was added before as a previous user action. To handle these two cases we can use `remote_hash_key` column, these values are an MD5 hash of the properties of the remote entity that conforms it's key. For example: if I'm trying to add a remote reference to an "Employ" entity from another database I will need to have the "employ_id" for the row I'm interested in, but because `remote_ref` needs to handle any kind of remote entities too we need an abstract representation of that remote key. That's where the hash is useful, with it we can also handle the case of another entity "BankAccount" with a composed key "number" and "owner" so I can hash those two into a single value and use the same table schema.

Now if later the user tries to add the same reference again we can avoid fetching the remote database and just add a row on our side (the operational datastore) pointing to the existing row in `remote_ref`. The step-by-step user action flow would look like this:
1. The user gets a list of the remote entities
2. One entity is selected and a request is made to "save" it into our operational database
3. A backend service inspects the entity, **having some knowledge of the remote schema** it extracts the required fields to build a unique key
3. Compute the MD5 of that key and check if a row exists in `remote_ref` filtering by `remote_hash_key`
4. If there is a row for that hashed key, then the `id` of that row is stored as a foraing key in the final table on the operational database
5. If there is no row for that hashed key, a request is made to the remote database using the entity fields required to build a unique key
6. If the entity is fetched successfully a new row is added to `remote_ref` with the fetched payload stored in `cached_entity`
7. If the fetch fails or no value is returned the "save" process finishes with an error

Did I miss something?

#### 2. (Re)Fetching a remote reference

Now suppose later that day the user navigates to a screen that requires to show the remote reference we saved before, three things can happen: everything is exactly as it was after the "save" action described above, our small cache might be invalidated so `cached_entity` will be empty, or the row is marked as `missing` or `deleted`. Any other case will be a referential inconsistency local to the operational database so it _should not_ happen given that we _should have_ our foreign keys set to catch those beforehand.

> everything is exactly as it was after the "save" action 

If we happen to find our row in `remote_ref` and it's complete we just grab our entity from `cached_entity` column and move on with our life. This first option is the happy path, we all love to live there but nothing kicks you out of it faster than working on backend stuff. More so in these kinds of distributed systems problems, there is no happy path here 😆.

> our small cache might be invalidated

Alright so we found our row which is not marked as `deleted` but `cached_entity` is empty so we need to re-fetch, this is grabbing the content of `remote_key_payload`, construct a query and run it against the remote database. If the re-fetch succeeds we update the value of `cached_entity` and set `missing` to _false_, if it fails we set `missing` to _true_ and report the case to the user... somehow.

> the row is marked as `missing` or `deleted`

If the row is `deleted` there is not much we can do other than let the user know the data is stale and delegate the decision of what to do to the user, **we should keep deleted rows as long as they are being referenced by the operational database**. For the `missing` ones, as described above rows are marked as _missing_ when the values are not cached and not marked as _deleted_ but they can not be re-fetched for some reason. There are many reasons why the fetching might fail and we should not make any assumptions about why that could be the case, ideally we should have a back-off-retry + circuit breaker to try to fetch values again when requested.

#### 3. Delete a remote reference

The foreign key relation to `remote_ref` is _n-1_ which means that over some time we might have orphan rows on `remote_ref` that we should clean up.

#### 4. ETL on the remote database

We assumed [before](#enough-referencial-integrity) that there would be an ETL running every day consolidating the remote dataset we are trying to get a hold of. I would say that for all this to work we will need to have some sort of CDC that pushes us deltas of what happens during the ETL process, more concretely we would need: IDs of the entities that changed and IDs of the entities that were deleted. If we could have that in a Kafka topic (or any other broker) we can hook a consumer that applies the changes to `remote_ref` by evicting cached values and marked rows as `deleted`, this might be a good spot to also do some cleanup of orphan rows in `remote_ref`, probably.

## Conclusion

The final picture looks like this:

![remote_ref table schema](/assets/images/remote_ref_complete.png)

_Note: I removed the `missing` column in favor of checking `entity_cached` IS NULL_

At this point, I'm ready to share this with some of my team members with more context than me on the use cases of our product and probably refine this some more to reach a consensus on the approach and start laying down a plan.