
---
layout: post
title: Cassandra's Design Condensed 
---

[Cassandra](https://www.cs.cornell.edu/projects/ladis2009/papers/lakshman-ladis2009.pdf) is a popular decentralized distributed key value store, designed for write heavy workloads. Cassandra was designed at Facebook to meet its needs for reliability and scalalbility in the Inbox Search problem. 

It was super intersting researching this as I had previosly used the Cassandra Database for aproject of mine back in undergrad.

### Data Model
A table in Cassandra is a multidimensional map indexed by a key. Every operation under a single row key is atomic per replica. Cassandra groups its columns into sets called column families. A Super Column family consists of columns within columns. 

The API to access the database consists of three simple methods - insert, get, and delete. All of these take the table name, and the key. Insert takes the row mutation as the last parameter, while get and delete takes the columnName as the last parameter. 

Typically a read or write request for a key gets routed to any node in the cluster, the node then determines the replicas for the key and then waits for the responses from all the replicas.

### Partitioning 

Cassandra partitions data across clusters using consistent hashing. As we have seen before in the case of DynamoDB, the basic algorithm for this has two problems - 
1) A node being randomly assigned in the ring leads to uneven data distribution and load balancing
2) It doesn't address the heterogeneity of the performances of the nodes

Dynamo fixes this by assign a node to multiple positions in the node. Cassandra analyses load information in the ring and moves lightly loaded nodes move to alleviate the burden on the hevily loaded nodes.

### Replication

Each data item is replicated on N hosts. Each key has a coordinator host, , and it replicates these keys at the N-1 nodes in the ring. Cassandra has various replication policies - 
1) Rack unaware - the N-1 replicas are the N-1 successors of the coordinator
2) Rack aware - A leader is elected amongst the nodes using Zookeeper. The leader then tells the nodes which ranges they are replicas for. Zookeeper stores this in the form of persistent metadata in case of the node crashing.

### Membership and Failure Detection

Membership is based on Scuttlebutt, which makes very efficient utilization of the CPU and the gossip channel.
Cassandra uses a modified version on the Accrual Failure Detector (discussed in an upcoming article)

### Bootstrapping

When a node joins the cluster for the first time, it is randomly assgned a position in the ring. This information is then gossiped around the cluster by the seed nodes. An adminisitrator can use a CLT to connect to a Cassandra node and issue a membership change to leave or join a cluster

### Scaling the Cluster

A node that joins the cluster is given a token so that it decreases the load on another. Data is transferred using a kernel to kernel copy technique. Future work including making this faster by having multiple replicas take place in the transfer similar to Bittorrent.





