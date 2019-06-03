
## EC2 Instances

A virtualized and abstracted subset of a physical server

### 1) Provisioning Your Instance

* You have to configure the instance's operating system and software stack, hardware specs (the
CPU power, memory, primary storage, and network performance), and environment.

* The OS is defined by the Amazon Machine Image (AMI) you choose, and the
hardware follows the instance type.

### 2) EC2 Amazon Machine Images

An AMI is a template document that contains the OS and application software info to include on the EC2's root data volume. A particular AMI will be available in only a single region—although there will often
be images with identical functionality in all regions.

* Amazon Quick Start AMIs - officially supported.
* AWS Marketplace AMIs - officially supported. and by industry vendors
* Community AMIs - created and maintained by independent vendors
* Private AMIs - You can also store images created from your own instance deployments
  
### 3) Instances Types

General purpose T3, T2, M5, M4
Compute optimized C5, C4
Memory optimized X1e, X1, R5, R4, z1d
Accelerated computing P3, P2, G3, F1
Storage optimized H1, I3, D2

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
* Reserve Instance -for 24/7 instances, for more than a year

one or two reserve instances to cover its normal customer demand but also allow autoscaling to automatically launch on-demand instances during periods of unusually high demand.

#### Instance Lifecycle

* Terminating - Will shut it down, resources will be reallocated
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

You can attach as many of these as you like. The AWS SLA guarantess atleast 99.999% availability, but even if the drive does fail, the data has already been duplicated and is revived asap. There are four EBS volumes, two using solid state drive (SSDs) and two using the older spinning hard drives (HDDs). An IOPS means input/output operations per second.

### EBS-Provisioned IOPS SSD - For applications requireing intense rates of I/O operations | 32k IOPS/vol |  500MB/s max throughput/vol
### EBS General-Purpose SSD - For general purpose applications and cheaper than the above | 10k IOPS/vol | 160 MB/s throughput/vol 
### Throughout-Optimized HDD - Only 500 IOPS/vol, but with 500MB/s max throughput/vol, and much cheaper
### Cold HDD - For larger volumes of data with infrequent access. 250 IOPS/vol  and 250 MB/s but the cheapest.

IOPS/vol = IOPS SSD>General Purpose SSD>Throughout Opt HDD>Cold HDD
Throughput/vol = IOPS SSD>Throughout Opt HDD>Cold HDD>General Purpose SSD
Cost = IOPS SSD>General Purpose SSD>Throughout Opt HDD>Cold HDD

##### EBS Volume Features

* All EBS volumes can be copied by creating a snapshot. 
* Existing snapshots can be used to generate other volumes 
* You can also generate an AMI image directly from a running instance-attached EBS volume
* EBS volumes can be encrypted to protect their data while at rest or as it’s sent back and
forth to the EC2 host instance. 
* EBS can manage the encryption keys automatically behind the scenes or use keys that you provide through the AWS Key Management Service (KMS).

#### Instance Store Volumes

Unlike EBS volumes, instance store volumes are ephemeral. The data is lost when the instances they're attached to are shut down. So why would you choose these over EBS?

* They are SSDs that are physically attached to the server, connected via a fast NVMe interface
* The price is included in th price of the instance itself
* Great for models where instances are launched fill short term roles.

#### Accessing Your EC2 Instance

All EC2 devices have unique IP addresses, and will have at least one private IPv4 address. You will only be able to connect to the instance within the subnet.

Running the following curl command from the command line while logged into the instance will return a list of the kinds of data that are available:

$ curl http://169.254.169.254/latest/meta-data/

### 6) Securing Your EC2 Instance

#### a) Security Group 
Plays the role of a firewall, will deny all incoming traffic by default.

#### b) IAM Roles
When a particular role is assigned to a user or resource, they’ll gain access to whichever resources were included in the role policies. You can also assign an IAM role to an EC2 instance.

#### c) NAT Devices
To manage your instances connections to the internet. A NAT instance and a NAT gateway both do the job, but since a NAT gateway is a managed service, you don't have to manually launch and maintain an instance.

#### d) Key Pairs
You’ll need to generate a key pair, save the public key to your EC2 server, and save its private half to your local machine.

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
