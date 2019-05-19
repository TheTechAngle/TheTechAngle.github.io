
## EC2 Instances

A virtualized and abstracted subset of a physical server

### 1) Provisioning Your Instance

* You have to configure the instance's operating system and software stack, hardware specs (the
CPU power, memory, primary storage, and network performance), and environment.

* The OS is defined by the Amazon Machine Image (AMI) you choose, and the
hardware follows the instance type.

### EC2 Amazon Machine Images

An AMI is a template document that contains the OS and application software info to include on the EC2's root data volume. A particular AMI will be available in only a single region—although there will often
be images with identical functionality in all regions.

* Amazon Quick Start AMIs - officially supported.
* AWS Marketplace AMIs - officially supported. and by industry vendors
* Community AMIs - created and maintained by independent vendors
* Private AMIs - You can also store images created from your own instance deployments
  
### Instances Types

General purpose T3, T2, M5, M4
Compute optimized C5, C4
Memory optimized X1e, X1, R5, R4, z1d
Accelerated computing P3, P2, G3, F1
Storage optimized H1, I3, D2

T2s are burstable, which means you can accumulate CPU credits when
your instance is underutilized that can be applied during high-demand
periods in the form of higher CPU performance.

### Configuring an Environment

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
