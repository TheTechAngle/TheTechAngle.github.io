You need to be able to build your own VPC from memory. 

What is it?

The networking layer of AWS, the Amazon Virtual Private Cloud lets you provision a logically isolated section of the cloud where
you have full control over the virtual network and can deploy AWS resources.

* You can have multiple VPCs in a region, but a VPC can exist only in one region and won't show up anywhere else
* Unlike a traditional TCP/IP network, a VPC doesn't have components like routers, switches etc. Instead it
they're abstracted into software functions, allowing it to scale.

#### 1) VPC CIDR Blocks
* CIDR - Classless interdomain routing block - a range of contiguous IP addresses. For example, the CIDR 
(10.0.0.0/8) includes all addresses from 10.0.0.0–10.255.255.255.
* Secondary CIDR Blocks - These blocks MUST come from either the *same address range* as the primary or a publicly
routable range, but they must not overlap with the primary or other secondary blocks. If the VPC’s primary CIDR is 172.16.0.0/16, you may specify a secondary CIDR
of 172.17.0.0/16. But you may not specify 192.168.0.0/16.
* IPv6 CIDR Blocks - You can't choose this block, it will be allocated to you at your request. The IPv6 CIDR will be publicly routable
prefix form the global unicast IPv6 address space. The prefix length is always /56
* CLI command - aws ec2 create-vpc-cidr-block 172.16.0.0/16


#### 2) Subnets
A subnet is a logical container within a VPC that holds your EC2 instances, to control and organize your internet access and control.
Once an instance is launched into a subnet, you can't move it. You will need to terminate it and create a new one.

* Subnet CIDR blocks - The CIDR block for your subnet has to be a subset of your VPC primary or secondary CIDR block.
They can't overlap and cannot be changed, and it cannot have more than one CIDR block.
The first four and last IP addresses are reserved by AWS. For example if your CIDR block is 172.16.100.0/16, then 172.16.100.0-172.16.100.3
and 172.16.100.255 will be reserved.

##### Availability Zones

An availability zone is a rough mapping to a geographic location like a datacenter. It is recommended you have subnets in two 
different availability zones to increase failure resistence.

* IPv6 CIDR Blocks - If you have an IPv6 CIDR allocated to your VPC, you can assign them to subnets in the VPC, with a prefix /64.
You must ALWAYS assign an IPv4 CIDR block to a subnet. (WHY) 
* CLI command - aws ec2 create-subnet-vpc-id [VPC resource ID] --cidr-block 172.16.100.0/24 --availability-zone us-east-1a 

#### 3) Elastic Network Interfaces
An elastic network interface performs the same basic functions as a network interface on a physical server. Every instance must have a
primary network interface (aka primary ENI) which is connected to only one subnet. It is possible to have additional ENIs, bu thtey must
be in the same availability zone (not necessarily same subnet) as the instance.

An ENI can exist independently of an instance and can be attached and detached as a secondary instance whenever. The instance 
can be terminated without deleting the ENI.

#### Primary and secondary Private IP Addresses

Each instance must have a primary private IP address from a range specified in the subnet CIDR, bound to the primary ENI which can't be
changed, though you can bind secondary addresses.
