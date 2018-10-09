---
layout: post
title: Appendix
---

#### Atomicity 
A transaction is said to be atomic if it is indivisible, or made up of a series of operations such that either they all take place or none do.

#### Availabilty 
The availability of the system is essentially the uptime of the system, i.e. when it is running and operational. 

#### Concurrent Transactions
When transacations can take place as if in parallel, but in reality might take place out of order or in partial order, without affecting the final outcome.

#### Consistency
Many times, data will be replicated across nodes. The consistency of a system is a way of stating how quickly and accurately the data is updated across the nodes such that it is consistent across all the nodes.See also strong consistency and eventual consistency.
See [here](https://mwhittaker.github.io/consistency_in_distributed_systems/1_baseball.html) for a detailed explanation.

#### Durability
The durability of a system guarantees that data that is committed persists, or is saved permanently.

#### Eventual Consistency
A system display eventual consistency when nodes evetually update each other with changes, before which it is possible that clients may see inconsistencies in the data. For example, client C may update system A by changing the value of x from 6 to 7. Since A doesn't immediately update the other nodes about this change, if C asks for the value of x again and the request is routed to system B, it will respond with the old value of x ie 6. If C repeats this request after some time during which the update takes place, it will get the new value of x.

#### Lazy space allocation

#### Serializability 
A transaction schedule is serializable if there exists a schedule where the transactions are executed in some sequence with the same outcome.

#### Strong Consistency
A system displays strong consistency when it behaves as if it is running on one node, i.e. when all the node immediately update each other for any change such that the client never sees any inconsistencies in the data.
