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