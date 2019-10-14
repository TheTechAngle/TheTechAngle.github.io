
## Database Systems

Two key features

* Multi-AZ - For Disaster Recovery only (Aurora is completely Fault Tolerant doesnt need this). Can change AZ by failover by reboooting RDS. 
* Read Replicas - For performance (write to one primary db, replicate to others)

##### What is Data Warehousing?

Used to pull in very large and complex data sets 

OLTP - Online Transaction Processing - transaction query
OLAP - Online Analytics Processing - analytics, hard core queries

Redshift! is Amazons answer to supporting OLAP
RDS is the solution to OLTP

#### Elasticache 
A webservice that allows you to deploy and manage In memory cache supports two open source in memory caching engines.
* MemcachedD - simple cache, scales horizontally, multithreaded
* Redis - Not multithreaded. Scales horizontally, advanced data types, ranking sorting data persistence multi AZ


### How to

* RDS runs on virtual machine, cannot log into these OSes. Amazon's responsibility to patch
* NOT serverless, with the exception of Aurora serverless
* RDS will be spun up behind another security group
* Allow MySql protocol in/out to the web security group for it to be able to talk to your web server

### Back up, Multi-AZ and read replicas

* Backups - Automated Backups allow you to recover your database to any point within a "retention period",
The retention period can be between 1-35 days, will take a daily snapshot and transaction logs. 
The restored version will be a new RDS instance with a new DNS endpoint

* DB snapshot - manual, and snapshot stored even after deletion of db.

#### Read replicas
* Must have automatic backups turned on. 
* Upto 5 permitted
* Each read replica has its own DNS endpoint
* Can have multi AZ
* Can promote them to be their own db, will break replication
* Can have one in a completely different region.

Encryption is done using Key Management Service for at rest data

#### DynamoDB

* Document + key-value
* Fully managed + reliable performance
* Stored on SSD
* Spread across 3 geographical distinct data centres
* Eventually consistent reads (default) - consistency reached within a second - best read performance
* Stringly consistent reads - before a second

#### Redshift
* fully managed + petabyte-scale data warehouse
* OLAP 
* Single Node or Multi node (Leader Node and upto 120 compute nodes)
* Column compression to save space, doesnt use indexes so less space. Chooses best compression scheme based on data
* Massively Parallel Processing - Automatically distributes and quert load across all nodes
* BAckups - default 1 day retention period, max is 35
* Always tries to maintain at least 3 copies of your data (2 on compute nodes, one in s3)
* Can asynchronously back up to s3 in another region
* Billed for 1 unit per node per hour, not chanrged for leader node
* Encryption in transit through SSL, and at rest 
* Default mananges keys, can mange your own through HSM
* Currently only in one AZ, can use snapshots on other AZs/

#### Aurora

* * MySql compatible, relational db.
* * Scales in 10GB increments to 64TB
* * 2 copies of data in each AZ, with minimum of 3 availability zones
* Handles the loss of up to two copies of data without affecting the db write availability and 3 without affecting read availability
* Self healing disks
* Can change mysql -> aurora by creating a read replica and promoting it 

Read replicas
* Aurora replicas - automated failover
* Mysql replicas - no automated failover

* Automated backups always enabled 
* BAckups do not impact db performance
* You can take snapshots with Aurora w/o affecting performance, which can be shared with other aws accounts


