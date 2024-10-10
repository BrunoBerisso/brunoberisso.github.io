---
title:  "RPC: An Evolutive View"
date:   2024-10-03 17:12:35 +0000
---

The other day I was working with a large repo holding inter-dependant protobufer definitions and a though cross my mind:

> How many times did we (as an indutrie) solve RPC before?

followed by

> I wonder what was the killer feature that deprecats the each old approach in favor for the new thing

so here I am trying to answer those.

## Remote Procedure Call

The term appeared alongside the first versions of the internet in the 60s when the _"remote"_ part of the game started to be more sounding 
given the apearance of computer networks. Before that the concept of _calling something outside_ was always limited to the current machine and was relegated to 
Inter Process Communication (IPC) which is still a concept today althought not as decupled of RPC as before.

According to my investigation (wikipedia) the first implementation of request-response system was release in 1969 as part of the 
"RC 4000 multiprogramming system" and was an asyncronouse IPC API that worked much like how the CPU works by using a shared memory space 
that the involved parts use as a means of comunication.

    Process 1: I'll put this here for Process 2
    Process 1: Hey P2 check this out
    Process 2: Oh cool
    Process 2: Got it, thanks

For the purpose of this analysis I'll define RPC as:
> An API that allows to execute one specific subroutine from a previously defined set of available subrutines on a remote machine.

## Solutions to RPC so far

These are the traces I found

### [1970 - Host-Host Protocol](https://dl.acm.org/doi/pdf/10.1145/1499799.1499878)
It's basically IPC over the netowrk were each "serve" proces provides specific functions over a specific packet format. The RPC stack here would be
just the sockets API and manually crafting the structs to be send/recevied over the wire, meaning that each application will need to define it's
own protocol.

This was the predominant solution for a good time.

> Make it work

### [1982 - The Newcastel Connection (Unix United)](https://web.archive.org/web/20160816184205/http://www.cs.ncl.ac.uk/research/pubs/articles/papers/399.pdf)
This was the first "one approach to fit all" implementation

It was a user-space solution that connects Unix machines by building a common directory structure between them, the implementation was done as a patch to
libc that catches all file access, if the file was in a remote location it would make the appropiate RPC calls to a process called "spawner" that serve the
files to the clientes. 

The communication API is not documented and the software was comercialy licensed by an owner that might not exist anymore, but in any case is safe to assume it uses the
prediminant approach of manually crafting your protocol over C sockets. I think this alighs pretty closely to what I would imagine will be the most simple approach to
RPC: just send a value over the wire that encode some procedure that the other side would understand.

    Process 1: (throws a rock with a paper wrapped around to Process2)
    Process 2: Hau! (look back, unwrapp the paper, it say "how old is your mather?")
    Process 2: (writes "76, why?" in the same paper)
    Process 2: (throws the rock back)
    ...etc

> Make it work for any of the known tools (mv, cp, tail, etc)

### [1984 - Implementing Remote Procedure Calls](https://www.cs.cmu.edu/~dga/15-712/F07/papers/birrell842.pdf)
It was part of the Cedar project at Xerox PARC, which among other things itroduced the first: mouse, tiling window manager, icons, etc...
The primary purpose of this project was to make distributed computation easy and they did so by providing a code generation tool called [Lupine](https://xeroxparcarchive.computerhistory.org/cyan/cedar6.0/rpc/.LupineUsersGuideP1.bravo!1.html) that would produce stubs for the client and server side given an module interface defined
using the Mesa programming language which was the main language used supported by the Cedar OS.

Mesa was a predecesor of Modula-2 (which all Fing students _loved_ for the first two years) so you can imaging it was a pretty high level language.
We call this kind of approach "modern" today 40 years later

Check these links for getting a sence of the advance Cedar was for it era:
- This is [Eric Bier doing a demo](https://youtu.be/z_dt7NG38V4) for the computer museum, was an researcher using the platform.
- Bret Victor has a [report of the system among their references](https://worrydream.com/refs/Teitelman_1984_-_The_Cedar_Programming_Environment,_A_Midterm_Report_and_Examination.pdf). If you don't know who he is, [go check it out](https://worrydream.com/)  


> Make it work for any tool (that can be expressed in Mesa pl)

### [1990 - Distributed Computing Environment (DCE)](https://en.wikipedia.org/wiki/Distributed_Computing_Environment)
At this point the internet was starting to leak to other areas outside of the labs, any of the known approaches didn't fit the bill and interconectivity was 
becomming a business blocker in the eyes of those pumpingh the .com bubble. So a group of important companies pushed DCE to become a standard which covered
many aspects of distributed computing like time sync, auth and a distributed file system which make it way more than just an RPC solutions.

    millioner 1: hey I saw you are making mony with that e-commerce thing, can I sell my stuff there?
    millioner 2: sure! can you sent me your stock articles, some photos, prices
    millioner 1: ah... yeah... that might take me ages... Can a computer do that for me?
    millioner 3: off course!
    millioner 1: ?!
    millioner 2: (who is this nerd?)
    millioner 3: but first, lets dive into the intricancies of time sync over an infinite network of computers shall we?
    etc...

In regard to RPC, Appollo Computer Inc. was the one providing the implementation with their NCS (Netowrk Computing System) which has all the features of the previous approach
pluse a location broker that allow to decouple services names from their phisical location.

Network World, a newspaper dedicated to "networking startegies" [posted this in 1989](https://archive.org/details/networkworld646unse/page/44/mode/1up)
mentioning Apollo's NCS among others:
![Network World newspaper, 1989](/assets/images/rpc_1.png)

Fun fact: this project was the first implementaiton of UUIDs

> Make it work for any tool on any OS for the forseable future

_(Things start to go fast from this point forward, I picked some years to flatten events in a line but they are fairly overlapped)_

### Mid 90s -> Common Object Request Broker Architecture (CORBA)
I remember tripping the first time I read about CORBA, I was learning C++ and the idea of being able to distribute a program across many machines
at an object level was just mind blowing to me... any how...

CORBA works pretty similar to Lupine, except it has a specific lanuage for descreibing the interaces (I'm looking at you Protobuf) and generates code
for multiple langues (I'm looking at you again...). The bigest issues with CORBA was that it not only cares about the RPC part but also all the
tangential issues that comes with it: remote object lifecycles, timeouts, network partitions, load balancing, etc... and place all the burthen
of handling this issues in the application developer. That's why it has that bad fame of "being difficult"

It also being criticized for have an expesive license fee.

> Make just RPC work for any tool on any OS using any develelopment platform for the forseeable future and become rich in the process

### 2000 - XML-RPC and SOAP
While all this happens the e-comerse on the web was exploding with much modest requirements to those of an infinite distributed system,
solutions like DCE, CORBA or JAVA RMI where expensive across every access: time, money and complexity. Microsoft read the situation perfectly and intriduced 
XML-RPC into Internet Explorer 5 primarly to use it as a way to support Exchange email client in the browser but it was quickly started to get over exploited
by other sites to make requests to their servers over HTTP.

SOAP was defined as the usage cases arond XML-RPC starting to satle down as patters, strictly speacking SOAP is no more than a specification over XML messages 
format which in conjution with the backed-in support in IE 5 JavaScript runtime (XMLHTTPRequest) became a defacto standard stearing the direction of future
RPC solutions to come.

> Make it work over HTTP (rolling back all previous efforts)

### 2005 - Web Services Description Languages (WSDL)
With the adacend of the web there was still the business need to further interconect providers to wider the offer to users, and allow companies to take
their slice of the pie. To address this Microsoft and IBM (again, smart move) re-introduced the old idea about having an independant desciption of a services
that can be shared or even automatically consumed.

This effort introduced the concept of "web services" and a "web serices description language" to define them alongside tools for generate code given a
WSDL specification. This played well with the wide adoption of what it was now call Ajax (aka: XMLHTTPRequest) in all major web browsers.

> Make it work over HTTP in all web browsers and development plafroms

### 2010 - APIs over HTTP (aka "RESTful")
As the web continue to expand to every corner of our lives teams needed to move faster and WSDL and SOAP was starting to become too rigid for the modern
web. It required much ceremonies, contract between the browser and servers where too strict, it didn't play well with the browser chache
control, it requires dealing with XML which compared to JSON was bit PITA, etc.

REST (Representatilnal State Transfer) is a high level software architecture style that goes way deeper than _"use the right HTTP verbs for your APIs"_,
having say that this simple interpretation seems to be the most important contribution it has done to the industry so far.

All these pushed SOAP to a corner where it still is to this day, and left us (the people building this stuff) with a less painful solution but still far
less sofisticated than the tools available in the 80s. As usual with web technologies they get quickly over exploited so we will spend the following then
years or so seding JSON payloads using XMLHTTPRequest (XHR in your broweser debugger) objects.

![WDSL is still in the party](/assets/images/rpc_2.png){: width="40%" }

> Make it less of a hack and still working over HTTP and keeping compatibility

### 2011 - Swagger and Open API
As projects start to settle the need for some control over the evolution and development of APIs was required, so the concept of "API definition" was
re introduced for the third time. This time instead of being a coordinated effort it was much more organic (just a mess), one guy did it because it
make sense for his team and a couple of years forward it become the Open API Specification we know about (at web speed).

And it's good! we have some specification now that we can use to evolve, share, test and even generated stubs like Lupine did in the past.

> Make it so we can evolve APIs with some control but not as strict as WSDL, and probably build some tooling around it.

### 2015 - GraphQL
Data started to grow and so it did the power of the client machines, the need form consume large amount of data at the user fingertips was in a steep acending
curve so Facebook invented GraphQL to help clients ask for specific parts of those giant datasets.

Not much was done on the RPC side, this was just a layer over the well stabished _API over HTTP_ pattern. It worth mentioning that it also introduced
streaming updates to a specific part of the dataset to the clients using web-sockets, which was a _the cool thing_ at this point in time.

> Tailor the RPC problem to data fetching and manipulation.

### 2016 - gRPC and Protobuf
Google was already a giant at this stage and had to start dealing with requirements only dreamed by the early versions of ARPANET, so to cope with that
they created an efficient low-level protocol, Procol Buffers, and a RPC framework around it, gRPC.

It provides a fairly complete declarative language that allows to compose multiple declarations and a compiler which can generate code stubs for both ends
of the comunication. Pluse the low-level format had backgward and forward compatibility backed-in supporting adding and removing of fields without sweat.
BUT... [it doesn't work in the browser](https://grpc.io/blog/state-of-grpc-web/) because it require low-level access to the request and responses and those APIs
doesn't exist yet.

> Add all the old features agian: code generation for client and server, works on any platform, declarative API defintions, etc But! don't work in the browser

## Conclustions

And that's the last important breakthrow in RPC solutions to this day, we are still using APIs over raw HTTP without much evolving around it. Given that the final destination
of most (if not all) of our process are expected to be a web browser and the amount of effort, time, ane money invested in all the machinery around it it's hard to imagine
anything new outside of the 

As for our initial definition of RPC it seems that we get it right in the first couple of iterations, at the time of the Cedar project (1984) we have all the features done.
From that point forward it was just a matter to adapt to the explosion of internet and dealing with the web, but the core features where there:
- typed interfases
- code generation
- formal documentation

Given the amount of smart people dealing with this problem for so long I think we might done with it, there is no much juice to get out of this rock. I think the next step
forward would be push the wold concept of "remote computation" one level deeper into our tools and instead of "make RPC calls as-is they where local" it might be "make our
software naturally distributed". The only serious effort in this direction that I know about is [Unison](https://www.unison-lang.org/), which has being in my radar scince their first appear in Strange
Loop many years ago. I'll be writing about it more here for sure.
