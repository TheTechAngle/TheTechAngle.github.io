
## Database Systems
* MariaDB, Oracle, Aurora DB, SQL Server, MySql
* Does not have Cassandra or MongoDB
* If you want your application to check RDS for an error, have it look for an ERROR node in the response from the Amazon RDS API
Two key features
* maximum size RDS volume - 16TB
* Multi-AZ - For Disaster Recovery only (Aurora is completely Fault Tolerant doesnt need this). Can change AZ by failover by reboooting RDS. 
* Read Replicas - For performance (write to one primary db, replicate to others) Only SqlServer doesnt support this

##### What is Data Warehousing?

Used to pull in very large and complex data sets 

OLTP - Online Transaction Processing - transaction query
OLAP - Online Analytics Processing - analytics, hard core queries

Redshift! is Amazons answer to supporting OLAP
RDS is the solution to OLTP

#### Elasticache 
A webservice that allows you to deploy and manage In memory cache supports two open source in memory caching engines.
* MemcachedD - simple cache, scales horizontally, multithreaded
* Redis - Not multithreaded. Scales horizontally, advanced data types, ranking sorting data persistence multi AZ, backups etc


### How to
* CAnb select which AZ you're deploying it into
* RDS runs on virtual machine, cannot log into these OSes. Amazon's responsibility to patch
* NOT serverless, with the exception of Aurora serverless
* RDS will be spun up behind another security group
* Allow MySql protocol in/out to the web security group for it to be able to talk to your web server
* MySQL installations default to port number 3306 

### Back up, Multi-AZ and read replicas

* Backups - Automated Backups allow you to recover your database to any point within a "retention period"
* changes to the backup window take effect immediately
* The retention period can be between 1-35 days, will take a daily snapshot and transaction logs. 
* The restored version will be a new RDS instance with a new DNS endpoint
* I/O may be briefly suspended while the backup process initializes (typically under a few seconds), and you may experience a brief period of elevated latency.
* DB snapshot - manual, and snapshot stored even after deletion of db.
* NOT ENALBLED by default

#### Multi-AZ
* RDS Reserved instances are available for multi-AZ deployments

#### Read replicas
* Must have automatic backups turned on. 
* Upto 5 permitted
* Each read replica has its own DNS endpoint
* Can be multi AZ
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
* There will always be a charge for provisioning read and write capacity and the storage of data within DynamoDB. Noo charge for the transfer of data into dynamdb provided you stay in a single region
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
* * 2 copies of data in each AZ, with minimum of 3 availability zones (six copies)
* Handles the loss of up to two copies of data without affecting the db write availability and 3 without affecting read availability
* Self healing disks
* Can change mysql -> aurora by creating a read replica and promoting it 

Read replicas
* Aurora replicas - automated failover
* Mysql replicas - no automated failover

* Automated backups always enabled 
* BAckups do not impact db performance
* You can take snapshots with Aurora w/o affecting performance, which can be shared with other aws accounts

#### WHAT
* You are hosting a MySQL database on the root volume of an EC2 instance. The database is using a large number of IOPS, and you need to increase the number of IOPS available to it. What should you do?
- Add 4 additional EBS SSD volumes and create a RAID 10 using these volumes

* Amazon Athena is an interactive query service that makes it easy to analyse data in Amazon S3, using standard SQL commands. It will work with a number of data formats including "JSON", "Apache Parquet", "Apache ORC" amongst others, but "XML" is not a format that is supported. 

