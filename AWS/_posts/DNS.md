Why is Amazon's DNS Service called Route 53?
-> Port 53, and the USA's first road was called Route 66
With Route 53, there is a default limit of 50 domain names. However, this limit can be increased by contacting AWS support.

#### DNS 
is used to convert human friendly urls like http://acloudguru.com to an IP address. Come in two different forms - IPv4 and IPv6
IPv4 - 32 bits, over 4 billion ips, running out
IPv6 - 128 bits

Last word - top level domain name. Controlled by IANA. Registrar assigns domain names under one pr mpre top level domains. InterNIC, a service of ICAAN enforces uniqueness
You can buy domain names directly with AWS, it can take up to 3 days to register 

#### DNS Type SOA - Start Of Authority Record - contains
* The name of the server that supplied the data for the xone
* The administrator of the zone
* The current version of the data file
* The default seconds for the TTL on resource records

#### DNS Type NS - Name Server Record
* Used by top level servers to direct traffice to the content DNS server
* The NS record gives you the SOA. Inside the SOA, you find all the DNS records, for example - 

#### DNS Type 'A' record 
or Address record. It is used by a computer to translate the name of the domain to an IP address

TTL - The length that the DNS record is cached on either the resolving server or the local PC. Default is 48 hours

#### DNS Type CName 
Canonical Name can be used to resolve one domain name to another. 
The DNS protocol does not allow you to create a CNAME record for the top node of a DNS namespace

####Alias Record 
- The same is kinda the same as the cname, but its cannot be used for naked domain names (like http://acloud.guru), it must be either an A record or Alias
Always choose Alias record over CName.
You can create an alias for the top node of a DNS namespace
Their main function is to provide special functionality and integration into AWS services

#### ELBS do not have a pre-defined IPv4 addreess, you resolve to thme using a DNS name

#### Health Checks
You can set Health Checks on individual record sets
If a record fails a health check it will be removed from Route53 till it passes
SNS pushes to notfiy you

The following Routing Policies are avialble with Route53:
* Create recordset within hosted zone. Amazon gives 4 different top level domain names jic. In the record name, you can add your ip address

Simple Routing - One record with multiple IP address, returns all values BUT in random order

Weighted Routing - You can choose the percentage of traffic to be sent to different zones. You create a record set per IP, and set its weight. For ex, if 20 and 30, first one gets 40%, second gets 60%

Latency-based Routing - Route your traffic based on the lowest network latency. You create a latency resource record set for the Amaxon EC2 or ELB resource in each region that hosts your website. 

Failover Routing - When we lose our active server, fails over to the passive one.

Geolocation Routing - Based on the geographic location of your users

Geoproximity Routing - Based on the geographic location of your users as well as your resources. You can use a 'bias' to choose how much traffic. You must use Route53 TrafficFlows

Multivalue Answer Routing - Allows yous to configure multiple values in response to DNS queries. So its like SImple Routing BUT it can perform health checks AND You create separate records for each
