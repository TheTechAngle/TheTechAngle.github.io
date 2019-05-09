## (A) S3 Service Architecture

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
Use only Amazon’s encrypted API endpoints for data transfers.

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
* The high durability rates delivered by most S3 are largely because they automatically replicate your data across at least three availability zones, with S3 standards having a durability % of 99.999999999%
* Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA), stores data in only a single availability zone, but durability of 99.999999999%
* Reduced Redundancy Storage (RRS) is rated at only 99.99% durability (because it’s replicated across fewer servers than other classes).
* S3 Standard == S3 One Zone > RRS

### 2) Availability - Can you connect?
S3 Standard (99.99) == Reduced Redundancy > S3 Standard IA (99.9) > S3 One Zone (99.5) % in a year

### 3) Eventually Consistent Data
* Because there isn’t the risk of corruption, S3 provides read-after-write consistency for
the creation (PUT) of new objects.

## (C) S3 Object Lifestyle
### 1) Versioning
* By default saving a file with the same name and location as another will overwrite it
* Enabling versioning saves old copies indefinitely

### 2) Lifecycle Management
* You can set the lifecycle rules for a bucket, to move an object's storage class after a number of days,
eventually deleting it.
* You can apply them to only certain objects in the bucket
* There a minimum time an object must spend in one class before it can be moved (30 days)

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

## (F) Storage Pricing

* Charged for storage, data retrieval operations like GET and PUT and lifecycle transition requests
* https://aws.amazon.com/s3/pricing/.

## (G) Other Storage-Related Services
#### 1) Amazon Elastic File System
EFS-based files are designed to be accessed from within a VPC via
Network File System (NFS) mounts on EC2 instances or from your on-premises servers
through AWS Direct Connect connections. Enables secure,low-latency, and durable file sharing among multiple instances.

#### 2) AWS Storage Gateway
Local devices can connect to the appliance as though it’s a physical backup device like a tape drive, while the data itself is saved to AWS platforms

#### 3) AWS Snowball

When requested, AWS will ship you a physical, 256-bit, encrypted Snowball storage device onto which you’ll copy your data. You then ship the device back to Amazon where its data will be uploaded to your S3 bucket(s).


