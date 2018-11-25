---
layout: post
title: Amazon's DynamoDB Design Condensed 
---

This article is on Amazon's Dynamo database, a highly distributed key-value storage system, designed to be always available.
Here is the paper if you want to read it all in full!
-> [Amazon's DynamoDB](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) <-

If not, here's my summary of the design. (While reading the Amazon paper on DynamoDB, I found out that S3 is a short form for simple storage service. Fun Fact)

### COMPLETELY DECENTRALIZED and SYMMETRICAL

A completely decentralized system, DynamoDB does not have a bottleneck or a single point of failure that is the master node.
Amazon wants its site to be available to customers no matter what. As we know from the CAP theorem for highly scalable systems, as the network keeps growing you risk partitions that divide the system. At this point you have to choose between consistency and availability. 
Amazon prioritizes scalability and availability over consistency. Data is partitioned and replicated using consistent hashing, and consistency is maintained using object versioning, with a quorum like technique and a decentralized replica synchronization protocol. Dynamo has a gossip protocol for failure prevention (This protocol led to a system outage for AmazonS3, [an interesting case](https://status.aws.amazon.com/s3-20080720.html) you should read). "Dynamo can be characterized as a zero-hop DHT, where each node maintains enough routing information locally to route a request to the appropriate node directly, to increase latency."

### Dynamo’s consistency
Writes are NEVER rejected. That's Interesting. So who resolves the conflicts? The datastore could use a last write wins method which, though the default, is not very smart. Instead the application layer is allowed to manipulate it, for example, maybe merge the two writes. 

### Dynamo System Interface
Dynamo has the two quintessential operations we’ve come to know and love - get() and put() Put takes in another parameter apart from the key and object - the context which store metadata related to the caller including the version set of the object.
An MD5 hash on the key gives you the storage node responsible for the object

### Partioning Algorithm 
Follows consistent hashing, with virtual nodes. Each physical node is then responsible for more than one node in the circle. Virtual nodes help in load balancing and can 'account of the heterogeneity in the physical infrastructure'

### Replication 
Dynamo replicates its data on multiple hosts. The key is replicated on N successor nodes, where all the nodes are also distinct physical nodes.

### Data Versioning 
Dynamo uses vector clocks to capture causality between different versions of the same object. (LINK to vector clocks) The servers give a stamp to the data [Server, version]. When the version set list reaches its maximum size, remove the oldest. Could cause problems, but hasn’t yet .. (?)

When the object diverges, both versions have to be presented to the client, and the client can decide how to merge it.
I like that they leave it up to the client, so much more easier

### Execution of get() and put()
One coordinator who manages replicas. Dynamo uses the quorum protocol(LINK) with R - minimum number of nodes required to participate for read, W being minimum for write, where W+R > N, so a read and write don’t take place simultaneously. For get, the coordinator returns all the causally unrelated items.
For a put request, the coordinator generates the vector clock for the new version, writes it locally and attempts to send it to W-1 other nodes. If the all reply with a success code, then the operation is deemed to be successful.

### Handling failures : hinted handoffs
To make sure the the system is always available, the sloppy quorum system is used, where N healthy nodes are considered during the vote, while the actual number of nodes will be greater than N.

Usually, the R+W>N approach makes sure that the data is always eventually consistent, but in case of a sloppy quorum that may not be true, as explained so well [here](https://jimdowney.net/2012/03/05/be-careful-with-sloppy-quorums/). What if during a write operation, a previously dead system comes back online again and participates in a read? The information sent back will be inconsistent.

If a server A is down temporarily, its load is transferred to another service B, which keeps it and updates it in a separate database. This system B may then participate in operations in A's stead. Then when the server A is up again, it attempts to send it back. This hinted hand-off make sure that as soon as the unhealthy system is back up, it is updated and doesn't participate in a read operation with stale operation.


To detect inconsistencies between replicas and to minimize the amount of transferred data, Dynamo uses a Merkle tree. (Really cool, learn and LINK) Disadvantage is that hashes have to be recalculated when a node leaves the system and its data is distributed

### Durability and Availability
We can choose the amout of availability we need by toggling the R and W values. For instance, we can decrease availabilty by increasing W. This may increase the probability of rejecting requests because more storage hosts need to be alive to process a write request. BUT this increases durability because the value is persisted in a greater number of nodes.

### Ring Membership
Gossip protocol propagates changes to a node's membership throughout the system.
Each node contacts a peer chosen at random every second and the two node reconcile their persisted membership change histories
But there might be cases where node A joins the system and node B joins the system. Their membership is propagated to the rest of the system, but the other's presence is not immediately known to the other. To prevent a logical partition, there are seed nodes. 'Seed' nodes are known to all nodes, and new nodes eventually reconcile their membership with a seed, logical partitioning isn’t possible.

### Implementation
"Dynamo’s local persistence component allows for different storage engines to be plugged in."
"Each client request results in the creation of a state machine on the node that received the client request. The state machine contains all the logic for identifying the nodes responsible for a key, sending the requests, waiting for responses, potentially doing retries, processing the replies and packaging the response to the client. Each state machine instance handles exactly one client request."		
"Communication handled by java NIO channels."
"If stale replies are received, it updates those with the latest versions." 	 	 		
"For the sake of brevity the failure handling and retry states are left out."
Communication with Dynamo is through JSON/HTTP protocol, which takes away from its performance
