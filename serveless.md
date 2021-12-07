# Building and deploying lambda functions

- write the code in different languages
- deploy the lambda to AWS lambda
- the code will be run in response to some "events"

# Serverless meaning ?

- no server management: no update or patching
- flexible scaling: either automatically scaled or define capacity in terms of total amount of work being done
- automated high availability and fault tollerance: spread across multiple availability zones automatically
- no idle capacity: charged only for the requests performed, only pay for the capacity that we use


# cost savings
- never pay for idle => infrastructure cost savings
- operations: host configuration, machine images, etc...
- reduced time to market: trivial to deploy high scaled application in short time; agility enhancement

# components
- requests (event source)
- Business logic (handler => AWS Lambda function)
- Backend (service that the handler interact with: Database, third party API, etc...)

# From monolith to Managed Services Connected by functions
there are a number of other Services that are part of the broader toolset that are critical to build a serverless architecture
- Amazon API Gateway: to manage a REST API
- Amazon Dynamo DB or object store Amazon S3
- Amazon Kinesis: to stream data from external systems
- Amazon SQS and SNS: to manage messaging flows
- Aws Step Functions: to combine multiple Aws Lambda functions into applications and microservices without having to write code to manage parallel processes, error handling, timeout, retry, etc...
- AWS Develop Tools: to help with deployment of the code

# Event-oriented architecture
In a Monolith architecture we are responsible for all the components: db, services, etc..
In a Serverless architecture we manage services connected together with decoupled lambda functions that implement a specific business logic.

Instead of focusing on `what's the data that i'm trying to store and what operations do i need to perform against that?` you focus on `what are the events that trigger something on my system?`.
For example, as the data in the DynamoDB table changes, those `state changes` become events as well.
This allows to write very loselly-coupled systems and integrate the data with different systems.

# Event sources:
- Data stores: S3, DynamoDB, Kinesis, etc...
- Endpoints: Alexa, API Gateway, AWS IOT Core, AWS Step Functions
- Repositories: CloudFormation, CloudWatch, CodeCommit, etc...
- Event/Message services: Amazon SES, SQS, SNS, Cron events

# Parallelization
The serverless platform allows you to scale the application horizontally and parallelize tasks much more effectivelly rather than a monolith application.
With serverless we do not have to overprovision the system!

Many of the tasks could often break down into parallelizable sub-tasks.
A 1h long task can be break into 360 x 10 second tasks. Result: same amount of work that used to last 1h in only 10s of workload time and the cost of executing the task in either those two models is roughly equivalent.

=> pywren: allow to run a python script on thousands of cores simultaneously!
  pywren is an open source project that allows you to do extremely high throughput computing jobs using Lambda as the compute engine behind the scenes. This gives you the ability to write a very simple python script that runs on your local machine that then delegates out to Lambda functions in order to massively scale out and parallelize the jobs that you're doing.

# Manage Environments and Deployments
Normally we have a prod environment, a staging and maybe a dev. Because mantaining different evnironments has a cost. The non-prod environment will not have much traffic and as a result we end up paying without not that much benefits.
In a serverless architecture, every developer can have its own environment!!

From the CI/CD perspective we can create individual demo environment for every feature branch. We can use frameworks like AWS Serverless Application Model (SAM) to automate the whole CI/CD process.

