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
The role need also to include a *Trust Policy* taht allows Lambda to *AssumeRole* so that it can take actions from other services.
Lambda provides managed roles with predefined permissions that helps simplify the process of creating an Execution Role => we do not have to create role and policy from scratch

By default, resource within a VPC are not accessible from within a Lambda function. Lambda runs from its own VPC. To enable the Lambda to access our private PVC we need to provide additional VPC configuration like subnet and security group IDs. AWS uses this information to set an Elastic Network Interface that enable to connect securely to other resources within our private VPC. For this reason IAM execution role needs permission to: create, describe and delete network interfaces.

## Resource/Function Policy
A resource policy is used to tell the Lambda service which events have permission to invoke the  Lambda function. Resource policies also make it easy to grant access to the Lambda function across AWS accounts.
- policy associated with a "push" event source
- created when you add a trigger to a Lambda function
- Allows the event source to take the *lambda:InvokeFunction* action
