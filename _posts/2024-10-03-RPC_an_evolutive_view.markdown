---
title:  "RPC: An Evolutive View"
date:   2024-10-03 17:12:35 +0000
---

The other day I was working with a large repo holding inter-dependant protobufer definitions and a though cross my mind:
> How many times did we solve RPC before?
followed by
> I wonder what was the killer feature that deprecats the old one in favor for the next
so here I am trying to answer those.

## Remote Procedure Call

The term appeared alongside the first versions of internet in the 60s when the _"remote"_ part start to be more sounding given the apearance
of computer networks. Before that the concept of _calling something outside_ was always limited to the current machine and was refered as 
Inter Process Communication (IPC) which is still a concept today althought not as decupled of RPC as before.

According to my investigation (wikipedia) the first implementation of request-response system was release in 1969 as part of the 
"RC 4000 multiprogramming system" and was an asyncronouse IPC API that worked much like how the CPU work by using a shared memory space 
that the involved parts use as a mean of comunication.

P1: I'll put this here for P2
P1: Hey P2 check this out
P2: Oh cool
P2: Got it, thanks

For the purpose of this analysis I'll define RPC as:
> An API that allows to execute a specific subroutine from a known list on a remote host machine.

## Solutions to RPC so far

These are the traces I found

1970 - Host-Host Protocol https://dl.acm.org/doi/pdf/10.1145/1499799.1499878
    It's basically IPC over the netowrk were each each "serve" proces provides specific functions over a specific packet format. The RPC stack here would be
    just the sockets API and manually crafting the structs to be send/recevied over the wire.

    This was the predominant solution for a good time.

-> Make it work

1982 - The Newcastel Connection (Unix United) https://web.archive.org/web/20160816184205/http://www.cs.ncl.ac.uk/research/pubs/articles/papers/399.pdf
    This was the first "one approach to fit all use cases" implementation

    It was a user-space solution that connects Unix machines by building a common directory structure between them. It has a main component called "spawner"
    that acts a as server handling all file related requests, clients were issue RPC calls to it when required remote access to a file. The comminication
    API is not documented and the software was comercialy licensed and the owner might not exist anymore, but in any case is safe to assume it uses a 

    So... given some of the high level description found it seems it just used raw TCP/IP messages implementing a custom protocol. I think this alighs pretty
    closely to what would imagine will be the most simple approach to RPC: just send a value over the wire that encode some procedure that the other side would
    understand.

    P1: (throws a rock with a paper wrapped around to P2)
    P2: Hau! (look back, paper say "how old is your mather?")
    P2: (writes "76, why?" in the same paper)
    P2: (throws the rock back)
    ...etc

-> Make it work for any of the known tools (mv, cp, tail, etc)

1984 - Implementing Remote Procedure Calls https://www.cs.cmu.edu/~dga/15-712/F07/papers/birrell842.pdf
    It was part of the Cedar project at Xerox PARC, which among other things itroduced the first: mouse, tiling window manager, icons, etc...
    The primary purpose of this project was to make distributed computation easy and they did so by providing a code generation tool called Lupine https://xeroxparcarchive.computerhistory.org/cyan/cedar6.0/rpc/.LupineUsersGuideP1.bravo!1.html
    that would produce stubs for the client and server side given an module interface defined using the Mesa programming language which was the
    main language used supported by the Cedar OS.

    Mesa was a precuros of Modula-2 so you can imaging it was a pretty high level language... We call this kind of approach "modern" today 40 years later

    https://youtu.be/
    https://worrydream.com/refs/Teitelman_1984_-_The_Cedar_Programming_Environment,_A_Midterm_Report_and_Examination.pdf


-> Make it work for any tool (that can be expressed in Mesa pl)

1990 - Distributed Computing Environment (DCE) https://en.wikipedia.org/wiki/Distributed_Computing_Environment
    At this point the internet was starting to explode, any of the known approaches didn't fit the bill and interconectivity was becomming a business blocker in
    the eyes of those pumpingh the .com bubble. So a group of important companies pushed DCE to become a standard which covered many aspects of distributed computing
    like time sync, auth and a distributed file system which make it way more than just an RPC solutions.

    In that regard, the objective was to abstract away the fact that a call was being executed in a remote location which in my view is generally a bad idea,
    more so given that the objective of the solution was to be so generic that could fit any problem.
    
    Fun fact: this project was the first implementaiton of UUIDs

https://archive.org/details/networkworld646unse/page/44/mode/1up


-> Make it work for any tool on any OS for the forseable future

(Thins start to go fast from this point forward, I picked some years to flatten events in a line but they are fairly overlapped)

mid 90s -> Common Object Request Broker Architecture (CORBA)
    I remember tripping the first time I read about CORBA, I was learning C++ and the idea of being able to distribute a program across many machines
    at an object level was just mind blowing to me... any who...

    CORBA works pretty similar to Lupine, except it has a specific lanuage for descreibing the interaces (I'm looking at you Protobuf) and generates code
    for multiple langues (I'm looking at you again...). The bigest issues with CORBA was that it not only cares about the RPC part but also all the
    tangential issues that comes with it: remote object lifecycles, timeouts, network partitions, load balancing, etc... which place all the burthen
    of handling this issues in the application.

    It also being criticized for have an expesive license fee.

-> Make just RPC work for any tool on any OS using any develelopment platform for the forseeable future and become rich in the process

2000 - XML-RPC and SOAP
    While all this happens the e-comerse on the web was exploding with much modest requirements to those of an infinite distributed system
    so solutions like DCE, CORBA or JAVA RMI where expensive across every access: time, money and complexity. Microsoft read this successfully and intriduced 
    XML-RPC into Internet Explorer 5 primarly to use it as a way to support Exchange email client in the browser but it was quickly started to get exploited
    by other sites to make requests to their servers over HTTP.

    SOAP was defined as the usage cases arond XML-RPC starting to satle down as patters, strictly speacking SOAP is no more than a specification over XML messages 
    format which in conjution with the backed-in support in IE 5 JavaScript runtime (XMLHTTPRequest) became a defacto standard stearing the direction of future
    RPC solutions to come.

-> Make it work over HTTP (rolling back all previous efforts)

2005 - Web Services Description Languages (WSDL)
    With the adacend of the web there was still the business need to further interconect providers to wider the offers to users, and allow companies to take
    their slice of the pie. To address this Microsoft and IBM (again, smart move) re-introduced the old idea about having an independant desciption of a services
    that can be shared or even automatically consumed.

    This effort introduced the concept of "web services" and a "web serices description language" to define them alongside tools for generate code given a
    WSDL specification. This played well with the wide adoption of what it was now call Ajax (aka: XMLHTTPRequest) in all major web browsers.

-> Make it work over HTTP in all web browsers and development plafroms

2010 - APIs over HTTP (aka "RESTful")
    As the web continue to expand to every corner of our lives teams needed to move faster and WSDL and SOAP was starting to become too strict and rigid
    for the modern web. It required much ceremonies, contract between the browser and servers where too strict, it didn't play well with the browser chache
    control, it requires dealing with XML which compared to jSON was bit PITA, etc.

    All these pushed SOAP to a corner where it still is to this day, and left us (the people building this stuff) with a less painful solution but still far
    less sofisticated than the tools available in the 80s. As usual with web technologies they get quickly over exploited so we will spend the following then
    years or so seding jSON payloads using XMLHTTPRequest (XHR in your broweser debugger) objects that we called Ajax.

    [meme of the guy in the party with the hat "they don't know"]

-> Make it less of a hack and still working over HTTP and keeping compatibility

2011 - Swagger and Open API
    As projects start to settle the need for some control over the evolution and development of APIs was required, so the concept of "API definition" was
    re introduced for the third time. This time instead of being a coordinated effort it was much more organic (just a mess), one guy did it because it
    make sense for his team and a couple of years forward it become the Open API Specification we know about (at web speed).

    And it's good! we have some specification now that we can use to evolve, share, test and even generated stubs like Lupine did in the past.

-> Make it so we can evolve APIs with some control but not as strict as WSDL, and probably build some tooling around it.

2015 - GraphQL
    Data started to grow and so it did the power of the client machines, the need form consume large amount of data at the user fingertips was in a step acending
    curve so Facebook invented GraphQL to help clients ask for specific parts of those giant datasets.

    Not much was done on the RPC side, this was just a layer over the well stabished _API over HTTP_ pattern. It worth mentioning that it also introduced
    streaming updates to a specific part of the dataset to the clients using web-sockets, which was a cool thing at this point int time.

-> Tailor the RPC problem to data fetching and manipulation.

2016 - gRPC and Protobuf
    Google was already a giant at this stage and had to start dealing with requirements only dreamed by the early versions of ARPANET, so to cope with that
    they created an efficient low-level protocol, Procol Buffers, and a RPC framework around it, gRPC.

    It provides a fairly complete declarative language that allows to compose multiple declarations and a compiler which can generate code stubs for both ends
    of the comunication plus. Pluse the low-level format had backgward and forward compatibility backed-in supporting adding and removing of fields without swet.
    BUT... it doesn't work in the browser https://grpc.io/blog/state-of-grpc-web/ because it require low-level access to the request and responses and those APIs
    doesn't exist yet.

    []

-> Add all the old features agian: code generation for client and server, works on any platform, declarative API defintions, etc But! don't work in the browser

And that's the last important breakthrow in RPC solutions, we are still using APIs over raw HTTP without much evolving around it. Given that the final destination
of most (if not all) of our process are expected to be a web browser we are stuck with HTTP 

## Conclustions

- REST seems to be the real step-forward in RPC (real REST, not just use POST for inserts and PATCH for updates)
