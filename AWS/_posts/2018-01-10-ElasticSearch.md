---
layout: post
title: (AWS) Using Elasticsearch 
---

#### What is Elasticsearch?

Let's go over the basics of Elasticsearch before we dive into Amazon's Elastic Search Service. 

Elasticsearch stores textual data in JSON document format. This is a schema-less database, a document oriented database
that uses some defaults to index the data unless you specify one. You can access all of its features via a REST API.

A cluster is a collection of servers that work together and share the data. A single server is called a node.
A shard is a subset of documents of an index, it can be divided into many shards. 

#### Amazon's Elasticsearch

Amazon ES allows you to deploy, manage and scale elasticsearch clusters in the AWS Cloud. It gives you access to ElasticSearch APIs and a console to set up and configure a domain, which is synonymous with an ElasticSearch cluster.

##### Features (as listed on AWS) 
* AWS Identity and Access Management (IAM) access control
* Easy integration with Amazon VPC and VPC security groups
* Encryption of data at rest and node-to-node encryption
* Data visualization using Kibana, which is a JavaScript based UI which lets you extract and analyze data (Something we will expand upon later) 

Waaiit what does all of that MEAN?
Let's go one by one. 

* IAM is a service that lets you manage user and group permissions and authorizations to other AWS resources. An IAM policy lets you grant access to a cluster's endpoint, allowing or denying Actions (HTTP methods) against Resources (the domain endpoint, indices, and API call to
calls to Amazon ES)  
* Amazon VPC or the Amazon Virtual Private Cloud gives you access to a logically 'isolated' network of servers, giving the user more control over the security of their cloud  
* This one is fairly straightforward!
* Kibana lets you visualize your Elasticsearch data and provides real time analysis. This is a tool I'll exapnd upon a bit more later.


#### Setting it up

##### 1) Setting up the cluster
I will skip the actual steps, AWS has them documented well enough. I'll just write about the things I found interesting as I went through the documentation.

* Split brain issue - Clusters have a dedicated number of master nodes. In case of a partition, one part of the network may elect another master node even though the other one is alive and healthy but unreachable. This is know as a split brain issue.
It is thus recommended N/2+1 nodes be needed to elect a master in a cluster. AWS suggests an odd number of instances in a cluster to avoid a split brain issue.

* You can opt for the even distribution of shards across two zones in the same region.

* ElasticSearch stores data as JSON documents. These are organized as types based on categories you choose, and assigned a unique ID. Indices default to 5 primary shards and one replica.

##### 2) Controlling access
I found [this blog post on aws](https://aws.amazon.com/blogs/security/how-to-control-access-to-your-amazon-elasticsearch-service-domain/)
to learn about controlling access to the elasticsearch service. To summarize and quote from Jon -

* Both the entity and the resource is covered by some sort of access policy. The union of these policies decideds on whether the entity calling the resource is allowed access to it.
* Authentication of requests (for both policies) can occur in the following two ways
    * Based on the originating IP Address
    * Based on the originating Principal, or signing the request using Signature Version 4.
    * Both the above two can be configured as demoed in blog on the modify access policy tab under the domain/cluster information
    * A dummy example for a resource based policy would look like this  
    
     Use the "Effect" (Allow) for the "Principal" (user alloweduser) for "Action" (here es:* stands for any Amazon ES-related action) on the "Resource" (somedomain).
    ```
    {
      "Version": "20xx-10-17",
      "Statement": [{
        "Effect": "Allow", 
        "Principal": {
          "AWS": "arn:aws:iam::111111111111:user/alloweduser"
        },      
        "Action": "es:*", 
        "Resource": "arn:aws:es:us-west-2:111111111111:domain/somedomain/*" 
      }] 
    }
###### 3) Principal based authentication 
There are three main parts in the process of sending a Signature Version 4 request to an Amazon endpoint - generateRequest(), performSigningSteps(), and sendRequest().
Java has an AWS SDK for this (AWS4Signer). By using the SDK, all you have to do is set up th e request, provide necessary credentials and then use the sign method in the AWS4Signer class. Ues the AWSDefaultCredentialsProvider to avoid hard coding your credentials.

###### 4) Keeping Kibana in mind
Kibana doesn't support ES access via signed requests. 
When using a IP based access policy, if Kibana has a large number of users, whitelisting all these IPs will be a pain. 
We will need to set up a proxy between Amazon ES and Kibana, which will recieve requests from Kibana and then forward it to Amazon ES. This was, only the proxy IP will need to be whitelisted.
If the application code can issue Signature Version 4 signed request, then no proxy is required for Principal based access policies.

Recently though, Amazon has released the [Amazon Cognito Authentication](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-cognito-auth.html) for Kibana, which should be easier to use and control access with.

##### Connecting to Elasticsearch behind a Virtual Private Cloud

I spent hours and hours reading up about our use case. Our elasticsearch cluster is behind a virtual private cloud making it really really hard.  When we placed our ES within the VPC, the endpoint that was created was NOT a [VPC Endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html) but an [Elasticsearch endpoint within a VPC](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-vpc.html), which are two wildly different things. The former allows access into a VPC, the latter allows access to ES within a VPC. So then, we had to give our Lambda function access to the VPC
 
This link summarizes it pretty well (https://medium.com/@lichnguyen/setup-aws-lambda-permission-to-query-aws-elasticsearch-and-send-notification-to-aws-sns-126ed2181b84) and there are additional AWS docs you can read to learn more (https://docs.aws.amazon.com/lambda/latest/dg/vpc.html) 
 
We had to add the AWSLambdaVPCAccessExecutionRole to the Lambda role, and the rest of the EC2 roles. You’ll have to go the Lambda function on the console and choose VPC access, choose the VPC, and give it subnets and security groups. Note that Elasticsearch places an endpoint in one subnet in each availability zone, so make sure you choose these subnets in the Lambda section. Follow those two posts. Once this is done, the lambda function should be able to connect to ES with a elasticsearch client code.
 
##### Elasticsearch Indexes and Types

An index is stored in a set of shards which are themselves Lucene indices. 

###### 1) Lucene Indices
A Lucene index is an inverted index, its an index over a dynamic collection of documents and provides rapid updates to the said collection.
It indexs 'terms' in the document, providing a mapping from terms to documents.

###### 1) Elasticsearch Indices
An index is stored as a set of shards, which are themselves Lucene indices. Lucene indices have a small overhead in terms of disk space
memory usage and file descriptors used, so a single large index is more efficient than several small indices


##### Java HTTP Client

There are several clients to connect to Elasticsearch in Java, and a native client provided by Elasticsearch. What does 'native' really mean though? 
A native client will often communicate in the internode protocol of Elasticsearch. The native client provided by Elasticsearch doesn't use REST API and connects to the cluster as a cluster node. The node doesn't contain any data
but remains cluster aware. This way, the client is aware of which nodes documents are saved at and can route requests accordingly.



We used the [Jest](https://github.com/searchbox-io/Jest/tree/master/jest#comparison-to-native-api) library for setting up a Elasticsearch client in Java.
Jest is a lightweight client, with an API that is similar to the native client but is REST based.I had the operations we wanted - to create/delete an index, and more importantly
bulk operations to add and delete data in bulk, and a 'delete by query' operation to allow deletions of rows identified by some field other than the id. The README in the link
I have given above contains everything required to set it up and the api details.

###### Create Index
`JestResult jestResult = jestClient.execute(new CreateIndex.Builder(index).build());`

###### Bulk Insert 
```
if(bulkBuilder == null){
   bulkBuilder = new Bulk.Builder()
                    .defaultIndex("someIndex")
                    .defaultType("someType");
}
// Add the 'action' to add the new row to the bulk builder
bulkBuilder.addAction(new Index.Builder(document).build());
count++;
// If the number of actions have reached the batchsize you have set,
// then upload the data and reset it to ready it for more actions
if(count%batchSize == 0) {
   JestResult jestResult = jestClient.execute(bulkBuilder.build());
   count = 0;
   bulk = new Bulk.Builder()
             .defaultIndex("someIndex")
             .defaultType("someType");
   log.error(jestResult.getErrorMessage());
}
```

###### Delete Index
`JestResult jestResult = jestClient.execute(new DeleteIndex.Builder(index).build());`

###### Delete By Query
```
DeleteByQuery deleteAction = new DeleteByQuery.Builder("{\"field\":\""+fieldToBeDeleted+"\"}")
                .addIndex("someIndex")
                .addType("someType")
                .build();
JestResult jestResult = jestClient.execute(deleteAction);
log.error(jestResult.getErrorMessage());
```



