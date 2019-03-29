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

##### Primary and secondary Private IP Addresses

Each instance must have a primary private IP address from a range specified in the subnet CIDR, bound to the primary ENI which can't be
changed, though you can bind secondary addresses.

* Web Console Command - * Network+Security -> Network Interfaces
                        * Specify subnet, choose default security group
                        * launch instance into the subnet, attaching ENI as primary ENI
                    
#### 4) Internet Gateways

An internet gateway allows the instance to connect to the internet. AWS identifies one by its resource ID, which begins with igw- followed by an alphanumeric string. You need a default route in a route table that points to the Internet gateway as 
a target.

#### 5) Route Tables

IP routing is implemented as a software function, called an implied or implicit router. a *main route table* is
created with every VPC, and associated with every subnet. You can have custom route tables. If a subnet is not 
associated with a table, it will automatically associate with the main route table.

##### Routes

Routing is based only on the destination IP address. A route must include 
* Destination - IP prefix in CIDR notation
* Target - AWS network resource (ex Internet gateway or ENI)

###### Local Route
Every route table has a mandatory local route to allow instances in different subnets to communicate. VPC
with the CIDR 172.31.0.0/16 would have Destination 172.31.0.0/16 with Target Local.

###### Default Route
The default route with Destination 0.0.0.0/0 is pointed to the internet gateway as target. A subnet that is associated with 
a route table containing a default route is called a *public subnet* 

* Web Console Command - * VPC Dashboard -> Internet Gateways -> Create Internet Gateway button -> Click Create.
                        * Select the Internet gateway you just created, and under the Actions menu, click Attach To VPC.
                        * Select the VPC you used in the previous exercises and then click Attach.
                        * VPC Dashboard -> Route Tables -> Main route table for the VPC.
                        * Click the Routes tab ->click Edit -> Add Another Route button.
                        * Enter 0.0.0.0/0 for dest, For the target, enter the identifier of the Internet gateway -> Click Save.
 #### 4) Security Groups
 
 A security group functions as a firewall to control traffic to and fro the ENI. Every ENI must have a security group, though its a many to many mapping. A security group needs a a group name, description, and VPC, inbound and outbounds routes. Each VPC contains a default security group that you can’t delete. The most specific rule is applied. 
 
 ##### Inbound Rules
 
 Requires a source, Protocol and port range. It is created with a default-deny or whitelisting, denying all traffic not allowed by a rule. For example, if you wanted to allow all TCP traffic coming in from port 443 (default for HTTPS)
to allow anyone on the internet access to the instance, and maybe SSH access to IP address 198.51.100.10 on port 22 (default port) you'd need - 
1) Source:Protocol:Port Range - 198.51.100.10/32:TCP:22 and 0.0.0.0/0:TCP:443

##### Outbound Rules
Requires a Destination, Protocol and port range. The default rule 1) Destination:Protocol:Port Range 0.0.0.0/0:All:All. 

##### Sources and Destinations
The source and destination in a rule can be any CIDR or the resource ID of a security group.

##### Stateful Firewall 
When you allow inbound HTTPS access, or any traffic in one direction, to an instance, the security group automatically allows reply traffic from the instance to the client.

* Web Console - EC2 or VPC dashboard -> click Security Groups -> Create Security Group.
              - Assign the security group a name, and select the VPC
              - Inbound tab, click Add Rule -> Create.
              
#### 5) Network Access Control Lists

An NACL also contains rules to allow traffic based on a source or destination, and every VPC has a default NACL ttat can't be deleted. Unlike a security group, it is stateless, much like an ACL on a router, it doesnt track the state of connections passing through it.

An NACL is not attached to an ENI, but to the subnet. So they can't be used to control traffic between two instances in the same subnet. A subnet can have only one NACL, but an NACL can be attached to many subnets in the same VPC.
 
##### Inbound Rules 

Each rule requires Rule number, Protocol, Port range, Source, and Action (Allow or Deny). NACL rules are processed in asecnign order.



