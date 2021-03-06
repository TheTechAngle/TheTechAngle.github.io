
## EC2 Instances

A virtualized and abstracted subset of a physical server

### 1) Provisioning Your Instance

* You have to configure the instance's operating system and software stack, hardware specs (the
CPU power, memory, primary storage, and network performance), and environment.

* The OS is defined by the Amazon Machine Image (AMI) you choose, and the
hardware follows the instance type.

### 2) EC2 Amazon Machine Images (AMI) 
Basically the virtual machine

* An AMI is a template document that contains the OS and application software info to include on the EC2's root data volume.
* A particular AMI will be available in only a single region—although there will often
be images with identical functionality in all regions.
* You can use AMIS to move an Ec2 volume from one AZ to another
You can choose based on region, OS, architecture (32 vs 64) launch permissions or storage (instance vs ebs)

* Amazon Quick Start AMIs - officially supported.
* AWS Marketplace AMIs - officially supported. and by industry vendors
* Community AMIs - created and maintained by independent vendors
* Private AMIs - You can also store images created from your own instance deployments
  
### 3) Instances Types

* General purpose T3, T2, M5, M4
* Compute optimized C5, C4 (CPU intensive stuff)
* Memory optimized X1e, X1, R5, R4, z1d (Apache spark etc) (X for extreme memory, R for RAM)
* Accelerated computing P3, P2, G3, F1 (ML, Mining, gaming)Think P for pics, G for graphics
* Storage optimized H1, I3, D2 (HDFS Map reduce, NoSql etc) (H for high disk throughput, I for IOPS, D dor density)

T2s are burstable, which means you can accumulate CPU credits when
your instance is underutilized that can be applied during high-demand
periods in the form of higher CPU performance.

### 4) Configuring an Environment

#### AWS Regions
Launch an EC2 instance in the region that’s physically closest to the majority of your customers.

#### VPCs
#### Tenancy
The default setting is shared tenancy, where your instance will run as a virtual machine on a
physical server that’s concurrently hosting other instances.
The Dedicated Instance option ensures that your instance will run
on its own dedicated physical server.
The Dedicated Host option allows you to actually identify and control the physical server

#### Configuring Instance Behavior

* Bootstrapping: The EC2 can execute commands as it boots based on user data in your instance configuration

#### Instance Pricing 

* On-demand model - flexible, can control how much you pay by stopping and starting your instances according to your need. Most expensive per hour.
* Reserve Instance -for 24/7 instances, 1 year or 3 years
  * Standard - Can't convert one reserved intance type to another
  * Convertible - Can switch
  * Scheduled reserverd instance - EXACTLY WHAT IT SAYS
* Spot Capacity - Amazons excess that is lent randomly, good for stuff with flexible start-end times. Have to pay by the hour unless Amazon ends tenancy
* Dedicated hosts - Cases that does not support multi tenancy
one or two reserve instances to cover its normal customer demand but also allow autoscaling to automatically launch on-demand instances during periods of unusually high demand.

#### Instance Lifecycle

* Terminating - Will shut it down, resources will be reallocated. Termination protection is turned off by default
* Stopping - Will lose nonpersistent public IP address and data on instance volume, but not data on EBS vol

You can also change its instance type to increase or decrease its compute, memory, and stor-
age capacity. You will need to stop the instance, change the type, and then restart it.

#### Resource Tags

AWS resource tags can be used to label everything, for the keen organizer.

#### Serive Limits

For example, you're allowed only 5 VPCs per region and 5000 key value pairs.

### 5) EC2 Storage Volumes

Virtualized spaces carved out of larger physical drives.

#### Elastic Block Store Volumes

* You can attach as many of these as you like. The AWS SLA guarantess atleast 99.999% availability, but even if the drive does fail, the data has already been duplicated within its avilability zone
and is revived asap. 
* There are four EBS volumes, two using solid state drive (SSDs) and two using the older spinning hard drives (HDDs). An IOPS means input/output operations per second.
* Same AZ as EC2 instance

* The EBS Root Volumes are where the OS is stored
* By default, the root EBS volume is deleted on termination
* EBS Root Volumes of your Default AMIs cannot be encrypted. The additional volumes can be though

### Provisioned IOPS SSD (API name io1) - For applications requireing intense rates of I/O operations | 32k IOPS/vol |  500MB/s max throughput/vol
### General-Purpose SSD (API name gp2) - For general purpose applications and cheaper than the above | 10k IOPS/vol | 160 MB/s throughput/vol 
### Throughout-Optimized Hard Disk Drive (API name st1) - Only 500 IOPS/vol, but with 500MB/s max throughput/vol, and much cheaper
### Cold HDD (API name sc1) - For larger volumes of data with infrequent access. 250 IOPS/vol  and 250 MB/s but the cheapest.
### EBS Magnetic - Older generater, for infrequently accessed data

IOPS/vol = IOPS > General Purpose > Throughout Opt > Cold HDD
Throughput/vol = IOPS > Throughout Opt  >Cold HDD>General Purpose SSD
Cost = IOPS SSD>General Purpose SSD>Throughout Opt HDD>Cold HDD

##### EBS Volume Features

* All EBS volumes can be copied by creating a snapshot. 
* Volumes exist on EBS, snapshoys exist on S3. Only blocks that have changes since your last snapshot are moved to S3 ie they are incremental
* Existing snapshots can be used to generate other volumes
* You can also generate an AMI image directly from a running instance-attached EBS volume
* EBS volumes can be encrypted to protect their data while at rest or as it’s sent back and
forth to the EC2 host instance. 
* EBS can manage the encryption keys automatically behind the scenes or use keys that you provide through the AWS Key Management Service (KMS).
* EBS volumes can be snapshot while the instance is running, unless they're root volumes, which is when the instance has to stop
* You can change the size and volume type on the fly
* AMIs can be created from both volumes and snapshots
* Volumes will be in the same availability zone as the EC2 instance

#### Instance Store Volumes
Cannot be added after launch.
Unlike EBS volumes, instance store volumes are ephemeral. The data is lost when the instances they're attached to are shut down (not rebooted). So why would you choose these over EBS?

* They are SSDs that are physically attached to the server, connected via a fast NVMe interface
* The price is included in th price of the instance itself
* Great for models where instances are launched fill short term roles.

#### Encrypted Root Device
 * EBS volume with the OS is called the root devide.
 * You couldn't encrypt before, had to create snapshot. Then create a copy of it and encrypt it. And then create AMI from it and spin up a new EC2 with it.
 * Now you can encrypt right from the start

#### Accessing Your EC2 Instance

All EC2 devices have unique IP addresses, and will have at least one private IPv4 address. You will only be able to connect to the instance within the subnet.

Running the following curl command from the command line while logged into the instance will return a list of the kinds of data that are available:

$ curl http://169.254.169.254/latest/meta-data/

Webserver
$ ssh ec2-user@publicipaddress -i key
$ sudo su :P
$ yum update -y 
$ yum install httpd // Intslls apache will turn ec2 instace to webserver
then anythin in /var/www/html will be accessible via port 80
$ service httpd start // will start it
$ chkconfig on // will make sure it starts even in case of a reboot

### 6) Securing Your EC2 Instance

#### a) Security Group 
* Plays the role of a firewall, will deny all incoming traffic by default, allows all outbound traffic.
* Changes to security group takes place immediately
* EC2-security group -> many to many relationship

#### b) IAM Roles
When a particular role is assigned to a user or resource, they’ll gain access to whichever resources were included in the role policies. You can also assign an IAM role to an EC2 instance.

#### c) NAT Devices
To manage your instances connections to the internet. A NAT instance and a NAT gateway both do the job, but since a NAT gateway is a managed service, you don't have to manually launch and maintain an instance.

#### d) Key Pairs
You’ll need to generate a key pair, save the public key to your EC2 server, and save its private half to your local machine.

### 6.5) Cloudwatch vs CloudTrail;

* Monitors performance, 5 mins by default, can have 1 min granularity
* CPU, Network, Disk, Status check
* Can have alarms
* CloudTrail is more of a cctv, recording api calls and console actions.

### 7) Other EC2-Related Services

#### a) AWS Systems Manager
Systems Manager Services (available through the AWS console) is a collection of tools for monitoring and managing the resources you have running in the AWS cloud and in your own on-premises infrastructure.

#### b) Placement Groups
Useful for multiple instances that require low-latency network interconnectivity.
* Cluster groups launch each associated instance within a single availability zone within close physical proximity to each other.
* Spread groups separate instances physically across hardware to reduce the risk of failure-related data or service loss.

#### c) AWS Elastic Beanstalk
Lets you upload your application code and define a few parameters, and AWS will configure, launch, and maintain all the infrastructure necessary to keep it running.

#### d) Amazon Elastic Container Service and AWS Fargate
AWS lets you launch a prebuilt Docker host instance and define th eway you want your Docker containers to behave (called a task). With Fargate, all you do is package your application and set your environment requirements.

#### e) AWS Lambda 
Lambda allows you to instantly perform almost any operation on demand at almost any time but without having to provision and pay for always-on servers.

#### f) Elastic Load Balancing and Auto Scaling
* A load balancer directs external user requests between multiple EC2 instances to more efficiently distribute server resources and improve user experience. 
* Autoscaling will react to preset performance thresholds by automatically increasing or decreasing the number of EC2 instances you have running to match demand.


### 8) AWS Console
* aws configure
* aws <service> ls
* cd .aws (for credentials
  
### 9) Bash Scripting

* In configure details -> Advanced options
* #!/bin/bash -> path to the interpreter -> takes commands and runs them
* you can put in any command you want in here, each on a new line
* Make sure the EC2 has admin access

### 10) Instance metadata

* curl http://169.254.169.254/latest/user-data/ -> you can see what was run while bootstrap
* curl http://169.254.169.254/latest/meta-data/ -> all sorts of stuff displayed, can view each individual thing like as below 
* curl http://169.254.169.254/latest/meta-data/local-ipv4/ -> for the ipv4 

### 11) Amazon Elastic File System (Amazon EFS)

* Elastic and easily scalable storage capacity - so that your applications have the storage they need, when they need it
* You can share it across two EC2 instances
* Has life cycle management too
* sudo yum install -y amazon-utils-efs -> to install it
* sudo mount -t efs tls fs-blahblah filepath (filepath indicates what its gonna store)
* Communicate via NFS (port 2040), so make sure to allow that on the EC2 security group
* You only pay for what you use unlike EBS
* Scale up to petabytes
* Can support thousands of concurrent connections
* data stored across multiple AZs in a regions
* READ AFTER WRITE consistency

### 12) EC2 Placement Groups

* Just a way of placing your instances. Name you give for the group must be unique in your account
* Only certain instances can use this grouping (compute optimized, GPU, Memory optimized, Storage optimized)
* Cant merge groups
* cant move an already existing instance to a group. Can create an AMI of it and launch a new one from the AMI into he group
* Clustered - grouping within a single AZ for low latency, and/or high throughput. CANNOT span multipe AZs
* Spread - each on distict isolated hardware, for apps that have small number of critical instances 
* Partitioned - each group has logical 'partitions' which has its own set of racks. No two partitions share the same rack, so as to isolate hardware failure - use case HDFS, cassandra clusters




