# Serverless Extras

## Step Functions Precisions

Model **workflows as state machines**.  
Written in **JSON**. Start workflow with SDK call, API Gateway, EventBridge...

### Task States

**Used to do some work in state machine**. It can invoke one AWS service.  
It can run an activity, such as EC2 or ECS...

```json
{
  "InvokeLambdaFunction": {
    "Type": "Task",
    "Resource": "arn:aws:states:::lambda:invoke",
    "Parameters": {
      "FunctionName": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME",
      "Payload": {
        "Input.$": "$"
      }
    },
    "Next": "NEXT_STATE",
    "TimeoutSeconds": 300
  }
}
```

States are:

- **Choice State** - Test for a condition to send to a branch (or default branch)
- **Fail or Succeed State** - Stop execution with failure or success
- **Pass State** - Simply pass its input to its output or inject some fixed data, without performing work.
- **Wait State** - Provide a delay for a certain amount of time or until a specified time/date.
- **Map State** - Dynamically iterate steps.
- **Parallel State** - Begin parallel branches of execution.

### Error Handling

**Runtime errors can occur** in case of state machine definition issue, task failure, transient issues...  
We can use **Retry** or **Catch** in the state machine to handle the errors.  

Predefined error codes:

- `States.ALL`: Match any error
- `States.Timeout`: Task ran longer than TimeoutSeconds or no heartbeat received.
- `States.TaskFailed`: Execution Failure.
- `States.Permission`: Insufficient privileges to execute code.

The state can report **its own errors**.

Retry:
```json
"HelloWorld": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME",
    "Retry": [
        {
            "ErrorEquals": ["CustomError"],
            "IntervalSeconds": 1,
            "MaxAttempts": 2,
            "BackoffRate": 2.0
        },
        {
            "ErrorEquals": ["States.TaskFailed"],
            "IntervalSeconds": 30,
            "MaxAttempts": 2,
            "BackoffRate": 2.0
        },
        {
            "ErrorEquals": ["States.ALL"],
            "IntervalSeconds": 5,
            "MaxAttempts": 5,
            "BackoffRate": 2.0
        }
    ],
    "End": true
}
```

Catch:
```json
"HelloWorld": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:....",
    "Catch": [
        {
            "ErrorEquals": ["CustomError"],
            "Next": "CustomErrorFallback"
        },
        {
            "ErrorEquals": ["States.TaskFailed"],
            "Next": "ReservedTypeFallback"
        },
        {
            "ErrorEquals": ["States.ALL"],
            "Next": "NextTask",
            "ResultPath": "$.error"
        }
    ],
    "End": true
},

"CustomErrorFallback": {
    "Type": "Pass",
    "Result": "This is a fallback from a custom lambda function exception",
    "End": true
},
```

ResultPath to add a state output JSON field to the output (can be the error):

```json
"HelloWorld": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:....",
    "Catch": [
        {
            "ErrorEquals": ["States.ALL"],
            "Next": "NextTask",
            "ResultPath": "$.error"
        }
    ],
    "End": true
},

"NextTask": {
    "Type": "Pass",
    "Result": "This is a fallback from a reserved error code",
    "End": true
}
```

### Wait for Task Token

Allow you to **pause Step Functions during a Task until a Task Token is returned**.  
Task might **wait for AWS services, human approval, 3rd party integration...**  
You should add `.waitForTaskToken` to the `Resource` field to wait for the task token to be returned.

```json
"Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken"
```

StepFunctions **pass** a token in the `MessageBody`, and you should call `SendTaskSuccess` API with this token.  

```json
{
    "Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken",
    "Parameters": {
        "QueueUrl": "https://sqs.eu-west-1.amazonaws.com/123456789012/MyQueue",
        "MessageBody": {
            "Input.$": "$",
            "TaskToken.$": "$$.Task.Token"
        }
    }
}
```

## Activity Worker

Performed by an **activity worker**.
Worker pps that can be running on EC2, Lambda, mobile device...

The worker **poll for a task** using the `GetActivityTask` API.  
When it completes its work, it send a response with `SendTaskSuccess` or `SendTaskFailure`.

To keep the task active in step function side, you define `TimeoutSeconds`.  
The worker send `SendTaskHeartBeat` within the time set in `HeartBeatSeconds`.

## Standard vs Express

| Feature               | Standard                                          | Express                                          |
|-----------------------|--------------------------------------------------|--------------------------------------------------|
| **Max. Duration**     | Up to 1 year                                      | Up to 5 minutes                                  |
| **Execution Model**   | Exactly-once Execution                            | Over 100,000 / second                            |
| **Execution Rate**    | Over 2000 / second                                | Over 100,000 / second                            |
| **Execution History** | Up to 90 days or using CloudWatch                 | CloudWatch Logs                                  |
| **Pricing**           | # of State Transitions                            | # of executions, duration, and memory consumption|
| **Use cases**         | Non-idempotent actions (e.g., processing Payments)| IoT data ingestion, streaming data, mobile app backends, ... |
| **Synchronous**       | Not Applicable                                    | At-most once                                     |
| **Asynchronous**      | At-least once                                     | At-least once                                    |


## AppSync Precisions

Create **GraphQL** and **realtime WebSocket** APIs.  
Needs a **GraphQL Schema**.  

Resolvers can be **DynamoDB, Aurora, Opensearch, Lambda, HTTP**.  
Have **IAM, API Key, OIDC, JWT and User Pools** support for security, and **CloudFront**.

## Amplify Precisions

A set of tools to create **mobile and web applications**.

- **Amplify Studio:** Visually build a full-stack app.
- **Amplify CLI:** Configure an Amplify backend with a guided CLI workflow.
- **Amplify Libraries:** Connect your app to existing AWS services.
- **Amplify Hosting:** Host web apps or websites via the AWS content delivery network.

Add authentication support with `amplify add auth`  
Add datastore with `amplify add api` (easy cloud synchronization)  
Add hosting, CI/CD, monitoring with `amplify add hosting`  
Run E2E tests with test commands in `amplify.yml` and generate UI reports.

