---
layout: post
title: Appendix
---

### Atomicity 
A transaction is said to be atomic if it is indivisible, or made up of a series of operations such that either they all take place or none do.

### Availabilty 
The availability of the system is essentially the uptime of the system, i.e. when it is running and operational. 

### CAP Theorem 
Consistency | Availabilty | Partition Tolerance
Can't have all three. Partition Tolerance always happens, so essentially choosing between consistency & availabilty.
Traditional DBs rely on ACID, so Consistency wins over Availabilty.
NoSql DBs can choose Availabilty over Consistency.

### Concurrent Transactions
When transacations can take place as if in parallel, but in reality might take place out of order or in partial order, without affecting the final outcome.

### Consistency
Many times, data will be replicated across nodes. The consistency of a system is a way of stating how quickly and accurately the data is updated across the nodes such that it is consistent across all the nodes.See also strong consistency and eventual consistency.
See [here](https://mwhittaker.github.io/consistency_in_distributed_systems/1_baseball.html) for a detailed explanation.

### Cross origin resource sharing 
Allows restricted resource on a web page to be requested from another domain outside the domain from which th efirst resource was served. Browser makes an HTTP OPTIONS (GET PUT POST) call for a url, and the server retursn a response saying "These other domains are approved to GET this url"

### Durability
The durability of a system guarantees that data that is committed persists, or is saved permanently.

### Eventual Consistency
A system display eventual consistency when nodes evetually update each other with changes, before which it is possible that clients may see inconsistencies in the data. For example, client C may update system A by changing the value of x from 6 to 7. Since A doesn't immediately update the other nodes about this change, if C asks for the value of x again and the request is routed to system B, it will respond with the old value of x ie 6. If C repeats this request after some time during which the update takes place, it will get the new value of x.

### Indexing
Organizing data for fast retrieval

### Lazy space allocation
Rather than allocating space for the file content as soon as it is created, the data is written onto a buffer first. This improves the chance that the data is written in a contiguous group of blocks, reducing fragmentation problems and increasing performance. 

### Partition Tolerance
When there's a partition between two hosts. Always happens in real life.

### Same origin policy 
A web browser permits scripts contained in a first web page to access data in a second web page only if bothe web pages have the same origin.to prevent Cross-Site Scripting (XSS) attacks.

### Scaling
Vertical Scaling - Add more memory/CPU power. Expensive and limited
Horizontal Scaling - Add more hosts. But have to worry about distribution concepts.

### Serializability 
A transaction schedule is serializable if there exists a schedule where the transactions are executed in some sequence with the same outcome.

### Service Level Agreement (SLA)

An agreement between the the service provider and customers, which documents the terms of the service, and the standards to which the customer can expect to hold the service. 

### Split Brain Situation
During a partition, one section may elect another master even though the old master is actually still alive
in the other partition. To prevent this,the number of masters must never be less than or equal to half the 
total number of nodes, or the number of votes need for electing a master must be at least one more than half .

### Strong Consistency
A system displays strong consistency when it behaves as if it is running on one node, i.e. when all the node immediately update each other for any change such that the client never sees any inconsistencies in the data.
