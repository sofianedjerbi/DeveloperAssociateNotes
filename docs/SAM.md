# Serverless Application Model

## AWS SAM

Used to **generate complex CloudFormation** from simple YAML files.  
It is focused on **serverless** applications.  
It can help you run **serverless services locally**.

You use `sam deploy` to package and deploy.

## SAM Accelerate

**Reduce latency** while deploying.
`sam sync` is going to synchronize your local template to AWS.  
It can bypass CloudFormation.

Parameters:

- `sam sync` (no options)
  - Synchronizes code and infrastructure
- `sam sync --code`
  - Synchronizes code only, bypassing infrastructure updates (updates in seconds)
- `sam sync --code --resource AWS::Serverless::Function`
  - Synchronizes all Lambda functions and dependencies
- `sam sync --code --resource-id HelloWorldLambdaFunction`
  - Synchronizes a specific resource by ID
- `sam sync --watch`
  - Monitors file changes for automatic synchronization
    - Uses `sam sync` for configuration changes
    - Uses `sam sync --code` for code changes

## SAM YAML

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: A starter AWS Lambda function.
Parameters:
  IdentityNameParameter:
    Type: String
Resources:
  Table:
  Type: 'AWS::Serverless::SimpleTable'
  Properties:
    PrimaryKey:
      Name: greeting
      Type: String
    ProvisionedThroughput:
      ReadCapacityUnits: 2
      WriteCapacityUnits: 2
  helloworldpython3:
    Type: 'AWS::Serverless::Function'
    Properties:
      Handler: lambda_function.lambda_handler # Path to the python method in src/
      Runtime: python3.9
      CodeUri: .
      Description: A starter AWS Lambda function.
      MemorySize: 128
      Timeout: 3
      Environment:
        Variables: # Env var
          TABLE_NAME: !Ref Table
          REGION_NAME: !Ref AWS::Region
      Events:
        HelloWorldSAMAPI:
          Type: Api # API Gateway config
          Properrties:
            Path: /hello
            Method: GET
      Policies:
        - DynamoDBCrudPolicy:
          TableName: !Ref Table
      IdentityName: !Ref IdentityNameParameter

```

## Deployment Commands

```bash
# Create a bucket for source code
aws s3 mb s3://code-sam

# This is equivalent to `sam package ...`
# It add s3 code links to a new template
aws cloudformation package --s3-bucket stephane-code-sam --template-file template.yaml --output-template-file gen/template-generated.yaml

# Deploy
aws cloudformation deploy --template-file gen/template-generated.yaml --stack-name hello-world-sam --capabilities CAPABILITY_IAM
```

## Policy Templates

List of **templates to apply permissions to Lambda Functions**.  
Example:
- **S3ReadPolicy:** Give Read Only permissions to objects in S3
- **SQSPollerPolicy:** Allows to poll an SQS queue
- **DynamoDBCrudPolicy:** CRUD = Create Read Update Delete

## SAM & CodeDeploy

SAM **natively uses CodeDeploy to update Lambdas**.  
It has a **traffic shifting feature**, **pre and post traffic hooks** to validate deployment.  
Automated rollback with **CloudWatch Alarms**.

```yaml
Resources:
  MyLambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs12.x
      CodeUri: s3://bucket/code.zip
      AutoPublishAlias: live
      DeploymentPreference:
        Type: Canary10Percent10Minutes
        Alarms:
          # A list of alarms that you want to monitor
          - !Ref AliasErrorMetricGreaterThanZeroAlarm
          - !Ref LatestVersionErrorMetricGreaterThanZeroAlarm
        Hooks:
          # Validation Lambda functions that are run before & after traffic shifting
          PreTraffic: !Ref PreTrafficLambdaFunction
          PostTraffic: !Ref PostTrafficLambdaFunction
```

## Project Setup

```bash
# Init directory
sam init --runtime python3.7

# Go into config
cd sam-app

# Build resource, download dependencies and build artifacts
sam build

# Deploy resources and create samconfig.toml with deployment configuration
sam deploy --guided
```

## SAM Local Capabilities

- `sam local start-lambda`: Local AWS lambda.
- `sam local invoke`: Invoke lambda function, should use --profile for AWS calls to the right environment.
- `sam local start-api`: Local API Gateway.
- `sam local generate-event`: Generate fake events for Lambda.

You can use **AWS Toolkits** to debug your lambda functions line per line.

## Sam with Multiple Environments

Done in `samconfig.toml` and used in `sam deploy --config-env dev/prod`:

```toml
version = 0.1

[dev.deploy.parameters]
stack_name = "my-dev-stack"
s3_bucket = "XXXXX-dev"
s3_prefix = "XXXXX/dev"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=Development"

[prod.deploy.parameters]
stack_name = "my-prod-stack"
s3_bucket = "XXXXX-prod"
s3_prefix = "XXXXX/prod"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"
parameter_overrides = "Environment=Production"

[dev.sync.parameters]
watch = true

[prod.sync.parameters]
watch = false
```

## AWS SAR (Serverless Application Repository)

Allows users to store, share, and deploy serverless applications compatible with AWS SAM.
