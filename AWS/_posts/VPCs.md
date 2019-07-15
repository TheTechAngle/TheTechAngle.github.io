NOTES FROM CLOUD GURU's AWS CERTIFIED SOLUTIONS ARCHITECT COURSE
=====
Edge locations also work the other way round, uploading your data quickly
5 VPCs in a region allowed, not charged for VPC, since its just a container
You need to be able to build your own VPC from memory. 

### What is it?

The networking layer of AWS, the Amazon Virtual Private Cloud lets you provision a logically isolated section of the cloud where
you have full control over the virtual network and can deploy AWS resources.

* You can have multiple VPCs in a region, but a VPC can exist only in one region and won't show up anywhere else
* Unlike a traditional TCP/IP network, a VPC doesn't have components like routers, switches etc. Instead it
they're abstracted into software functions, allowing it to scale.


* There are two ways for you to hit it - an internet gateway and a virtual private gateway. These hit the router and get routed to a network ACL, which acts as a first line of defense
* A new VPC will always have a Route table and a default NACL and security group, but nothing else
* This then connects it to a subnet. Subnets have the security groups.
* Only 1 internetgateway per VPC
* An instance has an ENI connected to only one subnet
* Both VPCS and ENI have security groups
* Always have a separate route table for the public subnet, to associate it with the private subnets

#### 1) VPC CIDR Blocks
* Largest allowed 10.0.0.0/16, smallest 10.0.0.0/28
* CIDR - Classless interdomain routing block - a range of contiguous IP addresses. For example, the CIDR 
(10.0.0.0/8) includes all addresses from 10.0.0.0–10.255.255.255.
* Secondary CIDR Blocks - These blocks MUST come from either the *same address range* as the primary or a publicly
routable range, but they must not overlap with the primary or other secondary blocks. If the VPC’s primary CIDR is 172.16.0.0/16, you may specify a secondary CIDR
of 172.17.0.0/16. But you may not specify 192.168.0.0/16.
* IPv6 CIDR Blocks - You can't choose this block, it will be allocated to you at your request. The IPv6 CIDR will be publicly routable prefix form the global unicast IPv6 address space. The prefix length is always /56
* CLI command - aws ec2 create-vpc-cidr-block 172.16.0.0/16


#### 2) Subnets
* A subnet cannot be spread over more than one availability zone.
* A subnet is a logical container within a VPC that holds your EC2 instances, to control and organize your internet access and control.
* Once an instance is launched into a subnet, you can't move it. You will need to terminate it and create a new one.
* You need to explicitly enable public ipv4 to allow EC2 instances within to be assigned public ip addresses
* Subnet CIDR blocks - The CIDR block for your subnet has to be a subset of your VPC primary or secondary CIDR block.
They can't overlap and cannot be changed, and it cannot have more than one CIDR block.
The first four and last IP addresses are reserved by AWS. For example if your CIDR block is 172.16.100.0/16, then 172.16.100.0-172.16.100.3
and 172.16.100.255 will be reserved.

* Bastion host - an EC2 instance in a public subnet used to ssh into an EC2 instance in a private subnet

##### Availability Zones

An availability zone is a rough mapping to a geographic location like a datacenter. It is recommended you have subnets in two 
different availability zones to increase failure resistence.

* IPv6 CIDR Blocks - If you have an IPv6 CIDR allocated to your VPC, you can assign them to subnets in the VPC, with a prefix /64.
You must ALWAYS assign an IPv4 CIDR block to a subnet. (WHY) 
* CLI command - aws ec2 create-subnet-vpc-id [VPC resource ID] --cidr-block 172.16.100.0/24 --availability-zone us-east-1a 

#### 3) Elastic Network Interfaces
An elastic network interface performs the same basic functions as a network interface on a physical server. Every instance must have a
primary network interface (aka primary ENI) which is connected to only one subnet. It is possible to have additional ENIs, bu they must be in the same availability zone (not necessarily same subnet) as the instance.

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
A subnet can have multiple route tables
Routing is based only on the destination IP address. A route must include 
* Destination - IP prefix in CIDR notation
* Target - AWS network resource (ex Internet gateway or ENI)

###### Local Route
Every route table has a mandatory local route to allow instances in different subnets to communicate. VPC
with the CIDR 172.31.0.0/16 would have Destination 172.31.0.0/16 with Target Local.

###### Default Route
The default route with Destination (0.0.0.0/0 for ipv4 and ::/0 for ipv6) is pointed to the internet gateway as target. A subnet that is associated with 
a route table containing a default route is called a *public subnet* 

* Web Console Command - * VPC Dashboard -> Internet Gateways -> Create Internet Gateway button -> Click Create.
                        * Select the Internet gateway you just created, and under the Actions menu, click Attach To VPC.
                        * Select the VPC you used in the previous exercises and then click Attach.
                        * VPC Dashboard -> Route Tables -> Main route table for the VPC.
                        * Click the Routes tab ->click Edit -> Add Another Route button.
                        * Enter 0.0.0.0/0 for dest, For the target, enter the identifier of the Internet gateway -> Click Save.
 #### 4) Security Groups
 
 A security group functions as a firewall to control traffic to and fro the ENI. Every ENI must have a security group, though its a many to many mapping. A security group needs a a group name, description, and VPC, inbound and outbounds routes. Each VPC contains a default security group that you can’t delete. The most specific rule is applied. 
 
 * Allow ICMP type access to be able to ping
 * Associated with ENIs/EC2s, not subnets
 
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

An NACL also contains rules to allow traffic based on a source or destination, and every VPC has a default NACL ttat can't be deleted. Unlike a security group, it is stateless, much like an ACL on a router, it doesnt track the state of connections passing through it. Note that in a NACL rule you can specify a CIDR only as the source or destination.
This is unlike a security group rule wherein you can specify another security group for the
source or destination.

An NACL is not attached to an ENI, but to the subnet. So they can't be used to control traffic between two instances in the same subnet. A subnet can have only one NACL, but an NACL can be attached to many subnets in the same VPC.
 
##### Inbound Rules 

Each rule requires Rule number, Protocol, Port range, Source, and Action (Allow or Deny). NACL rules are processed in ascending order. For example Rule:Number:Protocol:Port Range:Source:Action of 90:TCP:80:0.0.0.0/0:Deny will deny all traffic with a *destination port* of 80. The last rule is numbered \*, and is the default rule that denies all traffic that doesn't fall under any of the other rules.

Web Console - 1. VPC Dashboard, click Network ACLs -> Create Network ACL ->Select the VPC
              2. Select the NACL ->click the Inbound Rules tab.
3. If you launched a Linux instance, add an inbound rule numbered 100 to allow SSH
(TCP port 22) access from any IP address. If you launched a Windows instance, add
an inbound rule to allow RDP (TCP port 3389) access from any IP address.
4. Remove any rules that allow other inbound traffic.
5. Connect to the instance and remain connected.
6. Now add another inbound rule numbered 90 that denies all traffic. Your connection
to the instance should drop.

##### Outbound Rules 

Each rule requires Rule number, Protocol, Port range, Destination, and Action (Allow or Deny).

##### Default NACL inbound rules
  'RuleNum Protocol Port Range Source/Dest Action
  '100        All       All        0.0.0.0/0   Allow
  '\*          All       All        0.0.0.0/0    Deny
  
"If you do need to restrict access from the subnet—to block Internet access, for
example—you will need to create an outbound rule to allow return traffic over ephemeral
ports. Ephemeral ports are reserved TCP or UDP ports that clients listen for reply traffic
on. As an example, when a client sends an HTTPS request to your instance over TCP port
80, that client may listen for a reply on TCP port 36034. Your NACL’s outbound rules
must allow traffic to egress the subnet on TCP port 36034. To maintain compatibility since differnt clients use different ports, do not restrict outbound traffic using a NACL. Use a security group instead.
If your VPC includes an IPv6 CIDR, AWS will automatically add inbound and outbound
rules to permit IPv6 traffic"

##### Using Network Access Control Lists and Security Groups Together

#### 6) Public IP Addresses

A public IP address is required if you want the public to access an instance over the internet. You can opt for one when you launch an instance. You *cannot* opt for one later, and if the instance is stopped or terminated, you may lose it.

#### 7) Elastic IP Address

You can request an EIP for your account, after which you have absolute use over it till you release it.
An EIP is bound to a single ENI at a time, until you dissociate it.

Web Console - 1) VPC Dashboard -> click Elastic IPs -> Click Allocate New Address -> Click Allocate.
2. Click the EIP, and under the Actions menu, click Associate Address.
3. Select the instance -> Click Associate.

#### 8) Network Address Translation 

One-to-One NAT
When you give an ENI a public address, the Internet gateway maps that address to the ENI's private IP address using NAT. So if an instance with private IP address A is associated with EIP B, and wants to send something to an internet host C, it sends a packet Src:Dest=A:C to the internet gateway, that converts it to B:C. Likewise an incoming packet C:B will be converted to C:A.

#### 9) Network Address Translation Devices

Apart from the Internet Gateway, th NAT gateway and the NAT instance (called NAT devices) can also perform translations. This is to allow instances one way access to the internet.

Port Address Translation aka PAT
The instance itself doesn't have a public IP address associated with it, but the NAT device does. The packet is sent to the NAT, where the source IP is replaced by the NAT's source IP. At the internet gateway, this is then replaced by the NAT's EIP. Multiple instances can use a NAT device.

#### 10) Configuring Route Tables to Use NAT Devices
Instances that use a NAT device and the NAT device itself must use differnet routes, different route tables and must reside on different subnets.
 For example, the NAT device would be on a public subnet, and the instance would be on a private one.

| Subnet | Destination | Target |
| --- | --- | --- |
| Private |  0.0.0.0/0 | NAT device |
| Public | 0.0.0.0/0 | igw-0e538022a0fddc318 |

##### NAT Device - NAT Gateway
Nothing to manage or access, AWS does it for you. You just create it and assign it an EIP. It can be used in only one subnet. 
NAT gateway ID is of the form nat-0750b9c8de7e75e9f. If you use multiple NAT gateways, you can create multiple default routes, each pointing to a different NAT gateway
as the target.

##### NAT Device - NAT Instance
A NAT instance is a normal EC2 instance that uses a preconfigured Linux-based AMI.
UNLIKE a NAT Gateway
* It can't automatically scale to accomodate increased bandwidth requirements.
* It has an ENI attached to it, so you need to apply a security group, and a public IP address. You should also disable the source/destination checks for the EC2 instance to act as a NAT device, as it should be able to route traffic when it itself isn't the source or destination for traffic.
check on the instance's ENI, to allow it to use source and destination IP addresses other than its own.
* You can use it as a *bation* or *jump* host, to connect to instances w/o a public IP.
* NAT instance ID is of the form i-0a1674fe5671dcb00.
* You cannot create multiple default routes pointing to different NAT instances for resiliency. Why tho

#### 11) VPC Peering 
Allows instances between two VPCs to communicate. You will need to set up one (and only one) VPC peering connection between the two VPCs.
They cannot have overlapping CIDR blocks and cannot be used to share internet gateways and NAT devices. You can use it to share Network Load Balancer.
Routes will have to be set up to allow traffic in both directions, with the VPCs as the target.

#### 12) Setting up your own VPC

1) Create a VPC with primary CIDR as you see fit (eg 10.0.0.0/16) and Amazon provided IPv6 CIDR block
1.5) This will create a route table, a network ACL, and a security group
2) Create two subnets within, with subsets of IP addresses that don't overlap. Allow one to have publicly assigned IP addresses
3) Create an internet gateway, and attack it to the VPC. ONLY ONE PER VPC
4) Create a 'public' route table (map outbound 0.0.0.0/0 and ::/0 to the internet gateway) and  associate the subnet with this route table
4) Spin up two EC2 instances, one in each subnet
4.5) For the Web (public EC2 instance) create a new security group which allows SSH and HTTP from 0.0.0.0/0 and ::/0 to enable internet access. Download the key pair created
4.5) For the DB (private EC2 instance)create a new security group which allows SSH, HTTPS, HTTP and ICDMP from the public subnet's CIDR 
5) We'll need a NAT Gateway to all the private EC2 to download stuff off of the internet. (NAT instances on their way out, but they sit in the public subnet, and the route tables in private subnets route outbound traffic to the NAT instance). Attach it to the public subnet and create a new Elastic IP Allocation.
6) Edit the private subnet' route table to direct all outbound (0.0.0.0/0) traffic to NAT Gateway
