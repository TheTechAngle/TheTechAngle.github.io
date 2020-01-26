---
layout: post
title: Google's Bigtable Design Condensed 
---
Source: https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf

The first thing that jumps out at you when you read the paper is bigtable's flexibility in terms of the data size and latency requirements it supports.
It handles data from web indexing, Google Finance and Google Earth, from urls to images. (And that Orkut was a Google product, who knew)

### Datamodel

"A Bigtable is a sparse, distributed, persistent multidimensional sorted map". It is indexed by a row key, a column key and a timestamp, and the value stored in stored as an 
array of bytes. As you can tell, this will model accounts for much of its storage flexibility in terms of data types and sizes.

#### Row
Each update to a row is atomic, regardless of the number of columns. Rows are stored lexicographically, and can be partitioned accordingly,
based on how the client wants to distribute data. Each unit of data so partitioned is called a tablet. So far, so good, we've read similar models 
in dynamo and cassandra.

#### Column Families
Column keys are grouped into sets called column families, which form the basic unit of access control. All data
stored in a column family is usually of the same type

#### Timestamp
Each cell in a Bigtable can contain multiple versions of the same data; these versions are indexed by timestamp.
Bigtable timestamps are 64-bit integers. Applications that need to avoid collisions
must generate unique timestamps themselves.

#### API
It has functions for creating and deleting tables and column families. It also provides functions for changing the 
cluster, table and family metadata.

It supports single-row transactions, does not support functions across row keys.
It allows cells to be used as integer counters.
It allows supports the execution of client-supplied scripts

### Building Blocks
The Google SSTable file format is used internally to store BigTable data. An SSTable provides a persistent,
ordered immutable map from keys to values, where both keys and values are arbitrary bytes strings.

Bigtable relies on a highly-available and persistent distributed lock service called Chubby. A Chubby service consists of five active replicas, one of which is
elected to be the master and actively serve requests. The service is live when a majority of the replicas are running
and can communicate with each other. Chubby uses the Paxos algorithm to keep its replicas consistent in
the face of failure.

Chubby provides a namespace that consists of directories and small files. Each directory or
file can be used as a lock, and reads and writes to a file are atomic.

I read up on Chubby [here|https://medium.com/coinmonks/chubby-a-centralized-lock-service-for-distributed-applications-390571273052]
It was a really interesting read, about why the decision was made to use a locking system like Chubby over
a consensus based set up like Paxos.

### Implementation 
The BigTable implementation has three major components, a library linked to every client, one master server and 
many tablet servers. 

Tablet servers can be dynamically added (or removed) from a cluster to accomodate changes in workloads.
The master is responsible for assigning tablets to tablet servers, detecting the addition and expiration of tablet
servers, balancing tablet-server load, and garbage collection of files in GFS. In addition, it handles sche

The tablet server handles read and write requests to the tablets that it has loaded, and also splits
tablets that have grown too large.

Clients communicate directly with tablet servers for reads and writes. Because Bigtable clients do not rely on
the master for tablet location information, most clients never communicate with the master.

A Bigtable cluster stores a number of tables. Each table consists of a set of tablets, and each tablet contains
all data associated with a row range. Initially, each table consists of just one tablet. As a table grows, it is automatically split into multiple tablets, each approximately
100-200 MB in size by default

### Tablet Location

A three-level hierarchy is used to store tablet location information.

The first level is a file stored in Chubby that contains the location of the root tablet. The root tablet contains
the location of all tablets in a special metadata table. Each metadata tablet contains the location of a set of
user tablets. The root tablet is just the first tablet in the metadata table, but is treated specially—it is never
split—to ensure that the tablet location hierarchy has no more than three levels.

* The first level is a file stored in Chubby that contains the location of the root tablet.
* The root tablet contains the location of all tablets in a special METADATA table. 
* Each METADATA tablet contains the location of a set of user tablets. 
* The root tablet is just the first tablet in the METADATA table, but is treated specially—it is never
split—to ensure that the tablet location hierarchy has no more than three levels.
* The client library caches tablet locations. If the client does not know the location of a tablet, 
or if it discovers that cached location information is incorrect, then it recursively moves up the tablet location hierarchy

### Tablet Assignment

* Each tablet is assigned to one tablet server at a time
* When a tablet is unassigned, the master assigns the tablet by sending a tablet load request to the tablet server.
* Chubby is used to keep track of which tablet server owns which tablet, by requiring that the tablet server gain an exclusive lock on a uniquely named file in a specific directory
* The master monitors this directory to discover tablet servers
* If the tablet server loses its lock, it tries to reacquire it and kills itself if it cannot find the file
* To detect when a tablet server is dead, the master periodically asks each tablet server for the status of its
lock. If a tablet server reports that it has lost its lock, or if the master was unable to reach a server during its
last several attempts, the master attempts to acquire an exclusive lock on the server’s file. If the master is able to
acquire the lock, then Chubby is live and the tablet server is either dead or having trouble reaching Chubby, so the
master ensuresthat the tabletserver can neverserve again by deleting its server file. Once a server’s file has been
deleted, the master can move all the tablets that were previously assigned to that server into the set of unassigned
tablets
* The master kills itself if its Chubby session expires

The master executes the following steps at startup. 
(1) The master grabs a unique master lock in Chubby, which prevents concurrent master instantiations. 
(2) The master scans the servers directory in Chubby to find the live servers.
(3) The master communicates with every live tablet server to discover what tablets are already assigned to
each server. 
(4) The master scans the METADATA table to learn the set of tablets. (After scanning the root table to discover all the METADATA tablets)
