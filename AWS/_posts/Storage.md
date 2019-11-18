## (A) S3 (Simple Storage Service) Architecture

S3 is object oriented storage, and an S3 bucket and its contents exist only within a region, but the name must be GLOBALLY unique.

* Block-level storage - data on a raw physical storage device is divided into individual blocks whose use is managed by a file system. NTFS is a common file system used by Windows, while Linux might use Btrfs or ext4.
* Object storage (S3) - data is stored more on a flat surface and avoids some of the OS-related complications of block storage, allowing anyone easy access to the storage capacity. 

Access over HTTP - https://s3.amazonaws.com/bucketname/filename
Acess over AWS CLI - s3://bucketname/filename

### 1) Prefixes and Delimiters
You can use prefixes and delimiters to give your buckets the appearance of a more structured organization.

### 2) Working with Large Objects
No upper limit.
* Single object size limit -  5TB
* individual upload - 5GB
* Multipart Upload (!!! I've used this) for objects larger than 100MB. THe object is broken into many parts and uploaded. If any part fails, it can be tried again without affecting the other parts.
* CLI command to upload (aws s3 cp mylocalfile.txt s3://mybucketname/)

### 3) Encryption
* Encryption in transit is achieved by SSL/TLS
* Use only Amazon’s encrypted API endpoints for data transfers.

#### Server-Side Encryption for data at rest
3 options allowed by AWS for encrypting data on AWS
* Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)
where AWS uses its own enterprise-standard keys to manage every step of the encryption and decryption process
* Server-Side Encryption with AWS KMS-Managed Keys (SSE-KMS)
where, beyond the SSE-S3 features, the use of an envelope key is added along with a full audit trail or tracking key usage. You can optionally import your own keys through the AWS KMS service.
* Server-Side Encryption with Customer-Provided Keys (SSE-C)
which lets you provide your own keys for S3 to apply to its encryption.

#### Client-Side Encryption before uploading to S3
AWS KMS–Managed Customer Master Key (CMK), which produces a unique key for each object
before it’s uploaded. You can also use a Client-Side Master Key, which you provide through
the Amazon S3 encryption client.

### 4) Logging
* Disabled by default
* Need to specify both a source bucket (the bucket whose activity you’re tracking) and target bucket (the bucket to which you’d like the logs saved).
* Stores the account and IP address of the requestor, source bucket name,the action that was requested (GET, PUT, etc.), time and response status

## (B) S3 Durability and Availabilty

### 1) Durability - Your data endures
* The high durability rates delivered by most S3 are largely because they automatically replicate your data across at least three availability zones
* ALL S3 having a durability % of 99.999999999% (ELEVEN 9s) cept RRS
* S3 Infrequent Access (S3 IA) - data that is accessed less frequently, but rapid access when needed. Lesser cost
* S3 One Zone-Infrequent Access (S3 One Zone-IA), stores data in only a single availability zone
* S3 Intelligent Tiering - moves it around the other tiers based on what its learnt about your access with ML
* Reduced Redundancy Storage (RRS) is rated at only 99.99% durability (because it’s replicated across fewer servers than other classes). AWS deprecating it
* S3 Standard == S3 One Zone > RRS

### 2) Availability - Can you connect?
*S3 Standard and Glacier (99.99)* == Reduced Redundancy > S3 Standard IA (99.9) > S3 One Zone (99.5) % in a year

### 3) Eventually Consistent Data
* Because there isn’t the risk of corruption, S3 provides read-after-write consistency for
the creation (PUT) of new objects (able to read immediately)
* Overwriting is eventual consistency, you may get version 1 if you try within a second.

## (C) S3 Object Lifestyle
### 1) Versioning
* By default saving a file with the same name and location as another will overwrite it
* Enabling versioning saves old copies indefinitely
* Cannot disable versioning once enables. Can be suspended
* Size of the S3 bucket is th sum of all versions
* Even deltes are available
* Has MFA delete capabil ity

### 2) Lifecycle Management
* You can set the lifecycle rules for a bucket, to move an object's storage class after a number of days,
eventually deleting it.
* You can apply them to only certain objects in the bucket
* There a minimum time an object must spend in one class before it can be moved (30 days)

### 3) MFA deletes

## (D) Accessing S3 Objects

### 1) Access Control
* Access Control List (ACL) - Dont use, IAM ancestor
* S3 bucket policies - JSON text attached to your S3 bucket - limited scope (LINK)
* IAM policies - Contorla how users and roles access resources

### 2) Presigned URLs 
* Expire after a certain period of time 
* aws s3 presign s3://MyBucketName/PrivateObject --expires-in 600

### 3) Static Website Hosting

* Store the file, and set the index.html as your index doc in the bucket Properties tab
* You can also get a free SSL/TLS certificate to encrypt your site by requesting a certificate
from Amazon Certificate Manager (ACM) and importing it into a CloudFront distribution
that specifies your S3 bucket as its origin.

### 4) S3 and Glacier Select 
* For efficient, cost effective storage

## (E) Amazon Glacier

* 99.999999% durability, and can be incorporated into the S3 lifecycle config.
* Stored in vaults instead of buckets
* Vault names don't have to be globally unique
* 40TB archive limit, unlike the S3's 5TB limit
* Default archive encryption, S3's to be enabled
* Machine generated Ids, S3's are human readable
* VERY long retrieval time, meant for unusual in frequent access
* Choose S3 Glacier when some of your archived data is needed in as little as 1-5 minutes using Expedited retrievals. 
* S3 Glacier Deep Archive, in contrast, is designed for colder data that is very unlikely to be accessed, but still requires long-term, durable storage.Deep archive can take upto 12 hours

## (F) Storage Pricing

* Charged for storage, versions, data retrieval operations like GET and PUT and lifecycle transition requests
* https://aws.amazon.com/s3/pricing/.
* Data transfers within a region with a COPY request only (even with an EC2) are free, across regions are charged. 
* S3 > S3 IA > S3 IA One Zone > Glacier
* Cross Region Replication - 
* Amazon S3 transfer Acceleration - fast and easy transfers(uploads and downloads) using edge locations

## (G) Other Storage-Related Services
#### 1) Amazon Elastic File System
EFS-based files are designed to be accessed from within a VPC via
Network File System (NFS) mounts on EC2 instances or from your on-premises servers
through AWS Direct Connect connections. Enables secure,low-latency, and durable file sharing among multiple instances.

#### 2) AWS Storage Gateway
Local devices can connect to the appliance as though it’s a physical backup device like a tape drive, while the data itself is saved to AWS platforms

* File Gateway - for flat files to be stored directly on S3
* Volume Gateway - Stored Volumes - Entire dataset stored on site and asynchronously backed up to S3
* Volume Gateway - Cached Volumes - Entire dataset stored on S3 and most frequently accessed is cached on site
* Gateway Virtual Tape Library

#### 3) AWS Snowball

When requested, AWS will ship you a physical, 256-bit, encrypted Snowball storage device onto which you’ll copy your data. You then ship the device back to Amazon where its data will be uploaded to your S3 bucket(s).

Snowball edge offers both 

## H) Cross Region Replication
* Needs versioning enabled source and destination buckets
* Regions must be unique
* Existing files in bucket not replicted automatically
* Delete markers and deleting versions not replicated
* Amazon S3 Replication (CRR and SRR) is configured at the S3 bucket level, a shared prefix level, or an object level using S3 object tags.
* With S3 Replication (CRR and SRR), you can establish replication rules to make copies of your objects into another storage class, in the same or a different region. Lifecycle actions are not replicated, and if you want the same lifecycle configuration applied to both source and destination buckets, enable the same lifecycle configuration on both. 

## I) S3 Transfer Acceleration and CloudFront
* edge locations to accelerate uploads and downloads
* edge locations have caches, stored upto the ttl (time to live)
* Invalidating cached content is possible, you will be charged for it. Create invalidation in the CloudFrontDistributions
* Content Delivery Network (CDN) a system of distributed servers that delivers content to a user based on their geographic position, the origin of the webpage and a content deliver server
* A CDN is a network of edge locations, also called distribution
* Cloud Front offers two types of distributions - a web distribution for websites and RTMP for media streaming


READ S3 FAQS BEFORE TAKING THE EXAM
* Amazon Macie is an AI-powered security service that helps you prevent data loss by automatically discovering, classifying, and protecting sensitive data stored in Amazon S3
* You can have a bucket that has different objects stored in S3 Standard, S3 Intelligent-Tiering, S3 Standard-IA, and S3 One Zone-IA.
* Amazon S3 allows customers to run sophisticated queries against data stored without the need to move data into a separate analytics platform
* S3 offers multiple query in place options, including S3 Select, Amazon Athena (analytics heavy), and Amazon Redshift Spectrum (data size heavy), allowing you to choose one that best fits your use case.
* Amazon S3 event notifications can be sent in response to actions in Amazon S3 like PUTs, POSTs, COPYs, or DELETEs. Notification messages can be sent through either Amazon SNS, Amazon SQS, or directly to AWS Lambda.
* S3 Transfer Acceleration a better choice than CloudFront if a higher throughput is desired (for >1GB uploads)
* Storage Class Analysis, an S3 feature, you can analyze storage access patterns and transition the right data to the right storage class. 
* Amazon S3 Object Lock is a new Amazon S3 feature that blocks object version deletion during a customer-defined retention period so that you can enforce retention policies as an added layer of data protection or for regulatory compliance
