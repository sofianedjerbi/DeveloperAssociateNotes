# CloudFormation

## CloudFormation

You can't change a template, you should replace with **a new version**.  
CloudFront add **stack tag / resource tag**.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: A simple AWS CloudFormation template to create an EC2 instance.

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0ff8a91507f77f867  # Ensure this AMI is available in your AWS region
      KeyName: my-key-pair  # Replace with your key pair name
      SecurityGroups:
        - default  # Replace with your security group if different
      Tags:
        - Key: Name
          Value: MySimpleInstance

Outputs:
  InstanceId:
    Description: The Instance ID of the newly created EC2 instance.
    Value: !Ref MyEC2Instance
  PublicIP:
    Description: The public IP address of the EC2 instance.
    Value: !GetAtt MyEC2Instance.PublicIp
```

## Resource

They are **AWS components**.  
Type format: `service-provider::service-name::data-type-name`  
Type example: `AWS::EC2::Instance`

## Parameters

They are a way to **provite inputs** in templates, for reusability.  
Should be used if **something could be changed in the future**.  
They are **not just strings**. They can be int, allowed values, NoEcho, values from SSM...

To use a parameter you should use `!Ref <name>`, also works with resources.

```yaml
Parameters:
  InstanceType:
    Type: String
    Description: Enter the instance type for your EC2 instance (e.g., t2.micro, t3.medium, etc.)
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
      - t3.micro
      - t3.small
      - t3.medium

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0ff8a91507f77f867
```

## Pseudo Parameters

Predefined parameters.  
Example: `AWS::AccountId`, `AWS::Region`

## Mappings

Mappings are **fixed variables** within your template.  
All the values are **hardcoded**.

```yaml
Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0ff8a91507f77f867
      HVMG2: ami-0a584ac55a7631c0c
    us-west-1:
      HVM64: ami-0bdb828fd58c52235
      HVMG2: ami-066ee5fd4a9ef77f1

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", HVM64]
      InstanceType: t2.micro
```

## Outputs

An optional section that defines **values that can be imported in other stacks**. *(should be exported)*  
They are visible in the console / CLI.

Example usage:

```yaml
Outputs:
  BucketName:
    Description: The name of the S3 bucket
    Value: !Ref MyBucket
    Export:
      Name: BucketName
```

In another template:

```yaml
Resources:
  MyImportedBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !ImportValue BucketName
```

You **can't delete the underlying stack until all the references are deleted**.

## Conditions

Used to **control the creation of resources or outputs**.

```yaml
Parameters:
  CreateBucket:
    Type: String
    Default: 'false'

Conditions:
  CreateProdResources: !Equals [ !Red EnvType, prod ]

Resources:
  ConditionalBucket:
    Type: AWS::S3::Bucket
    Condition: ShouldCreateBucket
```

Available logical functions: **equals, and, or, if, not**

*Example conditions: Environment test, prod, region...*

## Intrinsic Functions

### To Know:
- **`Ref`**: Retrieves the value of a specified parameter or resource.
- **`GetAtt`**: Fetches the value of an attribute from a resource in the template.
- **`FindInMap`**: Returns a named value from a specifically defined map.
- **`ImportValue`**: Imports values exported from other stacks.
- **`Base64`**: Encodes content to Base64 format.
- **Condition Functions** (like `If`, `Equals`): Evaluate conditions to control resource creation and properties.

### Others:
- **`Join`**: Concatenates multiple strings into a single string with a specified delimiter.
- **`Sub`**: Substitutes variables in an input string with values that you specify.
- **`ForEach`**: Applies a template fragment to each item in a list or map.
- **`ToJsonString`**: Converts an object or value to a JSON string.
- **`Cidr`**: Divides an IP network into subnets according to a specified count and size.
- **`GetAZs`**: Returns a list of Availability Zones for a specified region.
- **`Select`**: Retrieves a single object from a list of objects by index.
- **`Split`**: Splits a string into a list of strings based on a delimiter.
- **`Transform`**: Specifies a macro to perform custom processing on parts of a stack template.
- **`Length`**: Returns the length of a string or list.

## Rollbacks

If Stack creation fails, **everythig is getting deleted by default**.  
If the update fails, the stack will **roll back to the last known working state**.  
In case of rollback failure you can use the **continue update rollback API**.

## Service Role

An **IAM role that allows CloudFormation to create/delete/update stack resources**.   
It allow users to create resources **even if they don't have the permission**.  
User must have **iam:PassRole**.

## Capabilities

- `CAPABILITY_IAM` & `CAPABILITY_NAMED_IAM`: Necessary to enable when you want to change IAM resources.  
- `CAPABILITY_AUTO_EXPAND`: Necessary when CloudFormation includes Macros or Nested Stacks.
- `InsufficientCapabilitiesException`: When the required capabilities are not given.

## Deletion Policy

Control what happens when the **template / stack is deleted**.  

- `Delete`: Default. Do not work on an non-empty S3 bucket.
- `Retain`: Preserve the resource.
- `Snapshot`: Create a final snapshot before deleting the resource.

## Stack Policy

During a CloudFormation Stack update, all update actions are allowed by default.  
A stack policy is a JSON document that defines **the update actions that are allowed**.

Protect resources against updates.

## Termination Protection

An option to **protect against deletion**.

## Custom Resources

Defines resources that are **not supported by CloudFront** / are on premises.  
Also used when you want to run **custom scripts** during *update / create / delete*.  

Define it with `Custom::ResourceName`.

```yaml
Description: Template to create an S3 bucket that empties itself before getting deleted.

Resources:
  # IAM Role for Lambda Function
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties: 
      AssumeRolePolicyDocument: 
        Version: '2012-10-17'
        Statement: 
          - Effect: Allow
            Principal: 
              Service: 
                - lambda.amazonaws.com
            Action: 
              - sts:AssumeRole
      Policies: 
        - PolicyName: S3AccessPolicy
          PolicyDocument: 
            Version: '2012-10-17'
            Statement: 
              - Effect: Allow
                Action: 
                  - s3:ListBucket
                  - s3:DeleteObject
                Resource: 
                  - arn:aws:s3:::my-bucket-name
                  - arn:aws:s3:::my-bucket-name/*

  # Lambda Function to empty S3 bucket
  EmptyS3BucketFunction:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          import json
          import boto3
          import cfnresponse

          def handler(event, context):
              s3 = boto3.resource('s3')
              bucket_name = event['ResourceProperties']['BucketName']
              bucket = s3.Bucket(bucket_name)
              
              if event['RequestType'] == 'Delete':
                  try:
                      for obj in bucket.objects.all():
                          obj.delete()
                      for obj in bucket.object_versions.all():
                          obj.delete()
                      cfnresponse.send(event, context, cfnresponse.SUCCESS, {})
                  except Exception as e:
                      cfnresponse.send(event, context, cfnresponse.FAILED, {'Message': str(e)})
              else:
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, {})

      Runtime: python3.9
      Timeout: 300

  # S3 Bucket
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-bucket-name

  # Custom Resource to invoke Lambda during stack deletion
  EmptyS3BucketCustomResource:
    Type: Custom::EmptyS3Bucket
    Properties:
      ServiceToken: !GetAtt EmptyS3BucketFunction.Arn
      BucketName: !Ref MyS3Bucket

Outputs:
  S3BucketName:
    Description: Name of the S3 bucket
    Value: !Ref MyS3Bucket
```

You can use this to delete content from a S3 bucket in order to delete it.

## StackSets

Allow you to deploy/update/delete a stack **across multiple accounts and regions**.  

