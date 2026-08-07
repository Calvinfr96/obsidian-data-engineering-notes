## S3

### Overview
- Object storage, **not** a database.
- Stores structured, semi-structured, and unstructured data.
- Excellent for data lakes, logs, backups, media, and analytics datasets.
- Very low cost and highly durable.
- Choose S3 for **large datasets**.
- Choose DynamoDB for **low-latency reads/writes on individual records**.
- Service selection should be driven by **workload and business requirements**, not cost alone.

### When to Use S3
- Use S3 when you need:
	- Highly scalable object storage
	- Low-cost storage
	- Data lakes
	- Backups
	- Log storage
	- Images/videos/documents
	- Analytics datasets
- Don't use S3 for:
	- Low-latency transactional reads/writes
	- Databases
	- Frequently updated individual records
- Think:
> **S3 stores files (objects), not database records.**

### Objects, Buckets, and Keys
- Bucket = Container
- Object = File
- Key = Unique object path
- **S3 doesn't have actual folders. Folders are simply prefixes in object keys.**
- Flat object store (no real folders).
- Objects are identified by unique keys.
- "Folders" are prefixes shown by the AWS Console.
- Organize objects using meaningful prefixes (for example, `year=2026/month=08`).
- Good partitioning enables **partition pruning**.
- Partition pruning reduces the amount of data read, improving performance and lowering cost.
- Think about both **write efficiency** (avoiding overwrites, idempotent ingestion) and **read efficiency** (fast analytics).

### Partitioning
- Organize data using meaningful prefixes.
- Partition based on **query patterns**, not arbitrary fields.
- Goal: enable **partition pruning**.
- Avoid high-cardinality partition keys unless the workload truly justifies them.
- Avoid millions of tiny partitions (small files problem).
- Avoid partitions that are too large.
- Good partitioning reduces data scanned, lowers cost, and improves query performance.
- Choose partition keys based on:
	- Query patterns
	- Data distribution
	- Partition size

### Storage Classes
- Choose storage classes based on **access patterns**, not just cost.
- S3 Standard → Frequently accessed data.
- S3 Standard-IA → Infrequently accessed but still immediately available.
- Glacier / Glacier Deep Archive → Long-term archival with lower storage costs and longer retrieval times.
- Use **Lifecycle Policies** to automatically transition objects between storage classes.
- Select archival storage based on **retrieval requirements**, not simply the lowest price.
- Automate storage management whenever possible.

|Storage Class|Best For|
|---|---|
|Standard|Frequently accessed|
|Standard-IA|Occasionally accessed|
|Glacier Flexible Retrieval|Archival|
|Glacier Deep Archive|Compliance / long-term retention|
- Choose based on:
	- Access frequency
	- Retrieval latency requirements
	- Storage costs

### Lifecycle Polices
- Automate:
	- Storage class transitions
	- Expiration
	- Cleanup
- Think: "Don't **manually** move data."

### Versioning
- Versioning keeps multiple versions of the same object.
- Enables rollback after accidental overwrite.
- Deleted objects become **delete markers**, allowing recovery.
- Versioning protects against user mistakes.
- IAM helps **prevent** mistakes.
- CloudTrail helps determine **who** made the change.
- Versioning is **not** a replacement for backups or disaster recovery.
- Protects against:
	- Accidental overwrite
	- Accidental deletion
- Remember:
	- Previous versions remain
	- Deletes create delete markers
- **Vesioning is for recovery, NOT prevention**.


### Event Notifications
- Event Notifications allow S3 to trigger downstream workflows after object events.
- Common events: Object Created, Object Deleted, Object Restored.
- Common targets: Lambda, SQS, SNS, EventBridge.
- Prefer asynchronous processing for long-running or independent tasks.
- Event-driven architectures improve scalability, reliability, and user experience.
- Keep services **loosely coupled** by letting S3 publish events instead of having the application orchestrate downstream processing.
- Event Notifications reduce the risk of missed processing if an application fails after a successful upload.
- Trigger workflows when:
	- Object created
	- Object deleted
	- Object restored
- Common use cases:
	- Thumbnail generation
	- ETL pipelines
	- Metadata extraction
	- Notifications

### File Formats
- **JSON**:
	- Pros:
		- Human readable
		- APIs
		- Debugging
	- Cons:
		- Large
		- Slow analytics
- **Parquet**:
	- Pros:
		- Columnar
		- Compressed
		- Predicate pushdown
		- Column pruning
	- Best for:
		- Analytics
- **Avro**:
	- Best for:
		- Streaming
		- Schema evolution
- **Protobuf**:
	- Best for:
		- Microservice communication

### Security
- Apply the **Principle of Least Privilege**.
- Use:
	- IAM roles
	- Temporary credentials
	- Bucket (resource-based) policies
- Grant only the permissions required for each role.
- Use **IAM Roles** and temporary credentials instead of long-lived access keys.
- Assign permissions through **roles/groups**, not individual users whenever practical.
- Use **Bucket Policies** to enforce resource-level access rules.
- IAM helps **prevent** mistakes; Versioning helps **recover** from them.
- Security should be layered: IAM + Versioning + CloudTrail + Encryption.

### Performance Considerations
- Avoid the **small files problem**.
- Many small files increase:
    - S3 request overhead
    - Metadata operations
    - Spark task scheduling overhead
- Small Files Problem:
	- Symptoms:
		- Many tasks
		- High scheduling overhead
		- High S3 request count
	- Solution:
		- Compaction
	- Also consider:
		- Partition pruning
		- Predicate pushdown
		- Column pruning
		- Appropriate file sizes
- Partitioning and file size should be designed together.
- Aim for reasonably sized Parquet files rather than extremely small or extremely large ones.
- Performance bottlenecks often come from **I/O and coordination overhead**, not just the amount of data.

### S3 vs. Other Storage
| Service | Best For                               |
| ------- | -------------------------------------- |
| S3      | Object storage, analytics, backups     |
| EBS     | EC2 block storage, databases           |
| EFS     | Shared filesystem across EC2 instances |

### Top 10 Interview Takeaways
1. **Always start with the workload.**
    - How is the data accessed?
    - How often?
    - By whom?
2. **Choose storage based on access patterns, not cost alone.**
3. **Choose partition keys based on query patterns.**
4. **Optimize by reading less data.**
    - Partition pruning
    - Column pruning
    - Predicate pushdown
    - Compression
5. **Automate operations whenever possible.**
    - Lifecycle Policies
    - Event-driven workflows
6. **Use preventive controls before corrective controls.**
    - IAM first
    - Versioning second
7. **Think in terms of tradeoffs, not absolutes.**
    - There is rarely a "best" solution.
8. **Measure before optimizing.**
    - Use metrics and evidence.
    - Don't guess.
9. **Separate responsibilities.**
    - Upload service uploads.
    - Processing service processes.
    - Analytics service analyzes.
10. **Be willing to revisit your design when requirements change.**

## DynamoDB

### Overview
- Designed for **transactional workloads**, not large analytical scans.
- Optimized for **single-digit millisecond** reads and writes.
- Automatically scales to handle changing traffic.
- Highly available across multiple Availability Zones.
- Serverless with minimal operational overhead.
- Flexible schema (NoSQL).
- Choose DynamoDB when your workload primarily accesses **individual items**, not large datasets.
- Use DynamoDB when you need:
	- High-throughput reads and writes
	- Low-latency access to individual records
	- Automatic scaling
	- Flexible NoSQL schema
- Don't use DynamoDB for:
	- Complex joins
	- Ad hoc analytical queries
	- Large table scans
	- Those are usually better suited for services like S3, Athena, or a data warehouse.
- Typical workloads:
	- Shopping carts
	- User profiles
	- Session stores
	- Product catalogs
	- IoT devices
	- Gaming leaderboards

### Partition Keys
- A partition key determines where an item is stored.
- DynamoDB hashes the partition key to map items to physical partitions.
- High-cardinality keys generally distribute data more evenly.
- Choose partition keys that:
    - Match access patterns.
    - Have many unique values (high cardinality).
    - Distribute traffic evenly.
- Avoid keys that create hot partitions.
- A partition key does **not** correspond directly to a physical partition.

### Hot Partitions
- Evenly distributed **data** does not guarantee evenly distributed **traffic**.
- A single frequently accessed partition key can become a hotspot.
- High cardinality helps, but it does not eliminate hot partitions.
- Choose partition keys that distribute **both data and access patterns**.
- Hot partitions are a form of **workload skew**, similar to data skew in Spark.
- Possible mitigations:
	- Choose a better partition key.
	- Use write sharding (salting) when appropriate.
	- Introduce caching for read-heavy hot keys.

### Sort Keys
- A sort key allows multiple related items to share the same partition key.
- Items within a partition are ordered by the sort key.
- Choose a sort key that supports the application's access pattern.
- Common sort keys:
    - Timestamp
    - OrderDate
    - Version
    - SequenceNumber
- The combination of **Partition Key + Sort Key** uniquely identifies an item.
- Use sort keys to support:
	- Recent items
	- Time-range queries
	- Ordered history

### Access Pattern Modeling
- Model access patterns, not entities.
- Always ask the following before designing a table:
	- What questions does the application need to answer? 
- When designing a table:
	1. List every application query.
	2. Choose a partition key that supports the most important access pattern.
	3. Choose a sort key that organizes related items naturally.
	4. Accept that one primary key usually can't optimize every query.
	5. Add secondary indexes when additional access patterns become important.

### Global Secondary Indexes (GSIs)
- A GSI provides an alternative way to access the same data.
- It has its own partition key (and optional sort key).
- Use a GSI when a new access pattern emerges.
- Prefer adding a GSI over changing the primary key if the original workload is still important.
- Think of a GSI as **another view of the same data**, automatically maintained by DynamoDB.
- Tradeoffs:
	- Additional write overhead.
	- Additional storage.
	- Additional operational complexity.
- **Don't create a GSI "just in case"**.

### Consistency
- **Strong Consistency**:
	- Always returns the latest committed data.
	- Use when stale data could lead to incorrect behavior.
	- Examples:
	    - Banking
	    - Inventory
	    - Shopping carts
	    - Account balances
- **Eventual Consistency**:
	- May briefly return stale data.
	- Lower latency and better scalability.
	- Use when stale data is acceptable and has little business impact.
	- Examples:
	    - Social media likes
	    - Weather
	    - Analytics dashboards
	    - Product reviews
- **Decision Rule**:
> 	Choose the weakest consistency model that still satisfies the business requirements.

### Transactions
- Use transactions when:
	- Multiple related writes must succeed together.
	- Partial success would create an incorrect business state.
- Examples:
	- Order creation + inventory update.
	- Money transfers.
- Avoid transactions when:
	- Only one item is modified.
	- Operations are independent.
	- Side effects (emails, notifications, analytics events) can be handled asynchronously.
- **Decision Rule**
> 	Use transactions to protect business correctness, not simply because an operation is important.

### Capacity Modes & Throughput
- **Provisioned Capacity**
	- Use when:
		- Traffic is stable and predictable.
		- Throughput can be estimated.
		- Cost optimization is important.
		- Scheduled scaling can prepare for known spikes.
	- Lowe cost, but requires capacity planning.
- **On-Demand Capacity**
	- Use when:
		- Traffic is bursty and unpredictable.
		- Large spikes may occur.
		- Simplicity is preferred over manual capacity planning.
	- Simpler operationally, generally more expensive per request.
- **Decision Rule**
> 	Choose based on workload predictability, not simply traffic volume.

### Time to Live (TTL)
- Use TTL when:
	- Data has a natural expiration time.
	- Expiration is deterministic.
	- Manual cleanup would add unnecessary operational overhead.
- Common examples:
	- Session tokens
	- Password reset links
	- Shopping carts
	- Idempotency keys
	- Temporary caches
- Avoid TTL when:
	- Data has legal or compliance retention requirements.
	- Deletion must occur at an exact moment.
	- The expiration policy depends on human judgment.
	- Records should be kept permanently.
- **Decision Rule**
> 	Use TTL to automate the lifecycle of temporary data.

### Streams
- Use Streams when changes to data should trigger downstream processing.
- Common use cases:
	- Send notifications
	- Update analytics
	- Synchronize and notify other systems
	- Trigger Lambda functions
	- Build event-driven architectures
- Benefits:
	- Loose coupling
	- Independent scaling
	- Failure isolation
	- Easier maintenance
- **Decision Rule**
> 	Let the application write data. Let downstream services react to changes independently.

### Dynamo DB Accelerator (DAX)
- Think of DAX as:
> 	A managed cache in front of DynamoDB.

- Use DAX when:
	- Workload is read-heavy (reads greatly outnumber writes).
	- Data changes infrequently.
	- Low latency is important.
	- Cache hit ratio is expected to be high.
- Avoid DAX when:
	- Data changes very frequently.
	- Every read must reflect the latest value.
	- The cache hit ratio would be low.
- **Decision Rule**
> 	Cache data that is read often and changes infrequently.

### 10 Interview Rules
1. **Design arount access patterns**. Not around the schema.
2. Choose partition keys that distribute **traffic**, not just data.
3. High cardinality helps. **Even traffic distribution matters more**.
4. A GSI is another access path. Not a replacement for the primary key.
5. Choose the **weakest** consistency model that satisfies the business requirements.
6. Use transactions only when multiple pieces of business state must remain consistent.
7. Provisioned Capacity is for predictable workloads. On-Demand is for unpredictable workloads.
8. TTL automates the lifecycle of temporary data. It is **best effort**, not instantaneous.
9. Use Streams to build event-driven architectures. Don't tightly couple your application to downstream services.
10. **Think in tradeoffs**. There is almost never a universally "correct" AWS service or feature.