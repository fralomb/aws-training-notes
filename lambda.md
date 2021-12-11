# AWS Lambda == heart of Serverless
- lets you run code without provisioning or managing Serverless
- triggers on your behalf in response to events
- scales automatically
- provides built in code monitoring and logging

AWS Serverless platform also include a number of fully managed services tighted intergrated with lambda:
- API Gateway
- Amazon S3, dynamoDB, etc..
- SNS and SQS
- AWS Step functions fo orchestration
- Kinesis and Athena for Analysis
- Developer tools (SAM)

# Typical example
![Example Serverless Architecture](img/example-serverless.png)

# Lambda Features
- developping in lambda is easy, support for different languages
- integrates with and extends other AWS services
- flexible resource and concerrency model: instead of scaling by adding servers, Lambda scales in response to events. You configure memory settings and AWS handles CPU, Network, IO throughput.
- flexible permissions model, use Identity Access Management (IAM) to grant access to the desired resources and gives fine-grained control for invoking functions
- High Availability and fault tolerance
- don't pay for idle

# Permissions
- permission to *trigger* a function
- permission to *interact* with other services

those permissions are managed by AWS IAM through:
- IAM Resource Policy/Function Policy: permissions to invoke function
- IAM Execution Role: controls what the function is allowed to do

## Execution Role
What the function is permitted to do (ex: write to this DynamoDB table..)
The Execution Role is a IAM role that is *selected or created* when you create a Lambda function.
The *IAM policy* defines the actions that the role is allowed to take with the resource (ex: write into the DynamoDB table)
The role need also to include a *Trust Policy* that allows Lambda to *AssumeRole* so that it can take actions from other services.
Lambda provides managed roles with predefined permissions that helps simplify the process of creating an Execution Role => we do not have to create role and policy from scratch
Creatir must have permission for `iam:PassRole`

By default, resource within a VPC are not accessible from within a Lambda function. Lambda runs from its own VPC. To enable the Lambda to access our private PVC we need to provide additional VPC configuration like subnet and security group IDs. AWS uses this information to set an Elastic Network Interface that enable to connect securely to other resources within our private VPC. For this reason IAM execution role needs permission to: create, describe and delete network interfaces.

## Resource/Function Policy
A resource policy is used to tell the Lambda service which events have permission to invoke the  Lambda function. Resource policies also make it easy to grant access to the Lambda function across AWS accounts.
- policy associated with a "push" event source
- created when you add a trigger to a Lambda function
- Allows the event source to take the *lambda:InvokeFunction* action

## Types of Event Sources
An `Envet source` is the entity that publishes an event, the `Lambda function` is the custom code that process those events.
There's lot of different option to trigger a lambda function.
- Data stores: Amazon S3, etc...
- Endpoints: emit events that triggers lambda function ex: Alexa, Api gateway, etc..
- Repositories: Amazon CloudWatch, CloudCommit based on specific action on repositories
- Message services: SES or SNS

Push Events:
- synchronous / Asynchronous (Serivce delivers events directly to function
Polling Events:
- Stream-based, not stream-based: lambda polls from events and delivers to function

when invkoking a lambda function programmatically we have to specify the *invocation type*

## Sync vs Async
- the event source directly triggers the function. when a client makes a request to the API Gateway, expects a response immediatly, the invoking application receives an error if it can't invoke the function. Use the `RequestResponse` api
- the triggering event doesn't wait around for a response. We can enable Dead Letter Queue (DLQ) or discardered the request will be discarded after 3 attemps
  - AWS Lambda destination: route the resoulte of an async invocation to an AWS service without writing code. We have 4 destination options:
    - Lambda function
    - SQS queue
    - SNS topic
    - EventBridge event bus
    we can specify a destination fo `onSuccess` and one for `onFailure`.
  Use the `Event` api

## Polling Events
Events put information into the stream (DynamoDB/Kinesis) or the queue (Amazon SQS). The Lambda funciton polls the stream or queue for changes and if it finds records it will deliver the payload and invoking the function.
For stream-based polling, if there is an error processing the records lambda will retry until the data expires (up to 7 days). Any error blocks lambda from reading any new record from the stream until process successfully or expiration. This because the sequence is important in the stream.
For the queue-based polling, if an invocation fails or times out the failed message comes back in the queue. It will be available for invocation again once the visibility timeout perid expires. Lambda will keep retry the exectution untill success or expiration.

# Lifecycle of a lambda function

When a function is first invoked ana execution environment is launched and bootstraped, this include downloading the code and loading the execution runtime for the environment. For example, if the function is written in nodejs, the nodejs framework needs to be loaded in the container.
Then the function code is executed. Then Lambda freezes the execution environment because we expect that there will be additional invocation in the future. If another invocation is made while the function is in the state, then the request goes through a `warm start` => the code is immediatly executed without have the need of bootstrapping the container.
This behaviour continues as long as the requests comes consistently. If the execution environments is idle for too long (`idle timeout`), then the environment is recycled and another invocation request will require again to pass through the bootstrap phase (`cold start`).

Good practices to take advantage of the Warm Start:
- Store and reference external dependencies locally
- Limit re-initialization of variables or objects at every invocation
- Check if a connection already exist and reuse it
- Add checks whether the local cache still has the data stored previously (500 MB available in the environment)

NOTE: testing from the console there's a cold start every time.

## Provisioned concurrency
`Provisioned concurrency` is a feature that keeps functions initialized and hyper-ready to respond in double-digit milliseconds. This is ideal for implementing interactive services, such as web and mobile backends, latency-sensitive microservices, or synchronous APIs. When you enable Provisioned Concurrency for a function, the Lambda service will initialize the requested number of execution environments so they can be ready to respond to invocations.
For more: [doc](https://aws.amazon.com/blogs/aws/new-provisioned-concurrency-for-lambda-functions/)

