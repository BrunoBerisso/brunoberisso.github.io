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

1982 - The Newcastel Connection (Unix United) https://web.archive.org/web/20160816184205/http://www.cs.ncl.ac.uk/research/pubs/articles/papers/399.pdf
    It was a user-space solution that connects Unix machines by building a common directory structure between them. It has a main component called "spawner"
    that acts a as server handling all file related requests, clients were issue RPC calls to it when required remote access to a file. The comminication
    API is not documented and the software was comercialy licensed :it's fine:

    So... given some of the high level description found it seems it just used raw TCP/IP messages implementing a custom protocol. I think this alighs pretty
    closely to what would imagine will be the most simple approach to RPC: just send a value over the wire that encode some procedure that the other side would
    understand.

    P1: (throws a rock with a paper wrapped around to P2)
    P2: Hau! (look back, paper say "how old is your mather?")
    P2: (writes "76, why?" in the same paper)
    P2: (throws the rock back)
    ...etc

1984 - Implementing Remote Procedure Calls https://www.cs.cmu.edu/~dga/15-712/F07/papers/birrell842.pdf
    