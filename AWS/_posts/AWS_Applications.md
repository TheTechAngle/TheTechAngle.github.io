### CloudFront
Edge Locations are endpoints for AWS which are used for caching content. Typically this consists of CLoudFront, Amazon's Content Delivery Network (CDN)

There are many more Edge Locations than Regions. Currently there are over 150 Edge Locations.

A region is a physical location with two or more AZs, an AZ is baiscally one more datacentre.

### Amazon SQS
  
* Distributed fail-safe queue system that stores messages while waiting for a computer to process them.
* Messages can contain upto 256KB of text in any format.
* Queue can act as a buffer if the producer and consumer operate at different speens
* Pull based
* Messages can be stored in the queue for 1 minute to 14 days, default is 4
* Visibility Time - Is the amount of time the message is invisible in the SQS queue after a reader picks up the message.
* If the job is not processed within this time, the message becomes visible again for another reader to process it
* If the job is processed within this time, the message is removed off the queue.
* Visibility Time max time out is 12 hours
* Long polling doesn't return a response until a message arrives in the queue, or it times out. SAVES MONEY
* Short polling return immediately, even if queue is empty
#### Standard Queues 
* Nearly unlimited number of transactions per second.
* Guarantee that a message is delivered at least once.
* Best Effort Ordering.

#### FIFO Queue
* No duplicates
* Strictly ordered
* Limited to 300 transactions per sec (TPS)

### Simple Work Flow Service (SWF)

* Coordinates work across application components, coordination of tasks (invocations of various processing steps, including human actions)
* Workflow executions can last upto 1 year
* Task oriented API (vs SQS msg oriented)
* HTTP based API, can be interacted with any language
* Task is assigned only once, never duplicated
* Keeps track of all the tasks and events in an application.
* SWF ACTORS  
Workflow Starters - initiate a workflow
Deciders - control the flow of tasks, once a task finishes or fails
Activity Workers - carry out the tasks

### SNS (SImple Notification Service)

* A web service that makes it easy to set up, operate and semd notifications from the cloud.
* Allows applications to push messages and immediately allow subscriber to recieve them
* Can deliver notifications directly to mobile devices, via sms or email, or to SQS or to any HTTP end point.
* SES (Simple email service)
* A topic is an access point, on its creation, an amazon resource name is created
* One topic can support multiple endpoint types
* Stored redundantly across multiple instances
* Push based, no polling
* Inexpensive, pay as you go model

### Elastic Transcoder

* Media transcoder in the cloud, to convert media files to different formats..
* Provides preset transcoder based on different devices so you don't have to guess
* Pay by the minutes and resolution you transcode

### API Gateway

* With a few clicks, you can set up an API to access data, logic, functionality from your backend.
* Uers call the API Gateway, which then redirects to the service
* Exposes HTTPS endpoints to define a RESTful API
* Serverless-ly connect to services like Lambda & DynamoDB
* Can send each API endpoint to different target
* Runs efficiently with low cost
* Scales effortlessly
* Throttles requests to prevent attacks
* Maintain multple versions of API (test vs pros)
* Can enable caches for responses for a specified TTL (time to live)
* Deploy to a stage, uses API GAtewat domain by default. Supports AWS certificate manager, free SSL/TLS certs
How do I configure
* Define an API (Container)
* Define resources and nested resources (URL paths)
* For each resource 
  * Select the HTTP methods (verbs)
  * Set security
  * Choose target (such as EC2, Lambda etc)
  * Set request and response transformations
* If you're using Javascript/AJAX that uses multiple domains with API Gateway, enable CORS
* CORS is enforced by the client (browser, but not curl or Postman)

### Kinesis

* For streaming data (generated continuously but in small packets)
* Load and analyze, and build cuotme applications
* Kinesis Streams
  * Stores the streaming data from 24 hrs to 7 days
  * Stored in shards, like a shard for each type of data
  * EC2 can analyze the data in the shards, and store it elsewhere
  * 5 tps for reads, 1000 records per second for writes
  * Max 2MB data read rate per sec, max 1MB total write rate per second
  * Total capacity of the stream is the sum of the capacity of its shards
  
* Kinesis Firehose
  * No persistent storage
  * Soon as it comes in, can trigger a lambda function INSIDE the firehose that processes it and stores it elsewhere (S3)
  * Kinesis Analytics analyses data in both
  
  ### Web Identity Federation and Cognito
  
  * Web Identity Federation: Gives temporary AWS access (mapped to an IAM role) after signing in with a web based identity provider like Amazon or Google, that gives you a token to Cognito, which then provided the access
  * Cognito acts as an Identity broker between your application and web ID providers
  * User Pools are user directories used to manage sign up and sign in functionality for mobile and web applications, handles user registration and authentication.
  * Identity Pool provides temporary AWS credentials to the resources
  * User -> User Pool -> Social Media -> User Pool -JWT Token-> User -JWT Token-> Identity Pool -AWS credential-> User -> AWS
    * Successful authentication provides a JSON Web token (JWT)
  * Uses SNS to let all devices associated with the user's user pool when data changes
