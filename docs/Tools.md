# Developer Tools

## Instance Metadata

Can be used to get **temporary credentials**.  

### IMDSv1

Available at `http://169.254.169.254/latest/meta-data`  
You can retrieve the name, private, public ip, IAM Role, but not the IAM Policy.

### IMDSv2

Get token from `http://169.254.169.254/latest/api/token` with PUT and `X-aws-ec2-metadata-token-ttl-seconds: XXX`  
And curl `http://169.254.169.254/latest/meta-data/<needed info>` with `X-aws-ec2-metadata-token: $token`

## CLI Profiles

Allow to manage **multiple AWS accounts** under one CLI.  
Configure profile: `aws configure --profile <name>`  
Change profile with `--profile <name>`

## CLI MFA

You can enable MFA on CLI with the `STS GetSessionToken` API Call.  
You can get it with `aws sts get-session-token --serial-number <MFA IAM> --token-code <app code>`  
You can then auth with MFA by configuring with the given credentials and adding `aws_session_token` into `~/.aws/.credentials`  

*Note: STS = Security Token Service*

## AWS SDK

Available in Java, .NET, Node.js, PHP, Python *(boto3)*, Go, Ruby, C++.  
**The CLI uses Python**.

## API Rate Limits

You can request an increase by **opening a ticket**.  
For consistent errors, request a rate limit increase with a **ticket**.

### Intermittent Errors -> Exponential Backoff

If you get **ThrottlingException** intermittently, use exponential backoff.  
A **retry mechanism** is already included in AWS SDK.  
But with **AWS API** you need to implement **the retries** yourself on **5xx** errors.  
You **double the wait time on every retry**. (exponential)

## Service Quotas

Tou can request an increase by using the **Service Quotas API**

## Credentials Provider Chain

**Don't store credentials in your code.**  
Use ENV variables outside AWS.

### CLI

**1/** CLI options *(--region, --output, --file)*  
**2/** ENV variables *AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN*  
**3/** ~/.aws/credentials  
**4/** ~/.aws/config  
**5/** Container credentials  
**6/** Instance profile credentials

### SDK

**1/** System Properties (aws.secretKey, aws.accessKeyId)  
**2/** Environment variables  
**3/** ~/.aws/credentials  
**4/** Container credentials
**5/** Instance profile credentials

## Sign API Requests

The process is **SigV4**.  
On SDK or CLI, requests are signed **automatically**.  
With the API the you need to **send the signature in every request**.

You can send the signature as an **HTTP Header** or as a **Query String**.

### HTTP Header

```
GET https://iam.amazonaws.com/?Action=ListUsers&Version=2010-05-08 HTTP/1.1
Authorization: AWS4-HMAC-SHA256 Credential=AKIDEXAMPLE/20150830/us-east-1/iam/aws4_request,
SignedHeaders=content-type;host;x-amz-date,
Signature=56d27d79c15b13162d9279b85cfba6789a8edb4c82c400e06b59246af2b5d7
content-type: application/x-www-form-urlencoded; charset=utf-8
host: iam.amazonaws.com
x-amz-date: 20150830T123600Z
```

### Query String

```
GET https://iam.amazonaws.com/?Action=ListUsers&Version=2010-05-08
X-Amz-Algorithm=AWS4-HMAC-SHA256
X-Amz-Credential=AKIDEXAMPLE%2F20150830%2Fus-east-1%2Fiam%2Faws4_request
X-Amz-Date=20150830T123600Z&X-Amz-Expires=600&X-Amz-SignedHeaders=content-type%3Bhost
X-Amz-Signature=37ac2f4fde00b0ac9bd9eadeb459b1bbee224158d66e7ae5fcadb70b2d181d02
content-type: application/x-www-form-urlencoded; charset=utf-8
host: iam.amazonaws.com
```

### SigV4 Process  

Not required for the exam.  
Read it [here](https://towardsaws.com/aws-sigv4-in-3-mins-c324d20f19cf).
