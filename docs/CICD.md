# CI/CD

## CodeCommit Precisions

Repos are encrypted with **KMS encryption** and **in-transit** *(HTTPS/SSH)*.  
There are events (push, comments...) that can trigger **SNS** notifications.  
To use `git` you must **generate Git credentials from IAM**.  
Supports **SSH keys, Git Credentials and AWS Access Keys**.

*Note: Do NOT supports IAM user / password*

## CodePipeline Precisions

A **visual workflow tool** for CI/CD.  
Compatible with almost every CI/CD tools **including 3rd party tools**...

Can leverage **S3 to store artifacts** and use them in the build / deploy process.

- **Actions**: Tasks like build, deploy, test.
- **Action Groups**: Sets of simultaneous actions.
- **Stages**: Phases like build, test, deploy. Contains action groups.

## CodeBuild Precisions

Instructions file is `buildspec.yml` in the repository.  
Output logs can be stored in **S3 / CloudWatch Logs**, supports events too.  

**Supported Environments:** *Java / Ruby / Python / Go / Node.js / Android / .NET Core / PHP*  
Docker is supported for other environments.

CodeBuild containers can run **inside a VPC**. Container is deleted at the end of the build.

```yaml
version: 0.2

env:
  variables:
    JAVA_HOME: "/usr/lib/jvm/java-8-openjdk-amd64"
  parameter-store:
    LOGIN_PASSWORD: /CodeBuild/dockerLoginPassword

phases:
  install:
    commands:
      - echo "Entered the install phase..."
      - apt-get update -y
      - apt-get install -y maven
  pre_build:
    commands:
      - echo "Entered the pre_build phase..."
      - docker login -u User -p $LOGIN_PASSWORD
  build:
    commands:
      - echo "Entered the build phase..."
      - echo "Build started on `date`"
      - mvn install
  post_build:
    commands:
      - echo "Entered the post_build phase..."
      - echo "Build completed on `date`"

artifacts:
  files:
    - target/messageUtil-1.0.jar

cache:
  paths:
    - "/root/.m2/**/*"
```

## CodeDeploy Precisions

Supports **auto rollback**, **EC2 deployment**...  
Deployment file is `appspec.yml` in the repository.

You can setup the following lifecycle events:
```
ApplicationStop
DownloadBundle
BeforeInstall
Install
AfterInstall
ApplicationStart
ValidateService
BeforeBlockTraffic
BlockTraffic
AfterBlockTraffic
BeforeAllowTraffic
AllowTraffic
AfterAllowTraffic
```

Supports **in-place deployments and blue/green deployments**.  
Must run **CodeDeploy Agent** on target instances.

Deployment speed:
- `AllAtOnce`
- `HalfAtATime`
- `OneAtATime`
- `Custom`

**In-place deployment:** **Stop applications** and recreate them  
**Blue/Green deployment:** **Create new apps** and stop old apps

CodeDeploy supports **traffic shift for Lambda Aliases**. *(Linear / Canary / AllAtOnce)*  
For **ECS** there is only **blue/green** deployments. *(Linear / Canary / AllAtOnce)*

You can **deploy to an ASG**, supports **in-place and blue/green**.  
For blue/green, **a new ASG is created**, and you **should use an ELB**.

### Rollbacks / Redeploy

You can **manually or automatically do rollbacks**.  
If a rollback happens, **CodeDeploy will redeploy** the last known good revision **in a new deployment**.

## CodeStar / CodeCatalyst Precisions

A global dashboard.
Supports **all CI/CD services** *(Github, CodeDeploy, CloudFormation, CodeBuild...)*  
Issue tracking integrated with **JIRA / Github**.  
Supports **Code9** as an IDE.

Supports **C#, Go, Ruby, Java, Node.js, PHP, Python, HTML**.

It is **free**.

## CodeArtifact Precisions

Supports proxying / caching requests to **public artifact repositories**.  
Works with **npm, pip, NuGet, Maven**.

You can **control used packages**.  
Artifact creation/deletion can trigger events. *(ex: rebuild app with updated dependency)*

Supports **IAM and Resource Policies**.

## CodeGuru Precisions

Works with an **agent**, configuration is:

- `MaxStackDepth`: Depth level of the research into the stacktrace
- `MemoryUsageLimitPercent`
- `MinimumTimeForReportingInMilliseconds`: Minimum time between two sending reports
- `ReportingIntervalInMilliseconds`: Reporting interval used to report profiles
- `SamplingIntervalInMilliseconds`

It **creates an EC2 instance** with Cloud9 on it.

