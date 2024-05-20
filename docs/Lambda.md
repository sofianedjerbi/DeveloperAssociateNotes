# Lambda

## Lambda Synchronous Invocations

Result is returned **right away**.  
Error handling must happend **client side**.

*Used by Cognito, Step Functions, ELB, API Gateway, CloudFront...*

## Lambda Asynchronous Invocations

Used when **you don't need to wait for the result**.  
The lambda function **will retry itself**.  
We can define a DLQ if we cannot process it now.  
DLQ can be **either SNS or SQS**.

*Used by S3, SNS, EventBridge, CodeCommit, CodePipeline, SES, CloudFormation...*

## Lambda Integration with ALB

To expose **Lambda as an HTTP(s) endpoint**.  
You can use an ALB or an API Gateway.  
The Lambda **must be registered in a target group**.

*HTTP is converted to JSON and given to Lambda.*

## ALB Multi-eader Values

`/path?name=a&name=b` will be converted as the array `["a", "b"]`

## Lambda Event Source Mapping

**Enables AWS Lambda to receive data** from various sources.  
It automatically **polls these sources to trigger Lambda functions** as needed.

On low traffic you can use a **batch window** to **accumulate records before processing**.  

### Kinesis Data Stream

You can process **multiple batches in parallel**. Up to 10 batches per shard.  
In-order processing is guaranteed for each partition key.

### SQS & SQS Fifo

Long polling is enabled by default. Specify **batch size**.  
Recommended: Set the **visibility timeout** to 6x the timeout of the Lambda function.

To use a DLQ, **configure it on SQS**. DLQ on lambda is only for async invocations.

Lambda **delete the item from the queue if the process is successful**.  

For FIFO queues, Lambda **scales up to the number of the active message groups**.
For Standard queues, Lambda adds **60 more instances per minute to scale up**. Up to **1000 batches in parallel**.

## Context Object

Context JSON object contains data properties about the **Lambda function runtime**.
Lambda runtime converts it to a native object (`dict` in Python).

## Event Object

JSON document containing **data from the invoking service**.  
Lambda runtime converts it to a native object (`dict` in Python).

## Destinations

Send the result of an **asynchronous invocation**.  
**Destinations:** SQS, SNS, Lambda, EventBridge bus.  

A great alternative to the DLQ. Compatible with **Event Source Mapping**.

## Resource Based Policies

Used to give **other account and AWS services permission to use Lambda**.  
Like S3 Bucket Policies.

## Environment variables

We can define **env variables** in the console.

## Logging and Monitoring

Logs are **sent to CloudWatch logs**. **Make sure the Lambda has the permission.**
Same with **CloudWatch Metrics** and **X-Ray**.

X-Ray env variables:
- `_X_AMZN_TRACE_ID`: The tracing header
- `AWS_XRAY_CONTENT_MISSING`: By default, LOG_ERROR
- `AWS_XRAY_DAEMON_ADDRESS`: The XRay Daemon IP:PORT

## Lambda Performance

Ram is from **128Mb to 10Gb in 1Mb increments**. **Increasing RAM also increases CPU**.  
At **1792Mb you get more than 1 vCPU**, and need multithreading to benefit from it.

## Lambda Execution Context

A temporary runtime environment that **initializes any external dependencies for Lambda**.  
Great for DB Connections, HTTP clients...  
The next function invocation can **reuse the contect to reduce execution time**.

```python
# BAD: Initializing the DB connection inside the handler

import os

def get_user_handler(event, context):
    DB_URL = os.getenv("DB_URL")  # Environment variable for DB URL
    db_client = db.connect(DB_URL)  # Establishes a new connection
    user = db_client.get(user_id=event["user_id"])  # Retrieves user data
    return user

# GOOD: Initializing the DB connection outside the handler

import os

DB_URL = os.getenv("DB_URL")  # Environment variable for DB URL
db_client = db.connect(DB_URL)  # Establishes a connection reused across invocations

def get_user_handler(event, context):
    user = db_client.get(user_id=event["user_id"])  # Reuses the existing DB connection
    return user
```

It also includes the `/tmp` directory than can be used for **temporary file processing**.  
To encrypt data in `/tmp`, you should use KMS Data Keys.

## Lambda Layers

Create **custom runtimes**: C++, Rust...  
Create **shared/external dependencies**.  

Max **5 layer per functions**. Up to **250Mb total**.

## Lambda File System Mounting

Lambda can access EFS **when running in a VPC**.  
Configure Lambda to **mount EFS to a local directory during initialization**.  

Limitations: **1 lambda = 1 connection**.

## Lambda Concurrency and Throttling

Up to **1000 concurrent executions**.  
You can set a **reserved concurrency** (limit) at function level.  

Each invocation over the limit will trigger a **throttle**:

- **Synchronous invocation:** `ThrottleError`
- **Asynchronous invocation:** retry up to 6h automatically and then go to the DLQ

The concurrency limit applies to **ALL THE FUNCTIONS IN YOUR ACCOUNT**. 

### Cold Start

Code and context is loaded **at first execution**. This can take some tome.

### Provisioned Concurrency

Concurrency is allocated **before the function is invoked**.  
**Application Auto Scaling** can manage concurrency.
Based on:

- **Version:** A specific, immutable snapshot of a Lambda function's code and configuration.
- **Alias:** A pointer to a Lambda function version that can be updated to refer to different versions.

## Lambda Function Dependencies

You need to **install the packages alongside your code and zip** it: `pip --target`  
If it is more than 50Mb, **upload it to S3**. You can create layers.

## Lambda and CloudFormation

Two options:

- **Inline:** Include the raw code in the template, without any dependency.
- **S3 Zip File:** Store the zip in S3 and refer the path, Bucket, Key, Version.

## Lambda Container Images

Images up to **10GB** in ECR.  
You can pack complex dependencies in a container.  
It needs to implements the **Lambda Runtime API**.

### Best Practices

Use **AWS Base Images**, they are **cached by Lambda**.  
Use **Multi-Stage Builds**: Build from Stable to Frequently changing.  
Most frequently changing things should be at the end of the `Dockerfile`.  
Use a single repository for Functions with Large Layers.


```docker
# Use an image that implements the Lambda Runtime API
FROM amazon/aws-lambda-nodejs:12

# Copy your application code and files
COPY app.js package*.json ./

# Install the dependencies in the container
RUN npm install

# Function to run when the Lambda function is invoked
CMD [ "app.lambdaHandler" ]
```

## Version

A **fixed version of the Lambda function**.

## Aliases

A **pointer to Lambda Function version**.  
Supports **weighted deployment**.

**CANNOT** reference another alias.

*Ex: prod / dev aliases*

## Lambda & CodeDeploy

CodeDeploy can **automate traffic shift** for Lambda Aliases:

- **Linear:** Grow every N minutes
- **Canary:** Try X percent then 100%
- **AllAtOnce:** Immediate

## Lambda Function URL

Dedicated **HTTP(s) endpoint for Lambda**.  
An unique url is generated and never changes.  
Can be applied to any alias or `$LATEST`.

URL: `https://<id>.lambda-url.region.on.aws`

### URL Security

**CORS** and **Resource-based Policy** are available.  
AuthType supports:

- **NONE:** Public
- **AWS_IAM:** IAM is used to authenticate and authorize requests.

## Lambda & CodeGuru

Give **runtime and code insights**.  
The CodeGuru profiler is used.

## Lambda Limits

- **Memory:** `128MB`-`10GB` by `1MB` increments.
- **Execution:** Max 15 minutes.
- **Env VariableS:** `4KB` max
- **`/tmp` capacity:** `512MB` to `10GB`
- **Concurrent executions:** 1000 (can be increased)
- **Deployment:** 50MB zip, 250MB uncompressed.
