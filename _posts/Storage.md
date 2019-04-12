## (A) S3 Service Architecture

S3 is object oriented storage, and an S3 bucket and its contents exist only within a region, but the name must be GLOBALLY unique.

* Block-level storage - data on a raw physical storage device is divided into individual blocks whose use is managed by a file system. NTFS is a common file system used by Windows, while Linux might use Btrfs or ext4.
* Object storage - data is stored more on a flat surface. 

Access over HTTP - https://s3.amazonaws.com/bucketname/filename
Acess over AWS CLI - s3://bucketname/filename

### 1) Prefixes and Delimiters
You can use prefixes and delimiters to give your buckets the appearance of a more structured organization.

### 2) Working with Large Objects
No upper limit.
* Single object limit -  5TB
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
* Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA), stores data in only a single availability zone.
* Reduced Redundancy Storage (RRS) is rated at only 99.99% durability (because it’s replicated across fewer servers than other classes).

### 2) Availability - Can you connect?
S3 Standard (99.99) == Reduced Redundancy > S3 Standard IA (99.9) > S3 One Zone (99.5) % in a year

### 3) Eventually Consistent Data
* Because there isn’t the risk of corruption, S3 provides read-after-write consistency for
the creation (PUT) of new objects.

## (C) S3 Object Lifestyle


