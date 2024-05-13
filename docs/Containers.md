# Containers

## Rolling Updates

Just like **Instance Refresh**, you defina a min / max percentage.  
It'll be used when upgrading, to guarantee a strict minimum of instances.

*Max can be > 100%*

## ECS Tasks Invoked by EventBridge

We can use **EventBridge** to create ECS Task on events.  
Works on schedule, works to intercept stopped tasks...

## ECS Task Definitions

**JSON** Metadata to describe **how to run a Docker container**.  
Contains crucial information such as:

- Image Name
- Port Binding for Container and Host Mapping
- Memory and CPU required
- Env variables
- Networking information
- IAM Role
- Logging configuration *(ex: CloudWatch)*
- Data volumes for shared ephemeral storage between tasks

IAM Role is defined at Task-Definition level.  
For env variables, you can hardcode them, use SSM Parameter Store, Secrets Manager. (uses ARN)

### Load Balancing EC2

You can use **Dynamic Host Port Mapping if you define only the container port**.  
The ALB will know the port. You must allow access for the ALB on any port.

### Load Balancing Fargate

Each task has an **unique private IP**, you can only define the **container port**.  
You only need to define the **container port**, it will be the same on every instance.

## Task Placement for ECS on EC2

Given CPU memory and port **constraint, there is an associated placement**.  
Placement can be defined using **placement strategies**.  

Placement process:

1. Identify instances that satisfy the CPU, memory and port
2. Identify instances that satisfy the task placement constraints
3. Identify instances that satisfy the task placement strategies
4. Select the instances for task placement

### Placement Strategies

- **Binpack:** Based on the least available CPU and memory. Minimizes the number of instances.
- **Random**
- **Spread:** Spread based on AZ

Placement strategies can be mixed together.

### Placement Constraints

- **distinctInstance:** Place each task on a different container instance.
- **memberOf:** Place task on instances that satisfy an expression *(Cluster Query Language)*

## AWS CoPilot

A **CLI tool** to build, release and operate production-ready containerized apps.

## /etc/ecs/ecs.config

Contains **ECS configuration** in the EC2 instance level.

*Ex: ECS_ENABLE_TASK_IAM_ROLE allow to enable IAM roles for ECS tasks*
