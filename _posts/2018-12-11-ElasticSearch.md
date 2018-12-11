---
layout: post
title: Elasticsearch 
---

### What is Elasticsearch?

Let's go over the basics of Elasticsearch before we dive into Amazon's Elastic Search Service. 

Elasticsearch stores textual data in JSON document format. This is a schema-less database, a document oriented database
that uses some defaults to index the data unless you specify one. You can access all of its features via a REST API.

A cluster is a collection of servers that work together and share the data. A single server is called a node.
A shard is a subset of documents of an index, it can be divided into many shards. 

### Amazon's Elasticsearch

Amazon ES allows you to deploy, manage and scale elasticsearch clusters in the AWS Cloud. It gives you access to ElasticSearch APIs and a console to set up and configure a domain, which is synonymous with an ElasticSearch cluster.

##### Features (as listed on AWS) 
* AWS Identity and Access Management (IAM) access control
* Easy integration with Amazon VPC and VPC security groups
* Encryption of data at rest and node-to-node encryption
* Data visualization using Kibana (Something we will expand upon later) 

Waaiit what does all of that MEAN?
Let's go one by one. 

* IAM is a service that lets you manage user and group permissions and authorizations to other AWS resources  
* Amazon VPC or the Amazon Virtual Private Cloud gives you access to a logically 'isolated' network of servers, giving the user more control over the security of their cloud  
* This one is fairly straightforward!
* Kibana lets you visualize your Elasticsearch data and provides real time analysis. This is a tool I'll exapnd upon a bit more later.




