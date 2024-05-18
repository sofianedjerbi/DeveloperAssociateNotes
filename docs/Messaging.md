# Messaging

## SQS Queue Access Policy

JSON policies that allow **cross account access**, allow **other services to send messages**.


```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "AWS": [ "111122223333" ] },
    "Action": [ "sqs:ReceiveMessage" ],
    "Resource": "arn:aws:sqs:us-east-1:444455556666:queue1"
  }]
}
```

*Ex: S3 Event Notification*

## SQS Dead Letter Queue Precisions

When the message has been processed **too many times uncessfully** (failure loop).  
It is send in a **dead letter queue**.  

The DLQ of a FIFO Queue is a **FIFO Queue**.

## SQS Delay Queue Precisions

Delay a message so the consumer don't see it immediately **up to 15 minutes**.  
You can set a **per message delay with DelaySeconds parameter**.

## SQS Long polling precisions

Can be enabled at queue level or at API level with **ReceiveMessageWaitTimeSeconds**.

## SQS Extended Client

Message limit is **256KB**. The extended client **uses an S3 bucket to store the real message**.  

## SQS API

- **CreateQueue** (MessageRetentionPeriod), **DeleteQueue**
- **PurgeQueue**: delete all the messages in queue
- **SendMessage** (DelaySeconds), **ReceiveMessage**, **DeleteMessage**
- **MaxNumberOfMessages**: default 1, max 10 *(for ReceiveMessage API batch processing)*
- **ReceiveMessageWaitTimeSeconds**: Long Polling
- **ChangeMessageVisibility**: change the message timeout

Batch APIs for **SendMessage**, **DeleteMessage**, **ChangeMessageVisibility** helps decrease your costs

## SQS FIFO Deduplication

Deduplication interval is **5 minutes**.  
Two deduplication methods:
- **Content-based deduplication:** SHA-256 hash of the body.
- **Provide a Message Deduplication ID**

## SQS FIFO Message Grouping

You can divide messages by **MessageGroupID**.  
Only **one consumer** per **MessageGroupID**.

## Kinesis Producers Precisions

Producers are **API, Kinesis Producer Library (C++, Java), Kinesis Agent (logs)**.  
The **partition key** is used to partition across shards.  
Try to avoid **hot partition**: Uneven data distribution overloads specific shards, causing throttling.

`ProvisionedThroughputExceeded` can happen if the throughput is exceeded. Solutions:
- Use a highly distributed partition key
- Retries with exponential backoff
- Increase shards (scaling)

## Kinesis Consumers Precisions

Consumers can be **API, Kinesis Consumer Library (Java), Kinesis Firehose / Analytics, Lambda**.  
**Enhanced fanout** can be used to distribute **throughput across consumers**.  
It **pushes** data instead of **pulling** it.

*i.e 2mb/s per consumer per shard vs 2mb/s per shard for all consumers*

## Kinesis Consumer Library (KCL)

A java library, **each shard is to be read by only one KCL instance**.  
A KCL instance can read many shards.

## Shard Operations

Both operations create new shards.

- **Splitting:** Increase the strean capacity by splitting. Used to divide an old shard, that will expire.
- **Merging:** Decrease capacity and reduce costs. Old shards will expire.
