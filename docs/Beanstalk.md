# Beanstalk

## Beanstalk Precisions

It uses **CloudFormation** to build and configure the infrastructure.  
It supports docker images.

## Deployment Options for Updates

- **All at once:** Fastest, but will create downtime
- **Rolling:** Update a few instances at a time (bucket) and then move onto the next bucket
- **Rolling with additional batches:** Like rolling but spins up the new instances to move the batch
- **Immutable:** Create a new temporary ASG, populate it then swaps all the instances
- **Blue Green:** Create a new environment and switch over when ready
- **Traffic Splitting:** Send a small % of traffic to new deployment

## Deployment Process

You need to describe dependencies with `package.json` or `requirements.txt` then upload the .zip.

## Beanstalk Lifecycle Policy

Beanstalk can store at most **1000 application versions**.  
To phase out old application versions, use a **lifecycle policy**.  
Based on age or total count. **Currently used versions won't be deleted.**  
You can **keep the source bundle in S3**.

## Beanstalk Extensions

All the parameters can be configured inside the `.ebextensions/` directory of the source code.  
Should be in `json` or `yaml` format. Every file should end with `.config`.  
Can be used for env variables, configuring other services such as Elasticache, enable HTTPS...

## Beanstalk Cloning

**Clone** an environment with the **exact same configuration**.  

## ELB Migration

After creating an environment you **cannot change the ELB type**.  
To migrate:

1. Create a new environment with the same configuration except the LB
2. Deploy the app
3. Swap traffic (CNAME / Route53)

## RDS on Beanstalk

RDS can be provisioned with Beanstalk, but this is **not great for production**.  
Because the **DB lifecycle is going to be the same as the Beanstalk environment**.  

To decouple RDS, just create a copy without RDS that use the original RDS.
Then protect the original RDS from deletion and delete the environment.
