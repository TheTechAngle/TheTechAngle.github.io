### Lambda

* Compute service, where you can upload your code, and that's it.
* Event driven, or in response to HTTP requests using Amazon API Gatewat or API calls made using AWS SDKs
* Supports Node.js, Java, C#, Python, Go, Powershell
* Billed
  * First 1 million requests free, 20 cents per 1 million requests after.
  * How long your code runs
* Cheap, continuously scaling. Scales OUT not up
* Lambda functions are independent, 1 function = 1 event
* Not serverless - RDS, EC2
* Serverless - Dyamodb, S3, API gateway, LambdaAurora RDS serverless
* AWS X-ray debugs serverless applications
* Lambda can do things globally
* Know your triggers!! RDS cannot trigger lambda. 


Building a Serverless Website
