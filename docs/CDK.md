# Cloud Develoment Kit

## CDK

Alloy to **define infrastructure using a programming language**: JS / TS, Java, .NET, Python
The code is **compiled into a CloudFromation template**.  

You can then **deploy infrastructure and application runtime code together**.

```java
export class MyEcsConstructStack extends core.Stack {
    constructor(scope: core.App, id: string, props?: core.StackProps) {
        super(scope, id, props);

        const vpc = new ec2.Vpc(this, "MyVpc", {
            maxAzs: 2 // Default is all AZs in region
        });

        const cluster = new ecs.Cluster(this, "MyCluster", {
            vpc: vpc
        });

        // Create a load-balanced Fargate service and make it public
        new ecs_patterns.ApplicationLoadBalancedFargateService(this, "MyFargateService", {
            cluster: cluster, // Required
            cpu: 512, // Default is 256
            desiredCount: 6, // Default is 1
            taskImageOptions: { 
                image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample")
            },
            memoryLimitMiB: 2048, // Default is 512
            publicLoadBalancer: true // Default is false
        });
    }
}
```

You can use **SAM CLI to test CDK apps**, `cdk synth` create a CloudFormation Template.  
Then, `sam local invoke -t CDKStack.template.json function` to test your Lambda function.

## CDK Commands

- `cmd npm install -g aws-cdk-lib` - Install the CDK CLI and libraries
- `cmd cdk init app` - Create a new CDK project from a specified template
- `cmd cdk synth` - Synthesizes and prints the CloudFormation template
- `cmd cdk bootstrap` - Deploys the CDK Toolkit staging Stack and provision needed resources for CDK 
- `cmd cdk deploy` - Deploy the Stack(s)
- `cmd cdk diff` - View differences of local CDK and deployed Stack
- `cmd cdk destroy` - Destroy the Stack(s)

## CDK Constructs

It is a component that **have everything CDK needs to create the CloudFormation stack**.  
It can represent **a single AWS resource** or **multiple related resources**.  

### Construct Library

A Collection of constructs included with CDK that **contains constructs for every AWS resource**.  

### Construct Hub

Contains **additional constructs** from AWS, 3rd parties and open-source CDK community.

### Layers

#### L1 constructs

**CFN Resources**. Represents all resources available in CloudFormation.  
Names start with `Cfn`, `CfnBucket`.  

You must **configure all resource properties**.

```ts
const bucket = new S3.CfnBucket(this, "MyBucket", {
    bucketName: "MyBucket"
});
```

#### L2 Constructs

Higher level API. Same as L1 but **with defaults**.

```ts
const s3 = require('aws-cdk-lib/aws-s3');

const bucket = new s3.Bucket(this, 'MyBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KMS
});

// Returns the HTTPS URL of an S3 Object
const objectUrl = bucket.urlForObject('MyBucket/MyObject');
```

#### L3 Constructs

**Patterns** that represents **multiple related resources**.

```ts
const api = new apigateway.LambdaRestApi(this, 'myapi', {
    handler: backend,
    proxy: false
});

const items = api.root.addResource('items');
items.addMethod('GET');  // GET /items
items.addMethod('POST'); // POST /items

const item = items.addResource('{item}');
item.addMethod('GET'); // GET /items/{item}
item.addMethod('DELETE', new apigateway.HttpIntegration('http://amazon.com'));
```

## Testing

**CDK Assertions Module** allow to test CDK apps.

```js
describe("StateMachineStack", () => {
  test("synthesizes the way we expect", () => {
    ...

    // Prepare the stack for assertions
    const template = Template.fromStack(MyStack);

    // FINE-GAINED ASSERTIONS
    // Assert it creates Lambda with correct properties...
    template.hasResourceProperties("AWS::Lambda::Function", {
      Handler: "handler",
      Runtime: "nodejs14.x",
    });

    // Assert it creates the SNS subscription...
    template.resourceCountIs("AWS::SNS::Subscription", 1);

    // SNAPSHOT TEST
    // Assert the synthesized CloudFormation template
    // against a previously stored baseline template
    expect(template.toJSON()).toMatchSnapshot();
  });
});
```
