# DynamoDB

## Sort Key

Used to **sort keys that have the same Partition ID**

## Read/Write Capacity Modes

**Read Modes:**
- **Eventually Consistent Read:** You can get stale data because of replication.
- **Strongly Consistent Read:** We will always get the correct data. Consumes twice the RCU.

### Provisioned Mode

You **specify the number of R/W per second**.  
Throughputt can be exceeded temporary with **Burst Capacity**.  
`ProvisionedThroughputExceededException` can be raise. *(Hot Keys / Partition, Very large item)*

One **Write Capacity Unit** (WCU) = **1 Write per second** up to **1KB**.  
One **Read Capacity Unit** (RCU) = **1 strongly consistent read per second** = **2 Eventually consistent read per second**, up to **4KB**.


### On-Demand Mode

Read/Write **automatically scale up/down**.  
More expensive, unlimited.

**Read Request Unit** (RRU) = Same as RCU.  
**Write Request Unit** (WRU) = Same as WCU.

## Partitions Internal

We **hash the partition key** to decide the partition in the table.  
**WCU and RCU are spread evenly accross partitions**.


## Operations

### Write

- `PutItem`: Create or fully replace an item
- `UpdateItem`: Edit/Add new attributes or create a new item
- `ConditionalWrites`
- `BatchWriteItem`: Up to 25 PutItem and/or DeleteItem in one call.
  * `UnprocessedItems` for failed operations.

### Read

- `GetItem`: Read based on a primary key. You can get precise attributes only.
- `Query`: Returns items based on
  * `KeyConditionExpression`: Partition Key value + optional sort key value
  * `FilterExpression`: Additional filtering after Query (non key attributes)
- `Scan`: Read an entire table (pagination)
- `Parallel Scan`: For faster performance
- `BatchGetItem`: Up to 100 items
  * `UnprocessedItems` for failed operations.

Note: `ProjectionExpression`: Only get certain attributes.

### Delete

- `DeleteItem`: Compatible with conditional delete
- `DeleteTable`: Delete everything.

### PartiQL

**SQL-Compatible query language for DynamoDB**.  
Not compatible with joins.

### Conditional Writes

You can specify a Condition expression to determine which items should be modified:
- `attribute_exists`
- `attribute_not_exists`
- `attribute_type`
- `contains` (for string)
- `begins_with` (for string)
- **ProductCategory** IN (:cat1, :cat2) and Price between :low and :high
- `size` (string length)

Note: Filter Expression filters the results of read queries, while Condition Expressions are for write operations.

## Indexes

### Local Secondary Index (LSI)

Gives an **alternative sort key**.  
Only at creation time.

### Global Secondary Index (GSI)

An **alternative primary key**. 
Throttling on the GSI will cause throttling on the whole table.  
Can be created after db creation.

## PartiQL

A **SQL-Like query language** for querying DynamoDB.  

```SQL
SELECT * FROM "demo_indexes" WHERE "user_id" = '123' AND "game_ts" = 'sortKeyValue'
```

## Optimist Locking

An attribute that act as a **version number**.
Allow doing **Conditional Writes** while ensuring that data **hasn't changed before you update/delete it**.

## DAX vs ElastiCache

- **DAX:** Individual object cache, query and scan cache.  
- **ElastiCache:** Store aggregationr results

## DynamoDB Stream Precisions

Also contains **reads**. Data retention up to **24 hours**.  
They are **made of shards** like Kinesis Data Streams. Provisioned by AWS.  
**Records are not retroactively populated !**
Ability to **choose the information that will be written** to the stream:
- `KEYS_ONLY`: only the key attributes of the modified item
- `NEW_IMAGE`: the entire item, as it appears after it was modified
- `OLD_IMAGE`: the entire item, as it appeared before it was modified
- `NEW_AND_OLD_IMAGES`: both the new and the old images of the item

## TTL

Does **not consume any WCU**.  
TTL should be a **UNIX timestamp**.  
**DynamoDB deletes item within 48 hours of the expiration**.

## DynamoDB Transactions

Coordinated **all-or-nothing** operations (add/update/delete) to multiple items across one or more tables.  
Consumes **2x WCU and RCUs**: We do 2 operations for every item, prepare and commit.

## CLI Options

- `--projection-expression`: one or more attributes to retrieve
- `--filter-expression`: filter items before returned to you

General AWS CLI Pagination options (e.g., DynamoDB, S3, ...):
- `--page-size`: specify that AWS CLI retrieves the full list of items but with a larger number of API calls instead of one API call (default: 1000 items)
- `--max-items`: max number of items to show in the CLI (returns NextToken)
- `--starting-token`: specify the last NextToken to retrieve the next set of items

## DynamoDB as a Session State Cache

A **serverless alternative** to ElastiCache with **automatic scaling**.

## Write Sharding

We can **add a suffix** to the partition key value to avoid **hot partition**.

## Write Types

- **Concurrent Writes:** One of the write will be overriden.
- **Conditional Writes:** Accept a random write and see if the second will overwrite or not.
- **Atomic Writes:** Both succeed.
- **Batch Writes:** Many items at a time.

## Large Objects Pattern

Use S3 and put the **path in DynamoDB**.  
DynamoDB can also be used for **indexing objects** by storing metadata.

## Operations 2

- **Table Cleanup:** Drop table and recreate
- **Copy Table:** Data Pipeline / Backup and Restore / Scan + PutItem or BatchWrite

## Security

**VPC Endpoints** to access without internet.  
Access is controlled by IAM.  
**Encryption at rest** with KMS and **in-transit** with SSL/TLS.

## Other features

- **Backup and Restore** + **PITR**.  
- **Global Tables** for a **multi-active** high performance **multi-region setup**.  
- **DynamoDB Local** to test apps without internet.
- **DMS** to migrate to DynamoDB
- **Fine Gained Access Control** to assign an IAM Role to ysers with a condition to limit API access.
- **LeadingKeys** to limit row-level access for users on the **primary key**
