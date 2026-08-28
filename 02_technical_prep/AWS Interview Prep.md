# AWS Data Engineering — Interview Prep

> Interview-focused summary of the uploaded AWS Services Q&A. The source is broad and repetitive, so this note emphasizes **mental models, service selection, tradeoffs, failure modes, and scenarios** rather than memorizing definitions. fileciteturn17file0

## 1. AWS Data Engineering Mental Model

```text
Sources → Ingestion → Storage → Transformation → Analytics
 RDS       DMS          S3        Glue / EMR       Athena
 APIs      Kinesis                Lambda            Redshift
 Apps      Firehose
```

Cross-cutting:
- **Security:** IAM, KMS, Secrets Manager, Lake Formation
- **Monitoring:** CloudWatch
- **Auditing:** CloudTrail
- **Orchestration:** Step Functions, Glue Triggers, EventBridge
- **Messaging:** SQS, SNS

**Interview framework:** Requirement → workload characteristics → candidate service → tradeoffs → failure modes → security → cost.

---

# 2. Amazon S3

### What is it?

Object storage and the typical durable storage layer for an AWS data lake.

- Bucket = container
- Object = stored data
- Key = unique object identifier
- Storage is decoupled from compute.

### Why S3 for a data lake?

- Highly scalable/durable
- Cost-effective
- Supports open formats such as Parquet
- Integrates with Glue, Athena, EMR, Redshift Spectrum, Lambda, etc.

### Important features

**Versioning**
- Keeps multiple versions of objects.
- Useful for accidental overwrite/delete recovery.

**Lifecycle policies**
- Automatically transition/delete objects based on age/access requirements.
- Useful for controlling long-term storage costs.

**Event notifications**
```text
Object created → S3 event → Lambda / SQS / SNS
```

**Replication**
- Same-Region Replication
- Cross-Region Replication
- S3 Batch Replication

**Access Points**
- Useful when many teams need different access to the same bucket.
- Avoids one enormous bucket policy becoming unmanageable. fileciteturn18file17

### Interview trap

> An S3 bucket can be accessed even when an IAM policy allows it, but an applicable **explicit deny** can still block the request.

---

# 3. Amazon DynamoDB

DynamoDB is a fully managed NoSQL database designed for fast, predictable performance and seamless scalability. It supports key-value and document data models. fileciteturn21file9

> **Core interview principle:** Design DynamoDB around **access patterns**, not a normalized relational schema.

## Primary Keys

A table can use:

- **Simple primary key:** partition key
- **Composite primary key:** partition key + sort key

The **partition key** determines data distribution. The **sort key** lets multiple related items share a partition key and supports ordered queries within that item collection. fileciteturn21file5

Example:

```text
PK = UserID
SK = Timestamp
```

## Hot Partitions

A poor partition key can concentrate traffic on one partition.

Bad:
```text
PK = current timestamp
```

Better:
```text
PK = UserID
SK = Timestamp
```

The source specifically identifies timestamp-only keys as a hot-partition scenario and recommends a composite key to spread writes across users. fileciteturn21file6

> **Interview answer:** "I'd choose a high-cardinality partition key that distributes traffic evenly and avoid keys that concentrate concurrent writes."

## Secondary Indexes

DynamoDB supports:

- **GSI — Global Secondary Index:** alternate key/access pattern
- **LSI — Local Secondary Index:** alternate sort key using the same partition key

Indexes should support known query patterns and can affect write capacity/performance.

### GSI Backpressure

The source includes a scenario where the main table has plenty of WCUs but its GSI has very little capacity:

```text
Main table → 10,000 WCUs
GSI        → 5 WCUs
```

Writes can still be throttled because the GSI must keep up with table writes. The source calls this **GSI backpressure**. fileciteturn21file8

### LSI and the 10 GB Item-Collection Limit

When a table has an LSI, the source states that the collection of items sharing one partition key cannot exceed **10 GB**. This can become a problem for a customer with millions of orders under one partition key. fileciteturn21file11

## Capacity

DynamoDB uses:

- **RCU — Read Capacity Units**
- **WCU — Write Capacity Units**

Capacity planning depends on:
- Reads/writes per second
- Item size
- Access patterns
- Indexes

fileciteturn21file5turn21file6

### On-Demand vs. Provisioned

**On-Demand**
- Automatically adapts to workload
- Pay per request
- Good for variable/unpredictable workloads

**Provisioned**
- Explicit read/write capacity
- Good for predictable workloads
- Can use auto-scaling

The source notes that auto-scaling may react too slowly to sudden spikes, making on-demand preferable for some unpredictable workloads. fileciteturn21file7turn21file8

## Eventual vs. Strongly Consistent Reads

**Eventually consistent**
- May temporarily return stale data
- More efficient

**Strongly consistent**
- Returns the most up-to-date value
- Use when the latest value is required immediately

The source uses an account-balance scenario to illustrate this distinction. fileciteturn21file10

> **Interview answer:** "I'd use eventual consistency when some staleness is acceptable. For a critical read where the latest value is required immediately, I'd use a strongly consistent read."

## Query vs. Scan

**Query**
> Retrieves items using a partition key and is the preferred pattern for targeted reads.

**Scan**
> Reads the entire table and then filters the results.

Therefore:

> **Prefer Query over Scan for production access patterns whenever possible.** fileciteturn21file6

## Pagination

The source states that a Query or Scan response can return a maximum of **1 MB** of data.

If `LastEvaluatedKey` is present, more data exists:

```text
Request → 1 MB → LastEvaluatedKey
                    ↓
             Next request using
             ExclusiveStartKey
```

fileciteturn21file10

## Condition Expressions and Optimistic Locking

Condition expressions require a condition to be true before a write succeeds.

They are useful for:
- Preventing overwrites
- Business rules
- Duplicate prevention
- Optimistic locking

For a lost-update race condition, the source recommends a version attribute:

```text
version = 1

Admin A:
UPDATE ... WHERE version = 1
→ version = 2

Admin B:
UPDATE ... WHERE version = 1
→ fails
```

This prevents Admin B from silently overwriting Admin A's changes. fileciteturn21file3turn21file4

## Transactions

`TransactWriteItems` allows multiple actions to execute as an **all-or-nothing operation**.

Useful when several writes must succeed atomically.

Tradeoff:

> Transactions can increase latency and reduce throughput compared with single-item operations. fileciteturn21file4

The source also discusses transaction item/size limits and application-level alternatives such as a Saga pattern when a single transaction cannot contain the required work. fileciteturn21file3

## DynamoDB Streams

DynamoDB Streams captures item changes in a time-ordered sequence.

```text
DynamoDB
   ↓
DynamoDB Streams
   ↓
Lambda
   ↓
Downstream processing
```

Use cases include downstream processing, search-index updates, replication, and asynchronous cleanup. fileciteturn21file5turn21file8

### Streams Failure Handling

The source includes a malformed-record scenario where Lambda repeatedly fails and `IteratorAge` increases because ordered stream processing is blocked.

Recommended mechanisms:
- **Bisect on Function Error** to isolate the bad record
- **On-Failure Destination / DLQ** after retries

fileciteturn21file11

## No Cascade Deletes

DynamoDB does not automatically cascade-delete child items.

For example:

```text
User:   PK=USER#1
Orders: PK=USER#1, SK=ORDER#A
        PK=USER#1, SK=ORDER#B
```

Deleting the User does not delete the Orders.

The source recommends querying the partition and deleting child items, or using DynamoDB Streams + Lambda for asynchronous cleanup. fileciteturn21file8

## Item Size Limit

The source states a maximum single-item size of **400 KB**.

For larger data:
- Split it across multiple items, or
- Store the large payload in S3 and keep a pointer/metadata in DynamoDB. fileciteturn21file4turn21file6

## TTL

DynamoDB TTL deletes expired items through a background process.

> **TTL is not real-time deletion.**

The source warns not to use TTL as the application's correctness mechanism; application logic should still determine whether data is expired. fileciteturn21file10

## Caching

The source identifies:

**DAX**
> In-memory caching specifically for DynamoDB.

**ElastiCache / Redis**
> External caching for frequently accessed data.

Caching can reduce latency and DynamoDB reads. fileciteturn21file7

## Backups and Recovery

DynamoDB supports:
- On-demand backups
- Continuous backups / point-in-time recovery

Backups can be used to recover table data. fileciteturn21file7

## Monitoring and Troubleshooting

Use CloudWatch to monitor:
- Read/write capacity
- Throttling
- Latency
- Table activity

Common errors in the source:
- `ProvisionedThroughputExceededException`
- `ConditionalCheckFailedException`
- `ResourceNotFoundException`
- `ThrottlingException`

Troubleshooting flow:

```text
DynamoDB problem
      ↓
CloudWatch metrics
      ↓
Throttling?
Capacity?
Latency?
Hot partition?
GSI bottleneck?
Query vs. Scan?
```

fileciteturn21file4turn21file7

## DynamoDB Scenarios to Practice

1. **Hot partition:** Timestamp as PK causes concentrated writes. Fix with a better partition key/composite key. fileciteturn21file6
2. **Scan in production:** Scan reads the entire table; design around Query. fileciteturn21file6
3. **GSI throttling:** A low-capacity GSI can throttle writes to the main table. fileciteturn21file8
4. **Lost update:** Use optimistic locking with a version attribute and conditional write. fileciteturn21file3
5. **Stale read:** Use strong consistency when the latest value is required immediately. fileciteturn21file10
6. **400 KB item:** Split the item or store the large payload in S3. fileciteturn21file4turn21file6
7. **Poison-pill stream record:** Isolate the record and use a DLQ/on-failure destination. fileciteturn21file11

---

# 4. IAM

IAM controls access to AWS resources.

## Users vs. Roles

**User**
- Persistent identity
- Can have long-lived credentials

**Role**
- Assumable identity
- Provides temporary credentials
- Preferred for AWS services such as Glue, Lambda, and EC2

> **Machines should generally use roles, not hard-coded IAM access keys.** fileciteturn18file3

## Identity-based vs. resource-based policy

**Identity-based**
> Attached to a user/group/role and says what that identity can do.

**Resource-based**
> Attached to a resource such as an S3 bucket and says who can access it.

## Least privilege

Give only the permissions required.

Bad:
```text
Glue Job → AdministratorAccess
```

Better:
```text
Glue Job Role
 ├── GetObject on source path
 └── PutObject on target path
```

## Explicit deny

```text
Default deny
    ↓
Explicit allow?
    ↓
Explicit deny anywhere?
    ├── Yes → DENY
    └── No  → ALLOW
```

## `sts:AssumeRole`

Used to obtain temporary credentials for another role.

### Cross-account pattern

```text
Account B
Glue Role
   │ sts:AssumeRole
   ↓
Account A
CrossAccountRole
   ↓
S3 data
```

The target role's trust policy must trust the source account/role, and the source must be allowed to call `sts:AssumeRole`.

## `iam:PassRole`

Controls whether an identity can assign a role to an AWS service.

Why it matters:

```text
Developer → create Glue job → attach AdminRole
```

Without PassRole controls, a developer could effectively escalate privileges through a service.

## Permissions boundaries

A permissions boundary is a maximum permission ceiling.

```text
Role policy
     ∩
Permissions boundary
     ↓
Effective permissions
```

Useful when developers need to create roles but must not be able to create unrestricted admin roles. fileciteturn20file18

## RBAC vs. ABAC

**RBAC:** access based on roles.

**ABAC:** access based on attributes/tags.

ABAC can reduce role explosion when access maps naturally to attributes such as department or project.

## Enterprise identity

For thousands of employees, prefer federation/IAM Identity Center rather than manually managing IAM users.

```text
Corporate IdP → IAM Identity Center → AWS permissions/roles
```

---

# 5. AWS Glue

Glue is a managed/serverless data integration and ETL service.

```text
Glue
├── Data Catalog
├── Crawlers
├── ETL Jobs
├── Triggers
├── Connections
└── Glue Studio
```

## Data Catalog

Stores metadata:

- Tables
- Schemas
- Columns/types
- Partitions
- Locations

> **Catalog = metadata, not the underlying S3 data.**

## Crawlers

```text
S3 / database
     ↓
Crawler
     ↓
Schema discovery
     ↓
Glue Data Catalog
```

Useful for automated schema discovery, but huge numbers of partitions can make crawlers expensive/slow.

## Glue jobs

- **Spark jobs:** large distributed workloads
- **Python Shell:** smaller workloads that don't need Spark fileciteturn18file5

Glue uses Spark for distributed ETL.

## DynamicFrame vs. DataFrame

DynamicFrames are Glue-specific and designed to handle messy/evolving schemas.

```python
df = dynamic_frame.toDF()
```

Convert to a standard Spark DataFrame when you want the normal Spark SQL/DataFrame API.

## Glue bookmarks

Track previously processed data so subsequent runs can focus on new/changed data.

```text
Run 1 → A, B
Run 2 → C
Run 3 → D
```

## Glue triggers

Can start jobs based on:
- Schedule
- Events
- Dependencies

## Glue Streaming

Glue can run streaming ETL using Spark Structured Streaming.

```text
Kinesis/Kafka → Glue Streaming → S3/Redshift
```

Good when minute-level latency is acceptable; specialized streaming engines are preferable for very low-latency/stateful processing.

## Glue performance

When a job is slow:

1. Check CloudWatch logs/metrics.
2. Check Spark stages.
3. Look for skew.
4. Look for shuffle/spill.
5. Check partition sizing.
6. Check tiny files.
7. Review transformations.
8. Only then add compute.

The source gives an OOM scenario where one executor was overloaded because of a skewed join; key salting was preferred over simply increasing DPUs. fileciteturn18file15

## Glue cost

Reduce:

- Compute size
- Runtime
- Reprocessing

Useful tools:
- Auto Scaling
- Flex execution for non-urgent jobs
- Bookmarks
- Efficient file formats
- Appropriate partitioning

---

# 6. Amazon Athena

Athena is a **serverless SQL query engine for data stored in S3**.

```text
S3 data
   ↑
Athena ← Glue Data Catalog
   ↓
SQL results
```

Athena owns neither the data nor the primary storage.

## When to use Athena

Good for:
- Ad-hoc SQL
- Data exploration
- Intermittent analytics
- Querying existing S3 data without loading a warehouse

## Athena vs. Redshift

| Athena | Redshift |
|---|---|
| Serverless | Data warehouse |
| Queries S3 | Warehouse-managed storage/compute |
| Ad-hoc workloads | Frequent analytical workloads |
| Pay for data scanned | Pay for compute/storage |
| Minimal infrastructure | More warehouse control |

## Athena cost model

The source emphasizes **data scanned** as the key cost/performance consideration.

Therefore:

```sql
SELECT *
FROM huge_table;
```

is usually worse than selecting only needed columns and filtering early.

## Parquet / ORC

Preferred for analytics because they provide:

- Columnar storage
- Compression
- Column pruning
- Less I/O
- Lower scan cost

The source includes a JSON→Parquet scenario where converting the data dramatically reduces bytes scanned. fileciteturn17file9

## Partitioning

Example:

```text
s3://bucket/events/year=2026/month=08/day=28/
```

A query filtering by date can skip irrelevant directories.

**Do not over-partition.**

High-cardinality partitioning can create:
- Huge numbers of directories
- Tiny files
- Metadata overhead

## Partition projection

Useful when partition counts become extremely large. Athena can derive partition locations from configured rules instead of enumerating every folder through the catalog. fileciteturn18file18

## CTAS

`CREATE TABLE AS SELECT` can transform/query data and write the result to S3.

Useful for:
- Format conversion
- Aggregation
- Creating optimized datasets

## Federated Query

Athena can query non-S3 sources through connectors.

```text
S3 ──────────┐
             ├→ Athena
DynamoDB ────┘
```

Convenient for occasional cross-source queries, but repeatedly querying operational databases can be inferior to building a proper analytical pipeline.

## Iceberg

The source highlights Iceberg when lake data needs update/delete/table-management capabilities rather than simple read/append semantics. fileciteturn18file18

---

# 7. AWS Lambda

Lambda is serverless, event-driven compute.

Good for:
- Lightweight transformations
- S3 event processing
- Kinesis processing
- Notifications
- Small orchestration tasks

Poor for:
- Large distributed transformations
- Long-running processing

### Selection rule

```text
Small + event-driven + bursty → Lambda
Large + distributed + long-running → Glue/EMR
```

The source contrasts Lambda with EC2/Glue for sporadic file processing. fileciteturn18file9

## Important constraints

The source emphasizes:
- 15-minute maximum execution time
- Memory/CPU are coupled
- Event payload limits matter

For large files:

```text
S3 object
   ↓
Lambda receives object key
   ↓
Lambda reads/processes the object
```

Don't try to pass the entire large file through the Lambda event.

## Lambda + IAM

Lambda gets AWS permissions through an execution role.

## Retries

Retry behavior depends on invocation type/event source.

> Interview point: know whether **Lambda, the event source, or the caller** owns the retry behavior.

## Failure destinations

Asynchronous failures can be routed to SNS/SQS after retries.

## Infinite feedback loop

Classic failure:

```text
Stream A → Lambda → Stream A → Lambda → ...
```

This can exhaust throughput and explode costs.

Fix:
1. Disable the event source mapping.
2. Correct the destination.
3. Re-enable processing.

The source explicitly presents this scenario. fileciteturn20file0

---

# 8. Kinesis Data Streams

Designed for real-time streaming ingestion and processing.

```text
Applications
     ↓
Kinesis Data Streams
     ↓
 ┌───┼────┐
 ↓   ↓    ↓
App Lambda Analytics
```

## Kinesis vs. SQS

**SQS**
> Work queue. A message is generally processed by one consumer.

**Kinesis**
> Stream. Multiple consumers can independently process/replay the data.

Kinesis also provides ordering within a shard. fileciteturn20file13

## Shards

A shard is a throughput unit.

The source uses the classic approximate figures:
- 1 MB/sec write
- 1,000 records/sec write
- 2 MB/sec read

Example:

```text
4.5 MB/sec peak
÷ 1 MB/sec/shard
= 4.5
→ at least 5 shards
```

Also consider record count and partition-key distribution.

## Hot shards

Bad partition key:

```text
Shard 1 → 90%
Shard 2 → 3%
Shard 3 → 3%
Shard 4 → 4%
```

Choose a better key/distribution strategy.

---

# 9. Kinesis Data Firehose

Firehose is a fully managed **delivery** service.

```text
Application/Kinesis
        ↓
     Firehose
        ↓
     S3 / Redshift / other destination
```

## Firehose vs. Data Streams

| Firehose | Data Streams |
|---|---|
| Managed delivery | Custom stream processing |
| Destination-oriented | Consumer-oriented |
| Buffers and delivers | Consumers process stream |
| Less operational work | More flexibility |

## Buffering

Firehose buffers by size/time.

```text
Smaller buffer → lower latency
Larger buffer  → larger batches / potentially better efficiency
```

## Lambda transformation

```text
Records → Firehose → Lambda transform → destination
```

Failed transformations can be routed to S3 for analysis. fileciteturn18file0

---

# 10. Amazon Redshift

Redshift is an analytical OLAP data warehouse.

Use for:
- Repeated analytical queries
- Large aggregations
- High-performance BI
- Structured warehouse workloads

## Redshift vs. RDS

**RDS**
- OLTP
- Transactional
- Row-oriented access patterns

**Redshift**
- OLAP
- Analytical
- Columnar storage
- MPP

The source uses this exact distinction in its scenario questions. fileciteturn19file15

## Architecture

```text
Client
  ↓
Leader Node
  ↓
Compute Nodes
  ↓
Slices
```

**Leader:** parses/plans/coordinates.

**Compute nodes:** store data and perform computation.

## Distribution styles

**KEY**
- Hash/distribute using a chosen column.
- Useful when tables frequently join on that key.

**EVEN**
- Distribute evenly.
- Useful when no strong key exists.

**ALL**
- Copy a small table to every node.
- Useful for small reference tables.

### Data skew

Bad distribution:

```text
Node 1 → 80%
Node 2 → 7%
Node 3 → 7%
Node 4 → 6%
```

One node becomes the bottleneck.

## Sort keys

Physically organize data around chosen columns to improve analytical access patterns.

## `COPY`

Preferred for bulk loading.

```text
Source → S3 → COPY → Redshift
```

Avoid millions of individual INSERTs for bulk ingestion. The source emphasizes parallel loading from S3. fileciteturn19file19

## `UNLOAD`

Exports Redshift data to S3.

## `VACUUM` vs. `ANALYZE`

**VACUUM**
> Reclaims space and helps restore sort order after updates/deletes.

**ANALYZE**
> Updates optimizer statistics.

The source explicitly tests this distinction. fileciteturn19file2

## Redshift Spectrum

Query S3 data without loading it into Redshift.

## Materialized views

Precompute/store results for repeatedly expensive analytical queries.

---

# 11. AWS DMS

DMS supports database migration and ongoing replication.

Components:

```text
Replication Instance
Source Endpoint
Target Endpoint
Migration Task
```

## Full Load vs. CDC

**Full Load**
> Copies existing data.

**CDC**
> Captures subsequent source changes.

Typical architecture:

```text
Source DB
   ↓
 DMS
   ├── Full Load
   └── CDC
        ↓
    Target DB
```

DMS tasks can combine full load and CDC. fileciteturn18file1

## Replication lag

Monitor:
- CDC lag
- Throughput
- Errors
- Task health

## Schema conversion

AWS Schema Conversion Tool (SCT) can assist with heterogeneous migrations.

---

# 12. AWS KMS

KMS manages encryption keys.

Used with:
- S3
- RDS
- Redshift
- Secrets Manager
- Application encryption

## AWS-managed vs. customer-managed keys

**AWS-managed**
> AWS manages the key for an AWS service.

**Customer-managed**
> You control policies, permissions, lifecycle, and configuration.

## Key policy

A KMS key policy is a resource-based policy controlling key access.

## Encryption context

Additional authenticated data associated with encryption/decryption.

```text
Encrypt(data, key, context={App: Frontend})
```

Can prevent a permitted principal from decrypting data intended for a different context. fileciteturn19file0

## Key deletion

Deletion is scheduled rather than instantaneous. If encryption suddenly fails, check:
- Key state
- Key policy
- Referenced key
- CloudTrail for `ScheduleKeyDeletion`

---

# 13. AWS Secrets Manager

Securely stores:
- Database credentials
- API keys
- OAuth tokens
- Other secrets

## Secrets Manager vs. Parameter Store

**Secrets Manager**
> Secret-focused, supports rotation.

**Parameter Store**
> General configuration/parameter storage.

## Rotation

A common database-password rotation flow:

```text
Secrets Manager
      ↓
Create new version
      ↓
Lambda updates database
      ↓
Test credentials
      ↓
Move AWSCURRENT
```

## Private subnet

A Glue job without internet access can use a **VPC Interface Endpoint / PrivateLink** to reach Secrets Manager without a NAT Gateway. fileciteturn19file8

## Cross-account secrets

Typically requires:
- Secret resource policy in the owning account
- Identity policy on the consuming role
- KMS permissions if applicable

---

# 14. CloudWatch

CloudWatch provides monitoring and observability.

```text
CloudWatch
├── Metrics
├── Logs
├── Alarms
├── Dashboards
└── Logs Insights
```

### Metrics vs. Logs

**Metrics**
> Numerical measurements.

**Logs**
> Detailed event records.

> **Metrics tell you that something is wrong; logs help explain why.**

### Alarms

```text
Metric → Threshold → Alarm → Action
```

Actions can include notifications or scaling.

### Logs Insights

Use to query/analyze logs during troubleshooting.

For Glue failures:
1. Check job status.
2. Inspect CloudWatch logs.
3. Find failing stage/error.
4. Check resource metrics.
5. Determine code/data/permission/resource cause.

---

# 15. CloudTrail

Think:

> **CloudWatch = monitoring**

> **CloudTrail = AWS API auditing**

Use CloudTrail to answer:

- Who changed this resource?
- Who deleted something?
- Who scheduled key deletion?
- Which API call caused the configuration change?

---

# 16. Lake Formation

Lake Formation provides data-lake governance and fine-grained permissions.

Useful distinction:

```text
IAM
 ↓
Coarse AWS/resource access
 ↓
Lake Formation
 ↓
Fine-grained data access
 ↓
Table / column / row
```

Example:

> User can query the Sales table but cannot see the SSN column.

The source presents Lake Formation as a data-centric permission layer while IAM still provides the underlying AWS permissions. fileciteturn18file18

---

# 17. EMR

EMR provides managed clusters for big-data frameworks such as Spark.

## Glue vs. EMR

**Glue**
- Serverless
- Lower operational overhead
- AWS-integrated
- Strong fit for managed ETL

**EMR**
- More cluster/environment control
- Broader big-data ecosystem
- Useful for specialized configurations
- More operational responsibility

### Interview answer

> "I'd choose Glue when I want managed serverless ETL and don't need much cluster control. I'd choose EMR when I need more control over the Spark environment or broader big-data configuration."

---

# 18. SQS and SNS

## SQS

Queue for decoupling producers and consumers.

```text
Producer → SQS → Consumer
```

Useful for:
- Async work
- Backpressure
- Retries
- Decoupling

## SNS

Pub/sub fanout.

```text
             SNS Topic
            /    |               ↓     ↓     ↓
         SQS   Lambda  Email
```

## Lambda + SQS partial batch failure

If one message in a batch fails:

```text
1 ✓
2 ✓
3 ✗
4 ✓
5 ✓
```

Report only message 3 as failed so successful messages can be removed while message 3 is retried. The source includes this as a failure-handling scenario. fileciteturn20file9

---

# 19. API Gateway

Managed API endpoints.

Useful for:
- Lambda APIs
- Event ingestion
- Authentication/authorization
- Throttling
- Routing

For users and API infrastructure in the same region, a **Regional** endpoint may avoid unnecessary CloudFront/edge routing overhead. The source gives this as a latency scenario. fileciteturn20file1

---

# 20. Step Functions

Workflow orchestration.

```text
Extract
  ↓
Transform
  ↓
Validate
  ↓
Load
  ↓
Notify
```

Can coordinate Glue, Lambda, EMR, ECS, and other AWS services.

> Use orchestration to coordinate workflow steps rather than putting an entire multi-stage pipeline into one function.

---

# 21. Service Selection Cheat Sheet

| Requirement | Candidate |
|---|---|
| Data lake storage | **S3** |
| Low-latency NoSQL/key-value access | **DynamoDB** |
| Metadata/catalog | **Glue Data Catalog** |
| Schema discovery | **Glue Crawler** |
| Serverless ETL | **Glue** |
| Distributed Spark with more control | **EMR** |
| Ad-hoc SQL over S3 | **Athena** |
| Analytical warehouse | **Redshift** |
| Database migration/CDC | **DMS** |
| Lightweight event processing | **Lambda** |
| Real-time stream | **Kinesis Data Streams** |
| Managed streaming delivery | **Kinesis Firehose** |
| Queue | **SQS** |
| Pub/sub fanout | **SNS** |
| Encryption keys | **KMS** |
| Secrets | **Secrets Manager** |
| Monitoring | **CloudWatch** |
| AWS API audit | **CloudTrail** |
| Fine-grained data-lake governance | **Lake Formation** |
| Workflow orchestration | **Step Functions** |
| Managed APIs | **API Gateway** |

---

# 22. High-Value Comparisons

## Glue vs. Lambda

**Lambda**
> Small, event-driven, short-lived work.

**Glue**
> Distributed ETL and larger data transformations.

## Glue vs. EMR

**Glue**
> Serverless, less infrastructure management.

**EMR**
> More control over Spark/cluster environment.

## Athena vs. Redshift

**Athena**
> Ad-hoc SQL directly over S3.

**Redshift**
> Repeated, performance-sensitive analytical workloads.

## Kinesis Streams vs. Firehose

**Streams**
> Custom consumers, multiple consumers, stream processing.

**Firehose**
> Managed delivery into destinations.

## Kinesis vs. SQS

**Kinesis**
> Stream/replay/multiple consumers.

**SQS**
> Queue/work distribution.

## CloudWatch vs. CloudTrail

**CloudWatch**
> Monitoring and observability.

**CloudTrail**
> API activity/auditing.

## IAM vs. Lake Formation

**IAM**
> AWS resource permissions.

**Lake Formation**
> Fine-grained data-lake governance.

## KMS vs. Secrets Manager

**KMS**
> Encryption keys.

**Secrets Manager**
> Secrets/credentials.

---

# 23. Architecture Scenarios

## Scenario 1 — Batch data lake

```text
Source
  ↓
S3
  ↓
Glue
  ↓
Parquet
  ↓
Athena
```

Use when batch processing and ad-hoc analytics are sufficient.

## Scenario 2 — Warehouse

```text
Source → S3/DMS/Glue → Redshift → BI
```

Use for frequent, complex analytical workloads.

## Scenario 3 — Real-time analytics

```text
Applications
     ↓
Kinesis Streams
     ↓
 ┌───┴────┐
 ↓        ↓
Lambda   Analytics
 ↓
S3/Redshift
```

## Scenario 4 — Streaming delivery

```text
Application → Firehose → S3 → Athena
```

Use when you mainly need managed delivery rather than complex stream processing.

## Scenario 5 — Database migration

```text
Source DB
   ↓
DMS
   ├── Full Load
   └── CDC
        ↓
Target
```

---

# 24. Troubleshooting Framework

When an AWS data pipeline fails, don't jump directly to changing the architecture.

## Step 1 — Identify the failure layer

```text
Network?
IAM?
Data quality?
Application code?
Compute?
Storage?
Service quota?
```

## Step 2 — Check observability

- CloudWatch logs
- CloudWatch metrics
- CloudTrail API history
- Service-specific logs/metrics

## Step 3 — Check common causes

### AccessDenied
Check:
- IAM identity policy
- Resource policy
- Explicit deny
- Permissions boundary
- SCP
- VPC endpoint policy
- KMS policy
- Lake Formation
- Cross-account trust

The source explicitly recommends checking endpoint policies, bucket policies, and permissions boundaries when an IAM role appears to have S3 access. fileciteturn18file16

### Slow Glue job
Check:
- Skew
- Shuffle
- Spill
- Tiny files
- Partitioning
- Excessive data movement
- Worker utilization

### Expensive Athena query
Check:
- Bytes scanned
- File format
- Compression
- Partitioning
- Columns selected
- Predicate filtering
- Partition count

### Redshift query is slow
Check:
- Distribution skew
- Distribution key
- Sort key
- Query plan
- Statistics
- VACUUM/ANALYZE
- Concurrency

---

# 25. Security Checklist

- [ ] IAM roles instead of long-lived machine credentials
- [ ] Least privilege
- [ ] Explicit deny behavior
- [ ] Identity vs. resource policies
- [ ] `sts:AssumeRole`
- [ ] `iam:PassRole`
- [ ] Permissions boundaries
- [ ] ABAC vs. RBAC
- [ ] IAM Identity Center/federation
- [ ] Cross-account roles
- [ ] S3 bucket policies
- [ ] VPC endpoint policies
- [ ] KMS key policies
- [ ] Secrets Manager
- [ ] Encryption at rest/in transit
- [ ] Lake Formation
- [ ] CloudTrail
- [ ] Confused deputy prevention

---

# 26. Performance Checklist

- [ ] Parquet/ORC
- [ ] Compression
- [ ] S3 partitioning
- [ ] Athena bytes scanned
- [ ] Athena partition projection
- [ ] Glue partitioning
- [ ] Spark shuffle
- [ ] Data skew
- [ ] Spill
- [ ] Tiny files
- [ ] Redshift distribution keys
- [ ] Redshift sort keys
- [ ] Redshift VACUUM/ANALYZE
- [ ] Redshift Spectrum
- [ ] Kinesis shard sizing
- [ ] Kinesis hot shards
- [ ] Firehose buffering

---

# 27. Cost Checklist

## S3
- Lifecycle policies
- Storage classes
- Compression
- Retention

## Athena
- Reduce bytes scanned
- Parquet/ORC
- Partitioning
- Avoid `SELECT *`

## Glue
- Right-size compute
- Auto Scaling
- Flex execution
- Bookmarks
- Efficient transformations

## Redshift
- Right-size compute
- Workload isolation
- Concurrency scaling where appropriate
- Spectrum for suitable S3 data
- Materialized views for repeated workloads

## Kinesis
- Right-size shards
- Avoid hot shards
- Tune Firehose buffering

> **Best interview answer:** optimize the whole system for cost, performance, reliability, and operational complexity rather than choosing the cheapest individual service.

---

# 28. Priority Study Order

## Tier 1 — Must Know

1. S3
2. IAM
3. DynamoDB
4. Glue
5. Athena
6. Lambda
7. Kinesis Data Streams
8. Kinesis Firehose
9. Redshift
10. DMS
11. KMS
12. Secrets Manager
13. CloudWatch
14. CloudTrail
15. Lake Formation
16. SQS/SNS
17. Glue vs. EMR
18. Athena vs. Redshift
19. Kinesis vs. SQS
20. IAM security patterns
21. S3/Glue/Athena troubleshooting

## Tier 2 — Strongly Recommended

21. Step Functions
22. API Gateway
23. EMR
24. S3 replication/access points
25. Athena federated queries
26. Athena CTAS
27. Partition projection
28. Redshift Spectrum
29. Redshift materialized views
30. KMS encryption context
31. Secrets Manager rotation
32. Cross-account patterns
33. ABAC
34. Permissions boundaries

## Tier 3 — Lower Priority

35. Detailed console configuration
36. Rare administrative edge cases
37. Less-common service-specific limits

For Data Engineering interviews, prioritize **Tier 1 + architecture scenarios + troubleshooting scenarios**.

---

# 29. One-Minute AWS Data Engineering Answer

> "I think of AWS's data engineering ecosystem as a set of specialized services that I compose based on the workload. S3 is usually my durable data-lake storage layer, with Parquet and partitioning for efficient analytics. Glue provides managed ETL, Spark processing, and the Data Catalog, while Athena provides serverless SQL directly over S3. For a warehouse workload I'd consider Redshift, particularly when queries are frequent and performance-sensitive. For real-time workloads, Kinesis Data Streams gives me a stream with multiple consumers, while Firehose is useful when I mainly need managed delivery into S3 or another destination. DMS is useful for database migration and CDC. Lambda is a good fit for lightweight event-driven processing, while EMR gives me more control over distributed compute when Glue isn't flexible enough. Across the platform, IAM, KMS, Secrets Manager, Lake Formation, CloudWatch, and CloudTrail provide security, governance, and observability. In an interview, I'd focus less on listing services and more on explaining why I would choose one service over another based on scale, latency, reliability, cost, and operational overhead."

---

# 30. Final Mental Model

```text
                              AWS
                               │
       ┌───────────────────────┼────────────────────────┐
       ↓                       ↓                        ↓
    STORAGE                 COMPUTE                ANALYTICS
       │                       │                        │
      S3                 ┌─────┼─────┐           ┌────┴────┐
       │                 ↓     ↓     ↓           ↓         ↓
 Raw/Curated            Glue   EMR  Lambda      Athena   Redshift
       │                 │
       │                 ↓
       │               Spark
       │
       └────────────────┬──────────────────────────────
                        ↓
                    INGESTION
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
            DMS      Kinesis    Firehose
             │          │          │
           CDC/DB     Streams    Delivery
                        │
                        ↓
                   REAL-TIME DATA

                 CROSS-CUTTING
                        │
      ┌─────────────────┼─────────────────┐
      ↓                 ↓                 ↓
     IAM                KMS          Secrets Manager
      │                 │                 │
   Access           Encryption       Credentials
      │
      └────────── Lake Formation
                       │
                   Governance

            CloudWatch + CloudTrail
              Monitoring / Audit

                 ORCHESTRATION
        Step Functions / Glue Triggers
               / EventBridge
```

> **Final interview mindset:**  
> **Problem → workload characteristics → AWS capability → why it fits → tradeoffs → operational considerations.**
