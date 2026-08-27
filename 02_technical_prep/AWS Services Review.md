# Amazon Simple Service Storage (S3)

## Overview

- Almost every AWS data pipeline looks something like this:
	```
	Producer
	    │
	    ▼
	SQS / SNS
	    │
	    ▼
	Lambda / Glue
	    │
	    ▼
	S3
	    │
	    ▼
	Analytics / ML / Reporting
	```
	- Everything revolves around S3.
- Amazon S3 is AWS's highly durable, massively scalable object storage service. Unlike a tradition filesystem, S3 stores **objects**, not files on disks.
- An S3 object consists of:
	- Data
	- Metadata
	- A unique key
	- Example:
		```
		Bucket: customer-orders
		
		Key:
		2026/08/04/orders.parquet
		
		Object:
		orders.parquet
		```
		- The AWS console displays the `/` characters like folders, but S3 is fundamentally a flat key-value namespace within each bucket.
- **Why Data Engineers Love S3**:
	- S3 is often called the **data lake** for AWS because it excels at storing enormous volumes of data cheaply and reliably.
	- It works well for:
		- Raw event data
		- JSON logs
		- CSV files
		- Parquet datasets
		- Images
		- ML training data
		- Backups
		- Streaming outputs
	- It's designed for **high durability**, **virtually unlimited scale**, and **low operational overhead**, making it a natural landing zone for both raw and processed datasets.

### Comparison to DynamoDB
| S3                                       | DynamoDB                                                |
| ---------------------------------------- | ------------------------------------------------------- |
| Object storage                           | NoSQL database                                          |
| Optimized for storing files and datasets | Optimized for storing and retrieving individual records |
| Very low storage cost                    | Higher cost per GB                                      |
| Great for analytics                      | Great for transactional workloads                       |
| Seconds to stream huge files             | Millisecond lookups for individual items                |

### Knowledge Check Questions
1.  Why would you store data in S3 instead of DynamoDB?
> 	I would choose S3 when I need to store large volumes of structured, semi-structured, or unstructured data at a low cost. S3 is highly durable, virtually unlimited in capacity, and well suited for data lakes, backups, logs, and analytics workloads. DynamoDB, on the other hand, is a NoSQL database optimized for low-latency, high-throughput reads and writes on individual records. If an application needs to retrieve or update specific items in milliseconds, DynamoDB is a better fit. If I'm primarily storing large datasets for batch processing or analytics, S3 is usually the better choice.
	- Both S3 and DynamoDB are highly scalable services. However, S3 scales extremely well for storing massive datasets, while DynamoDB scales for high-throughput transactional access to individual items.
2. If S3 is so much cheaper than DynamoDB, why don't companies just store everything in S3?
> 	Although S3 is much cheaper, it's designed for object storage rather than serving as an operational database. S3 is ideal for storing large datasets, logs, backups, and files that analytics engines like Spark or Glue can process. DynamoDB, on the other hand, is optimized for low-latency reads and writes on individual records. If an application needs to quickly retrieve or update a user's profile, an order, or a shopping cart, DynamoDB is the better choice. Ultimately, the decision depends on the workload and business requirements, not just storage cost.
	- When answering technical questions during an interview, try to lead with **decisions**, not **facts**.
	- For example, say "I would choose S3 if my goal is to store large datasets for analytics or archival. If I need millisecond reads and writes on individual records, I'd choose DynamoDB instead." Instead of "S3 is cheaper and more scalable."
	- Use the decision > reasoning > tradeoff structure to answer questions. This is how senior engineers answer architectural questions because it demonstrates **decision making** rather than knowledge.

## Objects, Buckets, and Keys

- A **bucket** is the top-level container for storing objects.
- Think of a bucket as a namespace. For example:
	```
	company-raw-data
	
	company-processed-data
	
	ml-training-data
	```
	- This is a common practice used to separate data by purpose.
- **Objects**:
	- Everything stored in S3 is an **object**. An object consists of:
		- Data
		- Metadata
		- Unique key
	- Object metadata example:
		```
		Bucket:
		company-raw-data
		
		Key:
		sales/2026/08/05/orders.parquet
		```
		- The object itself might contain 500MB of Parquet data.
- **Keys**:
	- S3 does **not** actually store objects within a bucket in individual folders.
	- When you see an S3 key such as `sales/2026/08/05/orders.parquet`, it represents a **single** object, not a directory.
	- The `/` characters are simply part of the key.
	- Creating "folders" in S3 is really just creating keys with a consistent naming convention.
	- That convention becomes extremely important for analytics engines like Spark and Glue because they can efficiently locate relevant data based on key prefixes.
- **Prefixes**:
	- An object **key prefix** is the technical term for an S3 "folder".
	- Example:
		```
		sales/2026/01/file1.parquet
		sales/2026/02/file2.parquet
		sales/2026/08/file3.parquet
		```
		- All of these Parquet files share the same `sales/2026/` prefix. They're **not** stored in shared folder within the S3 bucket.
	- Naming prefixes appropriate is also crucial it allows tools like Spark and Glue to skip irrelevent data.
	- Example:
		```
		sales/
		    year=2025/
		        month=01/
		        month=02/
		    year=2026/
		        month=01/
		        month=02/
		```
		- `year=` and `month=` are explicitly used in the key name to allow Spark to perform partition pruning while scanning data.
	- Think of the directory-like structures as **partition keys encoded into object names**.

### Knowledge Check Questions
1. Does S3 actually have folders?
> 	S3 doesn't actually have folders. Every object is stored using a unique key, such as `sales/year=2026/month=08/orders.parquet`. The AWS Console displays the slash-delimited portions of the key as folders, but under the hood S3 is a flat object store. Those folder-like paths are really prefixes, which analytics tools like Spark and Glue can use to efficiently locate relevant data without scanning every object.
	- The "folders" primarily exist as a visual aid and are only used to construct the object key under the hood.
	- Answer the question using **mechanics** first, then explain **why**.
2. Imagine your company stores customer orders in S3.
	- One engineer suggests storing everything like this:
		```
		orders.parquet
		```
	- Another engineer suggests:
		```
		orders/
		    year=2026/
		        month=08/
		            day=05/
		                orders.parquet
		```
	- Why is the second approach better?
> 	The partitioned layout is much more efficient because it organizes the data according to common query patterns. If analysts frequently query data by year or month, Spark or Glue can prune entire partitions and read only the relevant files instead of scanning every object in the bucket. This significantly reduces I/O, improves query performance, and lowers processing costs. It also naturally organizes new data without overwriting existing objects.
	- When designing an S3 object key, a good engineer considers how the data will be **written** and **read**.
		- **Write path:** How do we ingest data safely? Will we overwrite existing objects? Is the write idempotent?
		- **Read path:** How will downstream systems efficiently consume the data?

## Partitioning

- Partitioning is simply organizing data so that queries can **avoid reading data they don't need**.
- Suppose you have sales data for five years:
	- You can simply store all sales data in one `all_orders.parquet` file. With this approach, every query needs to read the **entire dataset**.
	- Another approach would be to organize the data like so:
		```
		sales/
		    year=2025/
		        month=01/
		        month=02/
		    year=2026/
		        month=01/
		        month=02/
		```
		- Now, the storage layer matches common access patterns. If an analyst queries data for February 2026, Spark only needs to read objects that use that key prefix. It can skip all other objects.
		- This is very important because reading data is often the most expensive part of an analytics job.
- **Choosing Good Partition Keys**:
	- A **good partition key** is one that **matches common data access patterns**. 
	- For example, suppose analysts constantly ask:
		- Sales by month
		- Sales by year
		- Last week's orders
	- A great partitioning strategy would be: `year={YYYY}/month={MM}`.
	- If analysts frequently queries sales data by country, a good partition would be: `country={country_name}`.
	- A **bad partition key** is one that has high cardinality and does not match common data access patterns.
	- For example, partitioning by `OrderID` would be terrible because each partition would contain almost no data. You'd create **millions of tiny partitions**. This is called the **small files problem**.
	- **Rule of Thumb**:
		- Choose partition keys that:
			- Are commonly used in filters
			- Have reasonable cardinality
			- Create balanced partition sizes
		- Avoid partition keys that create:
			- Millions of tiny partitions
			- One or a few massive partitions
### Knowledge Check Questions
1. How do you choose a good partition key for S3?
> 	I choose partition keys based on how the data is commonly queried. The goal is to organize the data so query engines like Spark or Glue can prune irrelevant partitions and read only the data they need. I also want partition sizes to be reasonable, avoiding both very small partitions that create unnecessary overhead and very large partitions that require scanning excessive amounts of data.
	- S3 partition keys aren't necessarily designed to **evenly distribute** data like DynamoDB partition keys. S3 partition keys are mainly designed to match data access patterns and create reasonably sized partitions. Remember, S3 isn't performing any of the work when data is queried, so a hot partition isn't as much of a problem.
2. Would `UserID` make a good partition key?
> 	It depends on the workload. UserID is often a high-cardinality field, so partitioning by it can create millions of very small partitions, which increases metadata overhead and hurts query performance. However, if user-specific queries are extremely common and each user generates a substantial amount of data, it could be a reasonable choice. In general, I choose partition keys based on query patterns while avoiding partition strategies that create too many small partitions or partitions that are too large.

## Storage Classes

- S3 offers several different storage classes based on cost, availability (retrieval delay), and access frequency.
- Data can be manually moved between tiers, or it can be automatically moved between tiers based on retrieval patterns. This is accomplished using S3 Lifecycle Policies.
- The four major S3 storage classes are:
1. **S3 Standard**:
	- Used when:
		- Frequently accessed
		- Low latency required
	- Examples:
		- Current customer orders
		- Application assets
		- Active datasets
	- Highest storage cost. Lowest retrieval latency.
2. **S3 Standard-IA (Infrequent Access)**:
	- Used when:
		- Data is still important
		- Accessed occasionally
		- Must be immediately available
	- Examples:
		- Last year's sales reports
		- Older datasets
		- Backup files
	- Storage is cheaper. Retrieval incurs a small cost.
3. **S3 Glacier Instant Retrieval / Flexible Retrieval**:
	- Examples:
		- Historical logs
		- Audit records
		- Old reports
	- Much cheaper. Retrieval may have additional cost and, depending on the Glacier tier, can take from milliseconds (Instant Retrieval) to **minutes or hours** (Flexible Retrieval).
4. **Glacier Deep Archive**:
	- Used for:
		- Compliance
		- Legal retention
		- Disaster recovery copies
	- Cheapest storage tier.

| Your Tier | AWS Storage Class                         | Typical Use                                 |
| --------- | ----------------------------------------- | ------------------------------------------- |
| Tier 1    | **S3 Standard**                           | Frequently accessed data                    |
| Tier 2    | **S3 Standard-IA** (Infrequent Access)    | Older data that's still occasionally needed |
| Tier 3    | **S3 Glacier** / **Glacier Deep Archive** | Long-term archival and compliance           |

### Lifecycle Policies
- A **Lifecycle Policy** automatically performs actions on objects based on rules that you define.
- Common actions include:
	- **Transition** objects to a different storage class after a certain number of days.
	- **Expire** (delete) objects after a retention period.
	- **Abort incomplete multipart uploads** to clean up unused storage.
- Lifecycle policies save both money and operational effort because they automatically once configured.
- For example:

|Object Age|Action|
|---|---|
|0–30 days|S3 Standard|
|31–365 days|S3 Standard-IA|
|>365 days|Glacier|
|>7 years|Delete (if permitted by retention policies)|

### Knowledge Check Questions
1. Your company stores customer invoices in S3.
	- One afternoon, an application bug accidentally overwrites thousands of invoice PDFs with blank files.
	- The developers fix the bug quickly, but now all of yesterday's invoices are gone.
	- The CTO asks: "How can we prevent this from happening again?" How would you solve this problem?
> 	Similar to commits in a git repository, I would implement a feature that tracked how and when an object in S3 was modified and I would allow objects to be rolled back to a previous state if they're accidentally updated or deleted. This would be simpler and more efficient than storing backups of previous versions in a lower-cost storage tier, such as Glacier, since this is meant for disaster recovery and likely wouldn't contain recent versions of an object that are frequently accessed. To avoid users accidentally deleting or overwriting important data, I would implement IAM policies using the principle of least privilege to ensure only authorized users can modify important data. Using a versioning system similar to git would not require engineers to manually create backups.
2. What's the difference between Glacier Instant Retrieval and Glacier Flexible Retrieval?
> 	I know AWS offers multiple Glacier tiers that trade storage cost for retrieval speed. For my experience level, I would evaluate the retrieval requirements and choose the appropriate Glacier tier rather than memorizing every pricing difference.
3. Suppose your manager says: "Our finance department is complaining because our S3 bill keeps increasing every month. We almost never access data older than one year." What would you recommend?
> 	I would use an S3 Lifecycle Policy to automatically transition objects to less expensive storage classes as they age. For example, new data might remain in S3 Standard, transition to Standard-IA after 30 days, and then move to Glacier or Glacier Deep Archive after a year, depending on the business's retrieval requirements. This reduces storage costs while avoiding manual data management, and the exact storage class should be chosen based on how quickly the data needs to be retrieved.

## Versioning

- Suppose you upload `invoice.pdf` to an S3 bucket:
	- Version 1 is created.
	- Later, you upload another file **with the same key**.
	- Instead of replacing Version 1, Version 2 is created.
	- Both versions exist at the same time. Version 2 becomes the current version.
	- If Version 2 becomes corrupted, you can restore Version 1.
- This feature is called **S3 Versioning**.
- When versioning is enabled, S3 keeps track of each version of an object uploaded to an S3 bucket.
	- When an object is deleted in a bucket with versioning enabled, S3 does not permanently delete the object.
	- Instead, S3 creates a **delete marker**. With this, the object appears deleted, but can be recovered by removing the marker.
	- This is one of the biggest benefits of versioning.

### Knowledge Check Questions
1. If Versioning is enabled, why would anyone still need backups?
	- Versioning protects against:
		- Accidental overwrite
		- Accidental deletion
	- Versioning does **not** replace:
		- Cross-region disaster recovery
		- Protection against bucket deletion
		- Long-term archival
		- Business continuity planning
	- Versioning is a **safety net**, not a disaster recovery tool.

## File Formats

- S3 supports object storage in all major file formats, including:
	- JSON
	- Avro
	- Protobuf
	- Parget
- Related Notes: [[Data Modeling Fundamentals#5. Serialization & Data Formats|Serialization & Data Formats]]

### Knowledge Check Questions
1. Currently, all data is stored as JSON. Analysts complain that:
	- Queries are slow.
	- Glue jobs take much longer than expected.
	- Processing costs continue to increase.
	- The CTO asks: "Can we improve performance without buying larger servers?" How would you approach this problem?
> 	Since the primary workload is analytics rather than debugging, I would store the data in Parquet instead of JSON. Parquet is a columnar format, so query engines only read the columns required by a query instead of every field in each record. It also supports predicate pushdown and column pruning, reducing the amount of data scanned. I would partition the data by event date so Spark or Glue can prune entire partitions before reading files. Finally, Parquet supports efficient compression, which further reduces storage costs and the amount of data read from S3. Together, these changes would improve performance while lowering processing costs without requiring larger compute resources.
2. Parquet is obviously better than JSON, so we should always store everything in Parquet. Would you agree? Why or why not?
> 	I wouldn't say Parquet is always better. It depends on the workload. Parquet is excellent for analytical workloads because it's columnar, compressed, and supports predicate pushdown and column pruning. However, JSON is still a good choice for APIs, application logs, debugging, and event ingestion because it's human-readable and easy to work with. In many production systems, data is initially ingested as JSON and later transformed into Parquet for analytics. The storage format should be chosen based on how the data will be used.
	- Many modern architectures use Parquet **and** JSON:
		```
		Application
		      │
		      ▼
		JSON Events
		      │
		      ▼
		Kafka
		      │
		      ▼
		S3 (Raw JSON)
		      │
		      ▼
		Glue / Spark ETL
		      │
		      ▼
		Parquet
		      │
		      ▼
		Analytics
		```
	- The application emits JSON because it's easy to serialize, debug, and integrate with APIs.
	- The analytics layer converts it to Parquet because it's much more efficient for large-scale queries.
	- JSON excels in the following areas:
		- APIs (responses are formatted in JSON)
		- Log files (more human-readable)
		- Streaming:
			- Many streaming services transmit data in JSON, Avro, or Protobuf format.
			- This is because they're transmitting **individual events**, not performing analytical queries.
	- Parquet becomes valuable when you're storing large datasets for analytical queries.

## Event Notifications

- S3 Event Notifications are one tool that facilitate the design and creation of event-driven data pipelines in AWS.
- A simple implementation of an event-driven architecture in AWS would look like:
	```
	User
	   │
	   ▼
	Upload Image
	   │
	   ▼
	S3 Bucket
	   │
	   ▼
	S3 Event Notification
	   │
	   ▼
	Lambda
	   │
	   ├────────► Generate Thumbnail
	   │
	   ├────────► Scan for Malware
	   │
	   ├────────► Extract Metadata
	   │
	   └────────► Store Metadata
	```
- An **S3 Event Notification** is a mechanism that publishes an event whenever something happens in a bucket.
- Examples of bucket events includeL
	- Object Created
	- Object Deleted
	- Object Restored
	- Multipart Upload Completed
- Those events can trigger downstream processing.
- Common targets include:
	- Lambda
	- SNS
	- SQS
	- EventBridge
- S3 Event Notifications allow for **loose coupling**. If the owner of the image-processing service wanted to add AI image tagging, they wouldn't need to change the upload service to support it. They'd only need to modify the backend processing service.

### Knowledge Check Questions
1. Your company allows customers to upload profile pictures. The current workflow looks like this:
	```
	Customer
	    │
	    ▼
	Upload Image to S3
	```
	- The problem is that the uploaded images are often very large. The application needs to:
		- Generate a thumbnail.
		- Scan the image for malware.
		- Extract metadata (dimensions, file type, etc.).
		- Store the metadata in a database.
		- One engineer proposes: "After every upload, the application should immediately perform all of these tasks before returning a response to the user."
		- Another engineer proposes: "Let's upload the image first, then process it asynchronously."
		- Which approach would you recommend, and why?
		- Think about:
			- User experience
			- Scalability
			- Reliability
			- Coupling between components
			- What happens when one processing step fails
> 	I would recommend processing the image asynchronously because it provides a better user experience, is more scalable and reliable, decouples image uploading from image processing, and allows processing errors to be handled gracefully. Instead of having the user wait for a response, an "image processing..." message can be sent back immediately to signal to the user the upload was successful. Asynchronous processing allows multiple images to be uploaded by multiple users at the same time and processed concurrently by a sufficiently-scaled backend service. This saves a lot of time compared to uploading an image and having to wait for processing to succeed or fail before making another upload. If processing fails, a message can be sent back informing the user and asking them to try again. If processing succeeds, a confirmation message can be sent informing the user that processing is complete.
2. Suppose the interviewer asks: "Why use an S3 Event Notification instead of having the application call the Lambda function directly after uploading the image?"
> 	I would prefer using an S3 Event Notification because it separates the responsibility of uploading an object from processing it. The application only needs to upload the file successfully. Once S3 confirms the object has been created, it automatically emits an event that triggers downstream processing. This is more reliable because it avoids situations where the application crashes after the upload but before invoking Lambda. It also creates a loosely coupled architecture, making it easy to add additional processing steps, such as AI tagging or image moderation, without modifying the upload service.
	- Letting S3 trigger the Lambda instead of the application is an example of **loose** coupling, not tight coupling.

## Security

- There are two common ways to grant access to S3:
	- **Identity-Based Policies (IAM)**:
		- Attached to:
			- Users
			- Roles
			- Groups
	- **Bucket Policies**:
		- Attached to the bucket itself.
		- Examples:
			- Only requests from this AWS account may access this bucket.
			- Deny any request that isn't encrypted.
			- Allow this partner account to read these objects.

### Common AWS Security Implementations

| Requirement                | AWS Solution                                |
| -------------------------- | ------------------------------------------- |
| Least privilege            | IAM Policies                                |
| Temporary credentials      | IAM Roles + STS                             |
| Developers vs Production   | Separate IAM roles / accounts               |
| Prevent accidental deletes | IAM + Versioning + (optionally) Object Lock |
| Maintainability            | IAM Groups / Roles                          |

### Knowledge Check Questions
1. Your company has an S3 bucket that stores customer financial reports. Several different systems need access:
	- **Web Application**
	    - Can upload reports.
	    - Can download reports.
	    - Cannot delete reports.
	- **Analytics Pipeline**
	    - Can read reports.
	    - Cannot modify them.
	- **Backup Service**
	    - Can read everything.
	    - Can write backup copies.
	    - Cannot delete production data.
	- **Developers**
	    - Need access in development.
	    - Should **not** have unrestricted access to production customer reports.
	- The CTO asks: "How would you design access to this bucket?"
	- Think about:
		- Should everyone get full access?
		- How would you prevent accidental deletion?
		- How would you minimize security risk if credentials were compromised?
		- How would you make the permissions easy to maintain as the system grows?
> 	I would design access to the bucket using the Principle of Least Privilege. An entity is only given access to bucket resource it needs to fulfill its duties, not all bucket resources. To prevent accidental deletes, I would only grant delete permissions to entities that required it and appropriately restrict the circumstances under which a delete could occur. To minimize security risks in the event of a leak, I would configure requests to use a non-reusable token for authentication. To make permissions easy to maintain, I would assign permissions at a group level, instead of granular permissions at an individual entity level.
2. If Versioning already lets us recover deleted objects, why do we still care about IAM permissions?
> 	IAM and bucket policies are preventive controls, while Versioning is a corrective control. My goal is to prevent accidental deletions in the first place by applying the Principle of Least Privilege and restricting delete permissions to only the identities that truly require them. Versioning is still valuable because mistakes can happen, but recovering deleted objects is a manual operational task that consumes time and introduces risk. I'd rather prevent the deletion than rely on recovery afterward.

## Performance Considerations

- As with all AWS services, optimizing storage in S3 comes down to balancing cost, performance, and business objectives:

| Too Little         | Too Much                       | Goal                                     |
| ------------------ | ------------------------------ | ---------------------------------------- |
| One giant file     | Millions of tiny files         | Balanced file sizes                      |
| One huge partition | Millions of tiny partitions    | Balanced partitions                      |
| One storage class  | Deep Archive for everything    | Storage classes based on access patterns |
| No permissions     | Overly restrictive permissions | Least privilege                          |

### Knowledge Check Questions
1. Your team has built a nightly Glue job that processes clickstream data stored in S3. Recently, the job slowed down dramatically. After investigating, you discover:
	- The dataset is still **1 TB**.
	- The partitioning strategy hasn't changed.
	- The cluster size hasn't changed.
	- However, instead of **100 files that are 10 GB each**, the dataset now has **1,000,000 files that are 1 MB each**.
	- Your manager says: "It's still only 1 TB. Why is it so much slower?" How would you explain what's happening?
	- Think about:
		- Why file count matters
		- What Spark or Glue has to do before it processes the data
		- Whether this reminds you of any concept we've already discussed
> 	Although the total data volume is unchanged, each file introduces fixed overhead. Spark must discover the object, schedule a task, issue an S3 request, retrieve metadata, and open the file before it can begin processing. When the dataset is split into one million tiny files, those fixed costs dominate the workload. Instead of spending most of its time processing data, Spark spends much of its time managing files and scheduling tasks. This is known as the small files problem and can significantly increase both execution time and S3 request costs.
	- Files in S3 are already serialized (presumably in Parquet format), so Spark doesn't need to spend time serializing and deserializing each file.
	- The bigger issue is that **each file incurs fixed overhead before Spark can process its contents**.
	- For **every file**, Spark (and S3) has to:
		- Discover the object.
		- Read its metadata.
		- Open a connection/request.
		- Schedule a task.
		- Read the file.
		- Close the request.
	- If each file is only **1 MB**, that fixed overhead becomes a large percentage of the total work.
	- Additionally, Spark typically creates **one task per input split**, and with many small files that often approaches one task per file.
	- Now, Spark needs to schedule and orchestrate 1,000,000 tasks instead of 100. This introduces:
		- Scheduler overhead
		- Executor startup overhead
		- Metadata lookups
		- Many more S3 GET requests
		- More network round trips
2. Okay, then why don't we just combine everything into one gigantic 1 TB Parquet file? That would eliminate all the overhead, right?
> 	A single massive file has the opposite problem of many tiny files. While Spark can often split large files to some extent, having only a few very large files generally reduces the amount of parallel work available compared to a well-balanced dataset. Spark performs best when the data is organized into a reasonable number of appropriately sized files that allow many executors to work concurrently without creating excessive scheduling overhead. The goal isn't the fewest files possible—it's the right balance between parallelism and per-file overhead.
3. Suppose your Glue job has started slowing down, but you don't know why. Walk me through your debugging process.
	- Think about:
		- Partitioning
		- File sizes
		- Storage format
		- Lifecycle policies (if relevant)
		- Access patterns
		- Data growth
		- Query patterns
> 	I would begin by identifying where the bottleneck is rather than assuming the cause. I'd review job metrics such as executor utilization, task counts, stage durations, and the amount of data read from S3. If only a few executors are busy, I'd investigate data skew or poorly chosen partition keys. If all executors are busy, I'd determine whether the workload has increased because of more data, more files, or changes in query patterns. I'd also verify that the data is still stored in Parquet, that partition pruning is still effective, and that the average file size hasn't decreased enough to introduce the small files problem. Once I've identified the bottleneck, I'd choose an optimization targeted to that specific issue rather than making assumptions.
	- Other areas worth checking:
		- Did querying patterns change? If so, partition pruning and predicate pushdown may no longer be helping.
		- Did the file format change? Maybe someone started using JSON instead of Parquet.
		- Did someone change the partitioning strategy? Have millions of tiny partitions been created?
		- Did the average file size change?

## S3 vs. Elastic Book Store (EBS) vs. Elastic File System (EFS)

- Project A: **S3**
	- Requirements:
		- Huge datasets
		- Write once
		- Read many
		- Analytics
		- Spark
		- Glue
	- S3 was practically built for this.
	- Why not EBS? Because EBS is:
		- Attached to EC2
		- Block storage
		- Much more expensive for this workload
		- Not intended to be a 500 TB data lake
- Project B: **EBS**
	- Requirements:
		- PostgreSQL
		- Transaction logs
		- Low latency
		- Frequent writes
	- These requirements heavily imply the need for block storage.
	- This is because databases constantly:
		- Update blocks
		- Overwrite pages
		- Flush transaction logs
- Project C: **EFS**
	- The wording "50 EC2 instances sharing the same dataset" heavily implies the need for EFS.
	- This is because EFS is a shared network filesystem.
	- Every EC2 instance can mount it simultaneously.
	- An EBS volume generally attaches to a single EC2 instance (with some specialized exceptions that aren't relevant here).

### Mental Model
- S3:
	- Think:
		- "Am I storing objects?"
		- "I have files that I need to store."
	- Examples:
		- Images
		- Parquet
		- Logs
		- Videos
		- Backups
- EBS:
	- Think:
		- "Do I need a hard drive?"
		- "I have a disk that I need to store data on."
	- Examples:
		- PostgreSQL
		- MySQL
		- EC2 root volume
- EFS:
	- Think:
		- "Do multiple machines need the filesystem?"
		- "I have a shared network drive that needs to be used by multiple machines."
	- Examples:
		- Shared ML datasets
		- Shared application files
		- Multiple EC2 instances

### Knowledge Check Questions
1. Your company has three new projects:
	- Project A:
		- A data engineering team needs to store:
			- 500 TB of clickstream data
			- Historical logs
			- Parquet datasets
			- Daily backups
		- The data is mostly **written once and read many times** by Spark jobs.
	- Project B:
		- A backend application runs on EC2 instances. It stores
			- Application binaries
			- A PostgreSQL database
			- Transaction logs
		- The application requires **very low latency** reads and writes.
	- Project C:
		- A machine learning team has 50 EC2 instances training models simultaneously.
		- Every instance needs to read and write the **same shared dataset**.
	- The CTO asks: "Which AWS storage service would you choose for each project, and why?"
	- Think about:
		- Is this object storage or a filesystem?
		- Does the storage need to be shared?
		- Is low latency more important than scalability?
		- Are many machines accessing the same data?
> 	For Project A, I'd choose S3 because the workload is a large-scale analytics data lake. S3 provides highly durable, low-cost object storage and integrates well with Spark and Glue. Although EBS offers lower latency, it isn't designed to store hundreds of terabytes of analytical data economically.
>
> 	For Project B, I'd choose EBS because PostgreSQL requires low-latency block storage for frequent reads, writes, and transaction logs. This is the type of workload EBS is optimized for.
> 	
> 	For Project C, I'd choose EFS because it provides a shared filesystem that multiple EC2 instances can mount simultaneously. That's much simpler than trying to coordinate access through separate EBS volumes, and it's more appropriate than S3 when applications need traditional filesystem semantics.

## Mock Interview Questions

1. Imagine you're designing a data lake for an analytics platform. Why would you choose Amazon S3 instead of DynamoDB as the primary storage layer?
> 	Customer profiles are operational data. The application needs to retrieve and update individual records in milliseconds. DynamoDB is designed for that access pattern. S3 is optimized for storing and streaming objects, not for frequent low-latency lookups and updates of individual records. While S3 may be cheaper to store the data, the application would become more complex and provide a poorer user experience because it isn't the right storage model for this type of workload
2. Your team stores order data in S3.
	- Currently, the bucket looks like this:
		```
		orders/
		    order_000001.parquet
		    order_000002.parquet
		    order_000003.parquet
		    ...
		```
	- An analyst frequently runs queries like:
		```sql
		SELECT *
		FROM orders
		WHERE order_date BETWEEN '2026-08-01' AND '2026-08-31'
		```
	- The queries are becoming increasingly slow. What changes would you recommend to improve query performance, and why?
> 	Currently, orders seem to be partitioned partitioned by order ID. However, analysts are currently querying batches of orders based on order date. To improve query performance, I would recommend adjusting the partitioning pattern by partitioning orders by date, instead of order ID. This would greatly reduce the amount of data scanned during a query. Currently, all orders need to be scanned to scanned to find the records in the appropriate date range. Partitioning by date allows entire partitions to be skipped based on date while data is being scanned.
	- Suppose I tell you the data is already partitioned by date. What else would you investigate?
> 	Currently queries are pulling entire rows of order data. I would investigate whether this is really necessary and encourage analysts to only query the required columns. Since the data is stored in Parquet format, this would allow for further optimization by skipping entire columns of data within a partition.
	- Let's say the analysts really do need every column for this report, so we can't reduce the projection.
> 	If analysts really do need every column in each partition, I would investigate the number of files in each partition. If there are 1000s of tiny files in each partition, I would suggest compacting the data so that it is spread across fewer files. This would improve query performance by reducing I/O and scheduling overhead.
	- If the partitioning strategy was okay, all columns were needed by analysts, **and** the file size in each partition was optimal, the next avenue of investigation would be an increase in the amount of data being queried or a change in the query pattern.
3. Your team currently stores clickstream data in **JSON** files in S3. A senior engineer proposes converting everything to **Parquet**. Another engineer argues: "JSON is easier to read and debug. Let's just keep using JSON." What would you recommend, and what tradeoffs would you explain to the team?
> 	I would recommend switching to the file format that best suits the primary workload. If the clickstream data is primarily being manually analyzed by humans, JSON would be an excellent choice because of its human-readable format, although it consumes more memory and is harder to compress than other file formats. If the clickstream data is primarily being queried by analysts, I would recommend using Parquet because it is easier to compress than JSON and offers several advantages for well-partitioned datasets, including partition pruning, column pruning, and predicate pushdown. If clickstream data is primarily being used for streaming and needs good support for schema evolution, I would recommend Avro. If the data is primarily being transferred between micro-services, I would recommend protobuf.
	- Suppose the product manager says: "I don't want multiple formats. I want one format everywhere because it's simpler." Would you still recommend using only Parquet? Why or why not? (Assume you have control over the architecture.)
> 	I would not recommend using Parquet everywhere because it less human-readable than JSON, making it a lot harder to debug. I would suggest compromising by using Parquet in production environments, where it best suits the primary workload, and JSON in development environments.
	- A common production architecture looks like this:
		```
		Application
		      │
		      ▼
		JSON Events
		      │
		      ▼
		Raw S3 Zone (JSON)
		      │
		Glue / Spark
		      ▼
		Curated S3 Zone (Parquet)
		```
		- Raw events are stored in JSON, while curated data is stored in Parquet format. JSON isn't just used in development environment. Remember, **storage is cheaper than compute**. Storing raw events, which aren't analyzed in JSON makes sense because it's cheap to store. Storing curated in data, which is analyzed, in Parquet makes sense because it's cheap to store **and** analyze.
		- JSON is also a good choice for APIs and event ingestion, **not just debugging**.
4. Your team has an S3 bucket containing millions of customer documents. One morning an engineer accidentally runs a script that deletes **50,000 objects**. Fortunately, S3 Versioning is enabled. The engineer says: "No problem. We have Versioning, so we don't really need to worry about IAM permissions anymore." Do you agree or disagree? Why?
> 	Versioning is a corrective control that allows us to recover from mistakes when preventive controls fail. It should not be relied upon as a primary recovery tool for preventable mistakes, as the convenience of being able to rollback objects to a previous state comes at the expense of increased costs. A better approach would be to prevent accidental deletes from happening in the first place. Delete permissions should primarily be reserved for necessary production workflows, not scripts that are run manually in development environments. Delete permissions for these scenarios should involve single-use credentials that are only obtained after going through the proper chain of command and peer review.
	- Suppose the engineer responds: "But recovery only takes a few minutes. Why spend all this time designing IAM policies and approval processes when Versioning already solves the problem?" How would you convince them that prevention is still worth the effort?
> 	Versioning is an important recovery mechanism, but it's a corrective control rather than a preventive one. My goal is to avoid accidental deletions through least-privilege IAM policies, temporary credentials, and approval processes wherever appropriate. Although Versioning allows us to recover deleted objects, recovery still consumes engineering time, introduces the possibility of additional mistakes, and may impact customers while the system is being restored. I'd rather prevent an incident than rely on recovering from it afterward.
	- An additional good point to make is that, even if recovery is relatively fast, there is still **customer impact** during the recovery process. Preventing customer impact should always be a priority.
5. Your company allows customers to upload videos to an S3 bucket. After each upload, the system needs to:
	- Generate multiple video resolutions (480p, 720p, 1080p)
	- Extract metadata
	- Generate thumbnails
	- Run a content moderation service
	- A junior engineer proposes this design:
		```
		User
		
		↓
		
		Application
		
		↓
		
		Upload to S3
		
		↓
		
		Application immediately calls:
		    - Thumbnail Service
		    - Metadata Service
		    - Video Encoding Service
		    - Moderation Service
		
		↓
		
		Return response to user
		```
		- They argue: "This is simpler because everything happens in one place." Would you keep this design?
		- If not:
			- What would you change?
			- Why?
			- What AWS services or architectural patterns would you introduce?
> 	I would recommend an event-driven architecture that triggers the backend services once the video upload completes successfully. Using this approach allows for loose coupling between the application and the backend service. The application only needs to worry about uploading the video, while the backend services worry about their respective duties. Tightly coupling the application with the backend services introduces the possibility of errors, such as calling the services before an upload is complete or after an upload fails. To implement this architecture, I would recommend using S3 Event Notifications to trigger a compute service, such as Lambda, to perform the backend operations. This could potentially allow the services to be called asynchronously, reducing response time.
	- I'm concerned about one thing you said. You suggested triggering **a Lambda** to perform the backend operations. But video transcoding can take several minutes, and we have four different pieces of work:
		- Thumbnail generation
		- Metadata extraction
		- Video encoding
		- Content moderation
	- Would you still have one Lambda do everything? Why or why not?
> 	I wouldn't use a single Lambda for all four tasks because each task has different execution characteristics, scaling requirements, and failure modes. Instead, I'd use an event-driven architecture where a successful S3 upload initiates a workflow. That workflow would orchestrate separate compute tasks for thumbnail generation, metadata extraction, video encoding, and content moderation. This improves separation of responsibilities, allows each component to scale independently, isolates failures, and makes it much easier to extend the pipeline with additional processing steps in the future.
	- This answer is good because it doesn't mention specific AWS technologies. It only talks generally about how the architecture would be implemented.
1. Your team has a Glue job that normally finishes in **15 minutes**. This morning it took **75 minutes**. Nothing was intentionally changed. The on-call engineer asks you to investigate.
	- Here's what you do know:
		- The amount of data processed is approximately the same as yesterday.
		- The Glue cluster size hasn't changed.
		- The data is still stored as Parquet.
		- There are no obvious infrastructure alarms.
	- You have access to:
		- CloudWatch metrics
		- CloudWatch logs
		- The S3 bucket
		- Glue job metrics
	- How would you systematically narrow down the possibilities?
> 	If the Glue job has slowed significantly, but the data volume being processed and the cluster size haven't changed, I'd check executor metrics to see if all executors are busy, or if one or a few are busy. If only one or a few executors are busy, I would suspect a hot partition as the root cause and explore solutions such as repartitioning or salting. If all executors were busy, I would suspect the data volume is being spread over an increased number of parquet files. To investigate, I would look at the number of tasks assigned to each executor. If there is a significant increase, with no change in data volume or cluster configuration, it would indicate a small files problem and I would explore solutions such as file compaction.
	- I checked the executor metrics:
		- Executor utilization is balanced.
		- Task count hasn't increased.
		- Average Parquet file size hasn't changed.
	- None of your hypotheses seem to be true. What would you investigate next?
> 	I would investigate whether the underlying transformation logic still aligned with the partitioning strategy used to store raw data in S3. If there is a disconnect, it could cause a significant increase in the amount of data processed by each task.
	- Other avenues of investigation:
		- Was a `JOIN` or `GROUP BY` added to the transformation logic? This could cause increased shuffling.
		- Did the schema change? Was there a significant increase in the number of columns?
		- Did partition pruning stop working due to modified query patterns?
		- Did the Glue job configuration change? Check:
			- Number of workers
			- Worker type
			- Spark configuration
			- Shuffle partitions
		- Did data skew increase? Even if executor utilization initially appears balanced, later stages (joins or aggregations) can still experience skew.
2. Your company is building a new analytics platform. Every day:
	- 500 GB of event data is written to S3.
	- Analysts primarily query the **last 90 days** of data.
	- Data older than **7 years** must be retained for compliance.
	- The company wants to minimize storage costs without affecting analyst productivity.
	- Design a storage lifecycle strategy for this data.
	- Walk me through:
		- Which storage classes you would use.
		- When you would transition between them.
		- Whether you would ever delete data.
		- How you would automate the process.
> 	I would design a lifecycle policy that allows event data for the past 90 days to be placed in S3 Standard. Next, I would move data older than 90 days, but less than 1 year old to be moved to a S3 Standard-IA. This would allow data to be stored more cheaply than S3 Standard, but still retrieved quickly for a small cost. This would be ideal for scenarios where analysts need to reprocess older historical data. Next, I would move data 1 to 7 years old in to Glacier Flexible Retrieval. This would allow data to be stored more Cheaply than Standard-IA, but would take longer to retrieve than Standard-IA. Finally, I would store data 7 years old or more in Glacier Deep Archive, since it's almost never retrieved and only retained for compliance purposes.
	- You moved data older than 90 days into Standard-IA. Suppose one of the analysts says: "About once every quarter, we rerun a machine learning model that trains on the last two years of data." That means every three months we're suddenly reading hundreds of terabytes from Standard-IA and Glacier. Would you change your lifecycle policy? Why or why not?
> 	The new requirement changes the access pattern significantly, so I'd revisit the lifecycle policy. If two years of data is accessed every quarter, Glacier may no longer be the best fit because retrieval costs and delays could outweigh the storage savings. I'd perform a cost analysis comparing Glacier retrieval costs against simply retaining two years of data in Standard-IA or even Standard if the access frequency justifies it. If the workload is highly predictable, I would also explore automating transitions before scheduled training jobs, but only if the operational complexity is justified by meaningful cost savings.
3. Your company ingests approximately **2 TB of application logs every day**. The current pipeline looks like this:
	```
	Application
	      │
	      ▼
	S3
	      │
	      ▼
	Glue
	      │
	      ▼
	Analytics
	```
	- Everything works well initially. Six months later, analysts begin complaining that reports are taking much longer to run.
	- You investigate and discover:
		- Data is stored as **Parquet**.
		- Partitioning is still by **year/month/day**.
		- Average file size is healthy.
		- The amount of data processed each day hasn't changed.
		- However, the analysts have changed the way they query the data.
			- Originally, they ran queries like: `WHERE event_date BETWEEN ...`
			- Now, most of their queries look like: `WHERE application_name = 'CheckoutService'`
		- What do you think is happening? Would you recommend changing the S3 layout, the query pattern, or something else?
> 	Query performance is no longer benefiting from partition pruning. Because analysts are starting to query by application_name instead of event_date, significantly more data needs to be scanned with each query. Queries are no longer benefiting from partition pruning. First, I would question why the change in query pattern occurred and if it's necessary. If the change is necessary because a majority of analysts are querying by application_name, I'd look into partitioning by application_name instead, carefully designing the partition key to evenly distribute data and avoid hot partitions.
	- Note: Multi-level partitioning (partitioning by `event_date` and `application_name`) is another possibility if it creates reasonably-sized partitions.
	- I'm a little worried about changing the partitioning strategy. We have **500 different applications**, but analysts still occasionally run reports by date. If we partition by `application_name`, haven't we just made those reports slower?
> 	The problem isn't necessarily the number of partitions, but how much data is in each partition. If there is a reasonable amount of data associated with each application, then 500 different partitions would be appropriate. If partitioning by application_name creates 500 tiny partitions, then repartitioning may not be worth it if the metadata overhead outweighs the improved query performance.
	- Again, repartitioning only makes sense if a majority of queries have shifted toward `application_name`. There needs to be a **significant** change in query patterns to justify a change in partition strategy.

# DynamoDB

## Overview

### Design Scenario
- Imagine you're building the backend for a large e-commerce website.
- The application stores:
	- Customer profiles
	- Shopping carts
	- Product inventory
	- Orders
- Traffic is fairly normal most of the day: 5,000 TPS
- Black Friday begins. Suddenly, traffic jumps to: 500,000 TPS
- The system must continue serving requests with **single-digit millisecond latency**.
- The CTO says: "I don't want customers waiting for the database to scale. It needs to absorb traffic spikes automatically."
- What characteristics would your ideal database have?
> 	I would design the database to be highly scalable so that it could handle high intensity traffic spikes without any degradation in performance. I would also make the database highly available by replicating database instances across multiple availability zone, so that the service could continue operating in the event of regional outages. As the business grows and changes over time, I would design the schema to be flexible and evolve with the the business, while also restricting changes that break downstream consumers. The database would be serverless and automatically scale with varying loads.
	- This is one of DynamoDB's defining characteristics. The service is capable of scaling **while maintaining predictably low latency**.
	- DynamoDB is also serverless, allowing developers to focus on the application **instead of database administration**.
	- NoSQL databases generally prioritize schema flexibility, which is valuable when different items may have different attributes or when the application evolves over time.
	- Handling massive traffic spikes in a traditional PostgreSQL database would typically involve the following tasks:
		- Provision larger instances in advance.
		- Add read replicas.
		- Configure load balancing.
		- Ensure replication has caught up.
		- Verify storage and network capacity.
- DynamoDB was primarily built so application developers don't have to solve distributed database problems themselves.
	- Instead of worrying about:
		- Partitioning data across machines
		- Replication
		- Failover
		- Capacity planning
		- Scaling
	- DynamoDB handles those concerns behind the scenes.
	- **This convenience does come with certain tradeoffs**.

### Knowledge Check
- Imagine you have two applications:
	- Application A: A Spark job scans **500 million records** every night to generate reports.
	- Application B: A shopping cart service retrieves **one customer's cart** in **single-digit milliseconds** whenever they click "View Cart."
- Which application is a better fit for DynamoDB and why?
> 	DynamoDB would likely be a better fit for Application B because the workload involves quickly pulling indivual details regarding one customer's cart. Application A performs nightly batch operations on hundreds of millions of records. DynamoDB is better optimized for transactional workloads like the one related to Application B.
	- Application A involves a **batch analytics** workload. This is much more naturally solved with:
		- S3
		- Parquet
		- Glue
		- Spark
		- Athena
	- Application B involves an **operational workload**. Perfect for DynamoDB.

## Partition Keys

### Design Scenario
- Suppose you're designing a shopping cart service.
- Each shopping cart looks like:
	```
	CustomerID
	Items
	LastUpdated
	TotalPrice
	```
- There are:
	- 100 million customers
	- 500,000 TPS during Black Friday.
- You have 100 database servers.
- Which server stores each customer's shopping cart?
- How would you distribute the data across the 100 servers?
> 	I would distribute data evenly across the 100 servers using a partition key based on a customer attribute. The attribute would need to be defined for all customers. It would need to be general enough to create at least 100 unique partitions, but specific enough to not create an excessive amount of partitions. The unique values for the attribute should also be fairly evenly distributed among the customer base to avoid creating a hot partition.
	- Unlike Spark or S3, a high-cardinality partition key is actually **exactly what you want**. This is because DynamoDB doesn't create one physical server per unique partition key. Instead, it **hashes** the partition key.
	- This allows it to avoid the metadata and scheduling overhead associated with high-cardinality keys in Spark or S3.
	- The hash function DynamoDB uses is designed to spread data evenly across the available physical partitions. Having **millions of unique customer IDs** is actually a benefit because it gives the hash function lots of values to distribute.

### Choosing a Good Partition Key
- You don't choose a partition key to create enough partitions to exactly match the number of available servers.
- You choose a partition key that has:
	- High cardinality
	- Even distribution
	- Stable values
	- Good access patterns
- DynamoDB figures out how to map those values onto physical storage.

### Knowledge Check
- Suppose you're designing a shopping cart table.
- Which partition key would you choose?
	- Option A: Country
	- Option B: Customer ID
> 	I would choose CustomerID because it has a higher cardinality than Country. Unlike Spark or S3, which use a key's value directly to create a partition, DynamoDB assings key values to physical partitions based on a key's hashed value. A good partition key has high cardinality _and_ produces an even distribution of traffic.
	- When using S3 as a storage layer, you ask: "How should we partition the data?"
	- When using DynamoDB as a storage layer, you ask: "What key should I hash?"
	- Remember, a partition key distributes **data**, not **traffic**. Hot partitions are still possible. You want to choose a high-cardinality key that will **also evenly distribute traffic**.

## Hot Partitions

### Design Scenario
- You're designing a DynamoDB table for an online retailer.
- Each item looks like:
	```
	ProductID
	Inventory
	Price
	Description
	```
- You choose `ProductID` as the partition key. At first, everything works perfectly. The table easily handles millions of products.
- Then, A famous YouTuber recommends one product.
- For the next hour:
	```
	Nintendo Switch
	
	↓
	
	300,000 requests/second
	```
- Every other product receives only a handful of requests.
- You chose a **high-cardinality** partition key. Why is the database suddenly struggling?
- Think about:
	- What happens after DynamoDB hashes the partition key?
	- Which physical partition receives those requests?
	- Why doesn't having millions of other partition keys help?
> 	The database is struggling because one partition is responsible for handling the 300,000 TPS load. Although the DynamoDB hashing function evenly distributes data among partitions, it does not guarantee traffic will be distributed evenly. Even though DynamoDB automatically manages capacity, a single hot partition key can become a bottleneck because all requests for that key are directed to the same partition. Having more cardinality in the ProductID won't help because the Nintendo Switch only has one ProductID, so it will hash to the same partition.
	- Partitions don't receive a set amount of compute resources when they're created. DynamoDB can automatically split and rebalance partitions over time, and in on-demand mode it does a lot of work behind the scenes to absorb changing traffic.
	- However, there are still limits to how much throughput a **single partition key** can sustain because all requests for that key ultimately target the same logical item or partition.
	- How would you fix the hot partition?
> 	Salting the partition key with a random suffix would allow traffic for a hot key to be distributed among multiple partitions. Using a composite key could also be effective, so long as the other component of the key doesn't create another hot partition.
	- How do you read the inventory using the salted key?
		- There are several approaches.
		- For example:
			- The application knows how many shards exist and queries all of them.
			- A routing layer maps logical keys to salted keys.
			- The data model is redesigned to reduce contention.
		- The important thing isn't memorizing one solution. It's recognizing that **there is no free lunch**. Every design introduces tradeoffs.
- **Spark**:
	- More partitions
	- Better parallelism
	- More scheduling overhead
- **S3**:
	- More partitions
	- Better parallelism
	- More metadata overhead
- **DynamoDB**:
	- Salted keys
	- Better write scalability
	- More complex reads

### Knowledge Check
- Suppose you have two candidate partition keys for a shopping cart table:
	- Option A: `CustomerID`
	- Option B: `LastUpdatedDate`
- Assume customers constantly update their carts throughout the day.
- Which key would you choose, and why?
> 	I would choose `CustomerID` because the primary access pattern is retrieving a specific customer's shopping cart. In DynamoDB, the partition key should first support the application's access patterns. `CustomerID` also has high cardinality, so it generally distributes data well. Although timestamps have excellent cardinality, they don't align with how the application retrieves data, making reads inefficient.
	- Choose the partition key based on how the application accesses the data first. Then worry about distribution.
	- Non-relational databases like DynamoDB need to be designed around **workflow**, not data.
	- While `LastUpdatedDate` has high cardinality and reduces the risk of a hot partition, **it does not match common access patterns**.

## Sort Keys

- Until now, we've assumed every DynamoDB table looks like:
	```
	Partition Key
	
	↓
	
	Item
	```
- But DynamoDB also allows:
	```
	Partition Key + Sort Key
	
	↓
	
	Multiple related items
	```

### Design Scenario
- Suppose you're building an order history service.
- A customer can place:
	- 1 order
	- 10 orders
	- 10,000 orders
- The application needs to support:
	1. Retrieve a specific order.
	2. Show the customer's most recent orders.
	3. Show all orders from last month.
- You already know the partition key should probably be: `CustomerID` because that's how the application identifies whose orders to retrieve.
- How would you distinguish one order from another?
> 	I would introduce order ID since it uniquely identifies an individual order. Using the order's timestamp wouldn't work since orders from different customers can be placed at the exact same time.
	- Option 1: Order ID
		```
		Partition Key: CustomerID
		Sort Key:
		1001
		1002
		1003
		1004
		```
		- Works great for finding a specific order.
		- Doesn't work for finding the most recent orders.
		- The sort order depends entirely on how `OrderID` is generated.If it's just a UUID or otherwise not time-based, it tells us nothing about recency.
	- Option 2: Order Timestamp
		```
		Partition Key: CustomerID
		Sort Key:
		2026-08-01
		2026-08-03
		2026-08-04
		2026-08-06
		```
		- Works great for retrieving the latest orders and orders from last month.
		- Retrieving a specific order is still possible, assuming you know the timestamp or you store the `OrderID` as another attribute and query appropriately.
- **Partition key is chosen based on the access pattern. Sort key is chosen based on how items within a customer's partition should be sorted**.
- In DynamoDB, the primary key is a combination of the partition key and sort key:
	- The **partition key** determines _which customer's collection_ you're in.
	- The **sort key** organizes items _within that customer's collection_.
	- Primary Key Example: `(CustomerID, SortKey)`.

## Access Pattern Modeling

- When designing tables in DynamoDB:
	- You don't start by designing the schema.
	- You start by listing every question your application needs to answer.
	- Then you design the table to answer those questions efficiently.

### Design Scenario
- You're building the backend for an online bookstore.
- Each order contains:
	```
	OrderID
	CustomerID
	OrderDate
	Books
	TotalPrice
	Status
	```
- The product manager tells you the application only needs to support these operations:
	- Retrieve a specific customer's order history.
	- Retrieve the customer's **10 most recent orders**.
	- Retrieve a specific order when the customer views it.
	- Add a new order.
- **That's it. No analytics. No reporting**.
- How would you model the table? Specifically
	- What would you choose as the **partition key**?
	- What would you choose as the **sort key**?
	- Why?
> 	I would choose CustomerID as the partition key because this is how the application would primarily filter orders. I would chose OrderDate as the sort key because orders are primarily being ordered chronologically.
	- The only requirement this design doesn't fulfill is retrieving a specific order when a customer views it. This does not mean the design is wrong.
	- There are several ways to solve this in DynamoDB:
		- Option 1:
			```
			Partition Key = CustomerID
			Sort Key = OrderID
			```
			- Great for direct lookups.
			- Less convenient for chronological queries.
		- Option 2:
			```
			Partition Key = CustomerID
			Sort Key = OrderTimestamp
			```
			- Great for history.
			- May need another index for direct `OrderID` lookups.
		- Option 3:
			```
			Partition Key = CustomerID
			Sort Key = OrderTimestamp#OrderID
			```
			- Use a composite sort key.
			- Now the sort key preserves chronological ordering while also ensuring uniqueness.
		- Option 4:
			- Keep your primary key exactly as you proposed (Option 2) and later add a **Global Secondary Index (GSI)** on `OrderID`.
		- A primary key needs to be designed so that it best supports the **most important** access patterns, then use additional techniques for the rest.
- Suppose the product manager says: "Actually, customer support needs to look up an order by `OrderID` alone. They don't know the `CustomerID`."
- Would you:
	- Change your primary key?
	- Add another table?
	- Add an index?
	- Something else?
> 	If customer support no longer has access to the CustomerID and only knows the OrderID, keeping CustomerID as the partition key would no longer match data access patterns. The partition key should be changed to OrderID and the sort key should be changed to OrderTimestamp if chronological filtering is still required.
	- This is not an ideal solution because optimized for **one new access pattern** at the expense of **three existing ones**.
	- The **original application hasn't changed**. We've just added another consumer.
	- An ideal solution would be:
		- Partition Key = `CustomerID`
		- GSI Partition Key = `OrderID`
		- Sort Key = `OrderTimestamp`
	- Don't redesign your primary key every time a new query appears.
	- Ask yourself:
		- Is this a new primary workload?
		- Is this an additional access pattern?
	- If it's the latter, a secondary index is often the better solution.
	- Polished Answer:
> 	My first instinct would be to keep the existing primary key because it still optimizes the application's core workflows. The new customer support requirement introduces an additional access pattern rather than replacing the original ones. Instead of redesigning the primary table, I'd look for a way to support that new lookup independently, such as a secondary index keyed by `OrderID`. That preserves the performance of the existing application while efficiently supporting the new requirement.

## Global Secondary Indexes (GSIs)

### Design Scenario
- Imagine you had a table with the following primary key:
	```
	Partition Key: CustomerID
	Sort Key: OrderTimestamp
	```
	- It supports:
		- Customer order history
		- Latest orders
		- Add new order
- Then customer support arrives. They need:
	```
	OrderID
	
	↓
	
	Find order
	```
	- They **don't** have access to the `CustomerID` field.
- Imagine DynamoDB didn't have GSIs. How would you solve the problem?
- Possibilities could include:
	- Change the primary key.
	- Create another table.
	- Duplicate some data.
	- Maintain a lookup table.
	- Something else.
> 	I would only consider changing the primary key if finding orders by OrderID became the primary access pattern. Otherwise, I would consider creating a table with the same data, but use a primary key that supported the customer support use case. Creating a lookup table could also work and would involve duplicating less data, but would involve customer support needing to query two tables instead of just one.
	- A GSI is conceptually very similar to maintaining another representation of the same data with a different primary key.
	- AWS automates the synchronization for you.

### Overview
- A Global Secondary Index is essentially AWS saying: "We'll maintain that second representation of the data for you."
- Imagine the primary table:
	```
	Primary Table
	
	CustomerID
	↓
	
	OrderTimestamp
	
	↓
	
	Order Data
	```
- Now imagine a second, automatically maintained index:
	```
	GSI
	
	OrderID
	↓
	
	Order Data
	```
- Now the application can answer either question efficiently.
	- Customer Application:
		```
		CustomerID
		
		↓
		
		Primary Table
		```
	- Customer Support:
		```
		OrderID
		
		↓
		
		GSI
		```
	- Neither application needs to compromise.
- Don't think of a GSI as another index. Instead, think of it as, "another door to the same data."
	- It would be like a library having an one catalog organized by `Author` and another catalog organized by `Title`.
	- The books didn't move. You just created another way to find them.
	- That's almost exactly what a GSI does.
- What's the downside of creating lots of GSIs?
> 	Maintaining an excessive number of GSIs for one table would hinder write performance because every record would need to be written to each GSI in order to maintain consistency.
	- **S3**:
		- More partitions
		- Faster scans
		- More metadata
	- **Spark**:
		- More partitions
		- Better parallelism
		- More scheduling overhead
	- **DynamoDB**:
		- More GSIs
		- More query patterns
		- More write overhead
		- More storage
			- Higher cost
		- More complexity
			- Which queries use which index?
			- Does this GSI still provide value?
			- Is it worth its write cost?
		- **Experienced engineers don't casually create new GSIs**.
- Why not just create a GSI for every field?
> 	Because every GSI introduces additional write overhead, storage costs, and operational complexity. I prefer to add GSIs only when they support a meaningful access pattern that justifies those tradeoffs.

### Knowledge Check
- Suppose your primary table is:
	```
	Partition Key: CustomerID
	Sort Key: OrderTimestamp
	```
- You create a GSI with:
	```
	Partition Key: OrderStatus
	Sort Key: OrderTimestamp
	```
- Now, the application can efficiently answer questions such as: "Show me all shipped orders."
- Would you create this GSI if only **one internal report**, run once per month, needed that query? Why or why not?
> 	I probably wouldn't create this GSI because it optimizes a query that's only run once per month while introducing continuous write overhead, additional storage costs, and operational complexity. I'd first ask whether that report could instead use a batch analytics solution such as S3, Glue, or Athena. If I did consider a GSI, I'd also evaluate whether `OrderStatus` provides sufficient traffic distribution for the expected workload, since a low-cardinality partition key could create hotspots under frequent access.
	- The fact that `OrderStatus` is a low-cardinality key doesn't automatically invalidate it as a GSI. It depends on the workload.
	- For example, suppose the monthly report ran once a month, overnight, during low traffic. A low-cardinality GSI might still be acceptable.
	- On the other had, if the application is serving 50,000 TPS for an `OrderStatus` of `Shipped`, then a hot partition becomes a real concern.
	- S3, Glue, and Athena are proposed as alternatives because they're optimized for **analytical** workloads. DynamoDB is optimized for **transactional** workloads.
	- Remember:
		- High cardinality can be helpful.
		- Even **traffic distribution** is the ultimate goal.

## Consistency

### Design Scenario
- You're building a banking application.
- A customer transfers **$500** from their checking account to their savings account. The transfer completes successfully.
	- The application reads:
		```
		Checking:  $4,500
		
		Savings:   $2,000
		```
	- A few seconds later they refresh again. Now they see:
		```
		Checking:  $4,500
		
		Savings:   $2,500
		```
		- The transfer actually succeeded immediately. The second read was simply stale.
- **Is this acceptable? Or should the application always return the most up-to-date data? Why**?
- Now imagine a completely different application.
	- A social media app shows: `Likes: 12,451`.
	- A user presses Like. The page briefly still shows: `Likes: 12,451`.
	- One second later, it shows: `Likes: 12,452`.
- **Would your answer be different for this application?Why**?
> 	For scenarios involving financial transactions, strong consistency is generally preferred because it ensures consistent reads, which is very important when working with financial data. However, for scenarios involving posts on social media, eventual consistency is acceptable because slight delays in the propagation of data changes to table replicas doesn't have a significant impact on user experience. The major tradeoff between strong and eventual consistency is latency. For the banking application, strong consistency may introduce additional read latency because the system must ensure the latest committed data is returned before responding. This is considered acceptable because waiting for accurate information is better than receiving inaccurate information immediately.
- Why doesn't everyone just use strong consistency all the time?
> 	Because many applications don't need it. If slightly stale data doesn't affect the user experience or business correctness, eventual consistency provides better scalability and lower latency. I reserve strong consistency for workloads where returning stale data could lead to incorrect business decisions, such as financial transactions or inventory management.
	- Match the consistency model to the business requirement.

### Knowledge Check
- Tell me whether you'd choose **Strong** or **Eventual** consistency, and explain why.
- Scenario 1:
	- An e-commerce website displays:
		```
		Remaining inventory:
		3 units
		```
	- Thousands of customers are trying to purchase the item at the same time.
	- **Would you prefer string or eventual consistency**?
- Scenario 2:
	- A weather application refreshes the current temperature every minute.
	- **Would you prefer string or eventual consistency**?
- Scenario 3:
	- A dashboard displays yesterday's total sales revenue for executives.
	- The dashboard refreshes every hour.
	- **Would you prefer string or eventual consistency**?
> 	For scenario 1, I would prefer strong consistency to ensure all customers see the same inventory count, so customers don't try to buy an item that is out of stock and receive an error message when they click the 'Buy' or 'Add to. Cart' button. For scenario 2, I would prefer eventual consistency. Weather information typically isn't used to make important decisions. Furthermore, the information is updated once every minute. Weather information isn't highly volatile, so seeing 1 or 2 minute old weather data would still be fairly accurate. For scenario 3, I would prefer strong consistency because total sales revenue can be highly volatile, especially during busy shopping periods. Furthermore, the information feeds a dashboard that is used to make important executive decisions.
	- Scenario 3 would actually use **eventual consistency**, but the reasoning behind strong consistency is sold.
	- Eventual consistency is used because:
		- The refresh interval is long.
		- The data represents historical aggregates.
		- The cost of a slightly stale read is very low.
	- Many executive dashboards are powered by:
		- Data warehouses
		- Batch ETL jobs
		- Eventually consistent data
	- This is because **the emphasis is on trends** rather than real-time operational decisions.
	- If the dashboard showed live sales this minute or current inventory during Black Friday, strong consistency would be more appropriate.

| Workload            | Important? | Needs Strong Consistency? |
| ------------------- | ---------- | ------------------------- |
| Bank transfer       | Yes        | Yes                       |
| Shopping cart       | Yes        | Often yes                 |
| Executive dashboard | Yes        | Maybe not                 |
| Weather app         | No         | No                        |
| Social media likes  | No         | No                        |

## Transactions

### Design Scenario
- You're building an online store.
- A customer buys the **last Nintendo Switch** in stock. The purchase involves two operations:
	1. Decrease inventory
	2. Create the customer's order
- Now imagine something goes wrong. The system crashes **after** Step 1 but **before** Step 2.
- The result is:
	```
	Inventory = 0
	
	Order = Missing
	```
	- The customer was charged. The inventory disappeared. But no order exists.
- **How would you design the system to prevent this**?
- Now imagine the opposite. The order is created, but the inventory update fails. Now you have:
	```
	Inventory = 1
	
	Order = Exists
	```
	- Two customers might now purchase the last item.
- What property would you want from the database so that **either both operations succeed or neither does**?
> 	I would want the transaction to be atomic, so that it either completely succeeds or fails. If the operation is allowed to partially succeed, it could result in poor user experience and increased operational overhead due to required manual intervention.
- Imagine a product manager says: "Every order should use a transaction because transactions are safer." Would you agree?
> 	Ideally, you would want to confirmation of an order to use a transaction so that it completely succeeds or fails. However, you might not want to make adding an item to a shopping cart a transaction since a customer could accidentally add an item to their cart or intentionally add an item, but never follow through with the purchase.
	- Transactions shouldn't be used **everywhere** because every operation doesn't necessarily need a guarantee.
	- Adding an item to a cart shouldn't be a transaction because **it doesn't involve multiple writes that must succeed together**, not because a customer might not follow through with the purchase. Adding an item to a cart typically only requires one write.
	- Checkout should be a transaction because it involves coordinating several operations:
		```
		Decrease inventory.
		
		Create order.
		
		Charge payment.
		
		Update shipment queue.
		```
		- Now there are multiple pieces of state that need to stay consistent.
		- That's where a transaction becomes valuable.
- When should you use a transaction?
> 	When multiple related writes must either all succeed or all fail to preserve business correctness. I avoid using transactions for simple, independent operations because they introduce additional coordination and overhead without providing meaningful benefit.

### Knowledge Check
- When would a transaction be appropriate:
	- Scenario 1:
		- A customer updates their email address.
		- Onely one item changes.
	- Scenario 2:
		- A customer transfers money between two accounts.
		- Two balances change.
	- Scenario 3:
		- A customer **submits** an order.
		- The system:
			- Creates the order.
			- Decreases inventory.
			- Updates the customer's loyalty points.
> 	Scenario 1 should not be a transaction because it involves a single write. Scenarios 2 and 3 should use transactions because they involve multiple, interdependent writes.

- Using transactions can be simplified to one decision tree:
	```
	Does this operation modify multiple pieces of related state?
	
	        │
	       No
	        │
	        ▼
	  Don't use a transaction.
	
	        │
	       Yes
	        │
	        ▼
	Would partial success leave the system in an incorrect business state?
	
	        │
	       No
	        │
	        ▼
	Probably don't use a transaction.
	
	        │
	       Yes
	        │
	        ▼
	Use a transaction.
	```
	- Notice the emphasis on **partial success leaving the system in an incorrect business state**. For example, suppose you had the following operation:
		```
		Create Order
		
		↓
		
		Send Confirmation Email
		```
	- If the confirmation email fails to be sent, the order should still be created. The email isn't part of the critical business state.
	- **Separate critical operations from auxiliary ones**.
- Would you include sending an SNS notification inside a transaction?
> 	No. The notification is a side effect, not part of the core business state. I'd commit the transaction first, then publish the notification asynchronously. If the notification fails, I can retry it without affecting the correctness of the order.

## Capacity Modes & Throughput

### Design Scenario
- Suppose you're building two completely different applications.
- Application A: A payroll system.
	- Every Friday at 9:00 AM:
		```
		10 requests/second
		
		↓
		
		100,000 requests/second
		
		↓
		
		Back to 10 requests/second
		```
		- The traffic spike lasts about 30 minutes.
		- The rest of the week, traffic is minimal.
- Application B: A internal HR system.
	- Traffic looks like this every day:
		```
		4,800 requests/second
		
		↓
		
		5,200 requests/second
		
		↓
		
		4,900 requests/second
		
		↓
		
		5,100 requests/second
		```
		- Very stable. Very predictable.
- Suppose you had to provision the infrastructure yourself. Which application would be easier to plan capacity for? Why?
- Now think about the payroll system.If you had to manually provision capacity for it, what risks would you worry about?
	- Consider things like:
		- Under-provisioning
		- Over-provisioning
		- Cost
		- User experience
> 	It would be easier to plan capacity for Application B because the traffic pattern is stable and predictable. Auto-scaling could handle the daily peaks in traffic. Application A would be more difficult because of how massive the spike is compared to Application B. Auto-scaling likely wouldn't be able to react quickly enough to handle it, so you'd need to provision capacity ahead of time and make sure it's ready to handle the spike when it occurs. If the capacity is provisioned too close to the occurrence of the spike, it might not be ready to handle it in time, so you'd need to early enough to handle the spike, but not too early so that you don't waste resources on idle compute.

### Overview
- **Provisioned Capacity**:
	- Think of it as saying: "I know roughly how much traffic I'm going to receive."
	- You tell DynamoDB: "Reserve this much capacity."
	- Pros:
		- Lower cost for predictable workloads.
		- More control over capacity planning.
	- Cons:
		- You have to estimate correctly.
	- More appropriate for **Application B**:
		- Easy to estimate.
		- Lower ongoing cost.
		- Very little operational risk.
- **On-Demand Capacity**:
	- Think of it as saying: "I don't know what traffic will look like."
	- DynamoDB automatically adjusts capacity as traffic changes.
	- Pros:
		- Excellent for unpredictable workloads.
		- Minimal operational effort.
	- Cons:
		- Generally higher cost per request.
		- Less opportunity to optimize for stable traffic.
	- More appropriate for **Application A**:
		- The operational simplicity often outweighs the higher per-request cost, especially if those spikes are difficult to predict or vary over time.
		- If the spike is **highly predictable** (for example, every Friday at exactly 9:00 AM), Provisioned Capacity **with scheduled scaling** could also be a very reasonable choice.
- When would you choose On-Demand over Provisioned?
> 	I choose based on workload predictability. For stable, predictable traffic, Provisioned Capacity is usually more cost-effective because I can accurately estimate the required throughput. For unpredictable or rapidly changing workloads, On-Demand reduces operational overhead by automatically scaling capacity without requiring me to forecast demand.

### Knowledge Check
- Scenario 1:
	- A startup launches a brand-new mobile app.
	- They have no idea how much traffic they'll receive on launch day. 
	- It could be:
		- 100 users
		- 100,000 users
	- Provisioned or On-Demand? Why?
- Scenario 2:
	- A mature banking application processes:
		```
		8,000 requests/second
		
		±5%
		```
		- This occurs regularly on a daily basis.
	- Provisioned or On-Demand? Why?
- Scenario 3:
	- A ticketing website sells concert tickets. Traffic is usually low.
	- The moment tickets go on sale:
		```
		500 requests/sec
		
		↓
		
		250,000 requests/sec
		```
		- This occurs in under one minute.
	- Provisioned or On-Demand? Why?
> 	For Scenario 1, I would use On-Demand capacity because the workload is unpredictable. For Scenario 2, I would choose Provisioned capacity because the workload is stable and predictable. I would use scheduled scaling to handle the 5% increase in workload if it occurred regularly and predictably. For Scenario 3, I would use On-Demand capacity because the workload is unstable. Provisioned Capacity is also an option, but would require careful planning and more operational overhead.
- Which capacity mode is better?
> 	Neither is universally better. Provisioned Capacity is generally more cost-effective for stable, predictable workloads where I can estimate throughput accurately. On-Demand Capacity is better for unpredictable or spiky workloads because it minimizes operational overhead and reduces the risk of under-provisioning.

## Time to Live (TTL)

### Design Scenario
- You're building an e-commerce platform.
- Every customer has a shopping cart.
- If a customer abandons their cart, the business wants to keep it for 30 days.
- After that:
	- The customer almost never returns.
	- The cart serves no business purpose.
	- Keeping millions of abandoned carts increases storage costs.
- How would you design the system so abandoned shopping carts are automatically removed after 30 days?
- Suppose instead you decide: "We'll just have an engineer run a cleanup script once a month." Would that be your preferred solution? Why or Why not?
> 	I would design the system so that shopping carts are automatically removed if they aren't updated for 30 consecutive days. This ensures a customer's cart is maintained even if they aren't modifying their cart on a daily or weekly basis, but can still be considered active on the platform. I would not prefer a manual monthly cleanup script because it is error-prone and adds unnecessary operational overhead. An active customer's cart could be accidentally deleted or a dormant customer's cart could accidentally be retained.
	- Although a well-tested cleanup script _could_ implement the same "30 days since last update" logic, it still has drawbacks:
		- Someone has to schedule and maintain it.
		- It becomes another operational process to own.
		- It's easier to forget, misconfigure, or accidentally disable.
	- TTL moves this responsibility to the database.

### Overview
- In DynamoDB, TTL deletion is **best effort**.
- This means:
	- The item becomes eligible for deletion at the expiration time.
	- DynamoDB removes it automatically, but **not necessarily immediately**.
- This is why TTL is great for things such as:
	- Shopping carts
	- Session tokens
	- Temporary caches
	- Verification codes
- It's **not** appropriate if your business requires deletion at an exact second.

### Knowledge Check
- Scenario 1:
	- Password reset tokens expire after 15 minutes.
- Scenario 2:
	- Employee records.
	- The company must retain them permanently.
- Scenario 3:
	- A fraud detection system stores temporary duplicate-request IDs for: 24 hours.
	- The IDs prevent the same payment request from being processed twice.
- Where is TTL a good fit?
> 	I would use TTL for scenarios 1 and 3 because the deletion policies for these records is stable and can easily be automated. I would not use TTL for employee records that may need to be retained for legal or compliance purposes, as this could lead to accidental deletion. TTL is ideal for cleaning up expired tokens, but my application should still check the expiration timestamp before accepting a token
	- Remember, TTL is **best effort**. The application should be configured to prevent the use of expired tokens. The application should rely on TTL alone to determine if a token is valid.
	- Instead:
		- The application independently enforces expiration.
		- TTL **eventually** cleans up the stale data.
- Why not just have a scheduled Lambda delete expired items every hour?
> 	Because expiration is an intrinsic property of the data. TTL lets the database manage that lifecycle automatically, eliminating the need to build, schedule, monitor, and maintain a separate cleanup process.
	- Prefer built-in automation over custom operational workflows when the built-in feature satisfies the business requirement.

## Streams

### Design Scenario
- You're building an e-commerce application.
- Every time an order is created, several things need to happen:
	1. Send a confirmation email.
	2. Update the customer's loyalty points.
	3. Notify the shipping system.
	4. Update a sales dashboard.
- The application could do this:
	```
	Create Order
	
	↓
	
	Send Email
	
	↓
	
	Update Loyalty
	
	↓
	
	Notify Shipping
	
	↓
	
	Update Dashboard
	
	↓
	
	Return Success
	```
- Would you design it this way? Why or why not?
- Suppose instead the application does only this:
	```
	Create Order
	
	↓
	
	Database
	```
- Then, the database publishes a `Order Created` event.
- Other services subscribe:
	```
	Email Service
	
	Shipping Service
	
	Loyalty Service
	
	Analytics Service
	```
	- Each service performs its work independently.
- What advantages does this architecture provide?
- Think about:
	- Coupling
	- Reliability
	- Scalability
	- Separation of responsibilities
	- Failure isolation
> 	I would not design the service this way because it tightly couples the application with the backend services. Every time the backend services are modified, added, or removed, the application also needs to be modified. Tightly coupling the application with the backend services could also create race conditions. The alternative design makes more sense because each service can operate and scale independently. It also isolates failures to an individual service, without affecting other services.
		- The bigger concern isn't necessarily a race condition. It's that the application now becomes responsible for coordinating multiple independent services.

### Overview
- DynamoDB streams are very similar to S3 event notifications:
	- S3 Event Notification:
		```
		Upload Image
		
		↓
		
		S3 Event
		
		↓
		
		Lambda
		```
	- DynamoDB Stream Event:
		```
		Order Created
		
		↓
		
		Stream Event
		
		↓
		
		Lambda
		
		↓
		
		SNS
		
		↓
		
		Other Services
		```
- Similar to event notifications, DynamoDB records a stream event every time an item is:
	- Inserted
	- Updated
	- Deleted
- Why use DynamoDB Streams instead of calling Lambda directly after writing to the table?
> 	Streams decouple the application from downstream processing. The application only needs to write to DynamoDB. Any services interested in changes can consume the stream independently, improving maintainability, scalability, and failure isolation.

### Knowledge Check
- Imagine you're building a social media platform.
- Whenever someone creates a new post, the system needs to:
	- Update the user's timeline.
	- Notify followers.
	- Update trending hashtags.
	- Run content moderation.
	- Update analytics.
- Would you put all of that logic inside the API that creates the post?
- Or would you use an event-driven architecture similar to the one we just discussed? Why?
> 	I would not put all of the logic inside the API that creats the post because the API should only be responsible for creating the post. Once the post is created, an event can be created and consumed by downstream services that independently perform their respective actions. This allows for loose coupling, independent scaling, and failure isolation. If the actions needed to be performed in a particular order, the event could trigger an orchestrated workflow that ensures the work is done in the proper order and that know work is unnecessarily repeated if one step fails.
	- An orchestrator (like Step Functions) isn't necessarily triggered by an event. The orchestrator only cares about performing the workflow properly, the **workflow** is triggered by the event and orchestrator orchestrates the workflow.
- When would you choose choreography versus orchestration?
	- **Choreography (events):** Independent services reacting to events where execution order isn't critical.
	- **Orchestration:** Multi-step workflows with dependencies, retries, branching, compensation, or strict ordering.

## Dynamo DB Accelerator (DAX)

### Design Scenario
- You're building a product catalog. Each product page displays:
	- Product name
	- Description
	- Price
	- Images
	- Reviews
- One particular product becomes extremely popular. Suddenly:
	```
	ProductID = NintendoSwitch
	
	↓
	
	200,000 reads/second
	```
- Most of those reads return exactly the same data. The product description changes maybe once every few weeks.
- Would you send all 200,000 reads directly to DynamoDB? Or is there a better approach? Why?
- Suppose you introduce a cache. Now the product price changes.
- How would you make sure customers don't continue seeing the old price?
- When would you **not** use a cache in front of DynamoDB?
- Can you think of workloads where adding a cache might provide little benefit—or even make the system more complicated than it's worth?
> 	Instead of sending all 200,000 reads directly to DynamoDB, I would cache the first read and use the cache fulfill the other 199,999 requests. Caching avoids putting excess, unnecessary strain on the database and is faster than calling the database correctly, assuming there's a cache hit. As long as the cache hit ratio is high, most requests are served from the cache, significantly reducing load on DynamoDB. I would not use a cache in front of DynamoDB for workloads that involve rapidly changing data. To handle product price changes, you would need to implement a way to notify the cache when an update occurs, possibly using a stream. When the cache receives the event, it invalidates the entry and either updates the value ahead of time or the next time there is a cache miss for the entry.
	- The first method of updating the cache is sometimes called **write-through** or **refresh-ahead**, depending on the implementation.
	- The second method of updating the cache is often called **cache-aside** or **lazy loading**.
- Why not put everything in the cache?
> 	Because caches are typically smaller, more expensive per GB than long-term storage, and optimized for fast access rather than durability. I use a cache for frequently accessed data with a high cache hit ratio, not as the system of record.
- Why not just use a TTL of 5 minutes and ignore invalidation?
> 	TTL eventually removes stale data, but it doesn't guarantee freshness immediately after an update. If users expect to see changes quickly, I would invalidate or refresh the cache when the underlying data changes. TTL is still useful as a fallback mechanism to clean up stale entries if an invalidation event is missed.

### Overview
- **DAX is not a new database**. It's simply a cache that sits in front of DynamoDB.
- Everything you've learned about caching still applies:
	- Cache hit ratio
	- TTL
	- Cache invalidation
	- Hot keys
	- Read-heavy workloads

## Mock Interview Questions

1. You're designing the backend for an online ticketing platform.
	- Requirements:
		- Customers can browse concerts.
		- Popular concerts may receive **200,000 reads/second**.
		- Purchasing a ticket must never oversell inventory.
		- Customer support needs to search for tickets using **TicketID**.
		- Executives run sales reports once every night.
		- Customers receive confirmation emails immediately after a purchase.
		- Abandoned reservations should automatically expire after **15 minutes**.
	- Walk me through your high-level design. Explain:
		- Which DynamoDB keys you would choose.
		- Whether you'd use any GSIs.
		- Where transactions are needed.
		- Whether you'd use DAX.
		- Whether you'd use Streams.
		- Whether TTL is appropriate.
		- Whether you'd use strong or eventual consistency for different operations.
		- Any major tradeoffs you would consider.
> 	For browsing concerts, I'd optimize for read performance by using DAX because concert metadata changes infrequently but receives very high read traffic. For ticket purchases, I'd use DynamoDB transactions with strong consistency to ensure inventory is never oversold. I'd store reservations in DynamoDB with a TTL so abandoned reservations expire automatically after 15 minutes. I'd use DynamoDB Streams to trigger downstream actions such as confirmation emails and analytics updates without tightly coupling the purchase API to those services. Customer support would use a GSI on `TicketID` for direct lookups. For executive reporting, I wouldn't build another GSI because the reports only run nightly; I'd instead export the data into an analytics platform such as S3 and Athena.

# Redshift

## Overview

- Redshift is an **analytical** data warehouse designed for large-scale **OLAP** workloads.
- Think about a workload like:
	```
	S3 / operational databases
	        ↓
	    ETL / ELT
	        ↓
	    Redshift
	        ↓
	Complex analytical queries
	        ↓
	Dashboards / BI / reporting
	```
- Redshift is optimized for queries that scan and aggregate **large amounts of data**, rather than for high-volume transactional operations. For example:
	```sql
	SELECT
	    customer_id,
	    DATE_TRUNC('month', order_date),
	    SUM(revenue)
	FROM orders
	GROUP BY customer_id, DATE_TRUNC('month', order_date);
	```
	- This is very different from "Give me the order details for `order_id = 12345`."
	- The first is an OLAP workload. The second is an OLTP workload.
- **Scenario 1 - Choosing The Correct Database**:
	- Suppose you're designing an e-commerce platform. The application receives:
		- **50,000 orders/sec**
		- Individual orders need to be created and updated quickly.
		- The application frequently looks up individual orders by `OrderID`.
		- Customers need low-latency access to their order information.
	- Separately, the analytics team needs to run queries such as: "What was the total revenue by product category and region for each quarter over the last five years?"
	- **Would you use Redshift as the primary database for the e-commerce application? If not, what characteristics of the two workloads make them better suited to different database technologies**?
> 	I would choose DynamoDB as the primary database for the e-commerce application since it primarily uses transactional workloads. Redshift would be better suited to support the needs of the analytics team.
		- The application has an **OLTP-style workload**:
			- Very high write volume
			- Frequent individual record lookups
			- Low-latency requirements
			- Predictable access patterns such as `OrderID → Order`
		- The analytics workload is fundamentally different:
			- Large historical datasets
			- Complex aggregations
			- Scanning many rows
			- Grouping and joining data
			- Analytical queries over long time periods
		- The important takeaway is: **Don't choose a database based solely on how much data it stores. Choose it based on the workload and access patterns**.

## Redshift vs. Athena

- Suppose your company already has **20 TB of historical sales data in S3**, stored as Parquet.
- The analytics team runs approximately **10 queries per day**. The queries are moderately complex but don't need sub-second latency.
- **Would you load this data into Redshift, or would you query it directly using Athena? What factors would you consider when making that decision?**
> 	Since the data is already stored in Parquet format and the analytics team only needs to run 10 moderately-complex queries per day, it could make sense to keep the data in S3 and query it using Athena. When making the decision to switch to Redshift, you'd need to evaluate the difference in cost, operational overhead, and future demand.
	- There's little reason to introduce a dedicated warehouse if the data is already in an analytics-friendly format and query demand is relatively low.
	- Athena:
		- No dedicated warehouse to manage
		- Query data directly in S3
		- Good for intermittent/ad-hoc analytics
		- Pay based largely on data scanned
	- Redshift:
		- Dedicated analytical warehouse
		- Better suited to **sustained, high-volume** analytical workloads
		- Can provide more predictable performance
		- Introduces ongoing compute/capacity considerations

## Performance Considerations

- Suppose you have a table with:
	- 1 billion orders
	- Columns:
		- `OrderID`
		- `CustomerID`
		- `ProductID`
		- `OrderDate`
		- `Region`
		- `Revenue`
		- `PaymentMethod`
		- `ShippingAddress`
- An analyst runs:
	```sql
	SELECT
	    Region,
	    SUM(Revenue)
	FROM orders
	WHERE OrderDate >= '2026-01-01'
	GROUP BY Region;
	```
	- Redshift doesn't need to treat the table like a traditional row-oriented transactional database. Redshift uses **columnar storage**, meaning data for the same column is stored together.
	- For the query above, Redshift primarily needs `Region`, `Revenue`, and `OrderDate`. The rest of the columns can be ignored when scanning the data.
- **Why is columnar storage particularly beneficial for analytical queries like this, but potentially less useful for an OLTP workload that frequently retrieves or updates an entire individual row**?
> 	Columnar storage is beneficial for analytical workloads because it provides the ability prune unnecessary columns, significantly reducing the amount of data that needs to be scanned. For transactional workloads, which retrieve or update entire roads, this ability provides little to no performance benefit.

### Data Distribution
- Redshift is a distributed system. A large table can be spread across multiple compute nodes so that queries can execute in parallel.
- Suppose you have:
	- 1 billion orders
	- Redshift Cluster:
		- Node 1
		- Node 2
		- Node 3
		- Node 4
	- If one node receives substantially more data than the others, that node becomes a bottleneck.
- Redshift provides several distribution strategies:
	- **Key Distribution**:
		- You select a column as the **distribution key**, and Redshift uses that value to determine where rows should be stored. For example, if `customer_id` is chosen as the distribution key, rows will be distributed across nodes based on `customer_id`.
- **Scenario 1**: Your `orders` table contains 1 billion rows. You need to frequently join it with a `customers` table using: `orders.customer_id = customers.customer_id`.
- **Why might choosing `customer_id` as the distribution key for `orders` improve query performance? And what potential problem would you investigate before making that choice**?
> 	I would consider using customer_id as the distribution key for orders because it needs to be joined with the customers table. Before making the choice, I'd investigate how well customer_id distributes data across the combined table.
	- If both `orders` and `customers` are distributed using the same `customer_id` key, rows belonging to the same customer can be colocated on the same Redshift node. Thiscan reduce the amount of **data redistribution across nodes during the join**, substantially improving performance for large tables.
	- Setting the distribution key: `DISTKEY(customer_id)`
	- Don't focus on how well `customer_id` distributes data across the combined table. Instead, focus on the **distribution of `customer_id` values within the table being distributed**. For example, if `Customer A` has 400 million rows, while the average customer has hundreds of rows on average, `customer_id` would make a poor distribution key because it would create a **data-skewed node**.
- **Scenario 2**: Suppose `customer_id` is highly skewed, so using it as a `DISTKEY` would create an uneven distribution. However, your queries **frequently join `orders` with `customers` on `customer_id`**. You still want to avoid expensive data redistribution during the join.
- **What would you consider instead of forcing `customer_id` to be the distribution key**?
	- If `customer_id` is **highly skewed**, you have two competing goals:
		- Keep related rows colocated to make joins efficient.
		- Avoid concentrating too much data on one node.
	- Redshift gives you another distribution strategy: **`ALL` distribution** (`DISTSTYLE ALL`). With `ALL`, Redshift stores a copy of the table on **every compute node**. This can make sense for a **small dimension table** such as `customers`.
	- Then you could distribute the much larger `orders` table using a more evenly distributed strategy, while every node already has a copy of `customers`. The join can then happen locally without having to redistribute the entire `customers` table.
	- **The tradeoff**: `ALL` isn't appropriate for huge tables because you're storing multiple copies of the data. It's generally useful when a relatively small table is frequently joined with large distributed tables.
	- Another option: `EVEN`
		- If you don't have a good distribution key, Redshift can distribute rows **evenly across the nodes**.
		- So if `customer_id` is heavily skewed, you could rely in `EVEN` distribution instead.
		- The major tradeoff is that joins may require **data distribution across nodes**.
> 	If customer_id is too skewed to use as the distribution key, I'd consider distributing the large orders table evenly and using ALL distribution for the relatively small customers dimension table. That avoids the skew while keeping the dimension table available on every node for local joins.
- **Key takeaway:** just like with DynamoDB and Spark, **a good distribution key needs to balance the access pattern with even data distribution**.

### Sort Keys
- A **sort key** determines how rows are physically ordered within Redshift tables. This is particularly useful when queries frequently filter or range-scan on a particular column.
- Suppose you have: `orders`: `[order_id, customer_id, order_date, region, revenue]`. Analysts frequently run:
	```sql
	SELECT
	    region,
	    SUM(revenue)
	FROM orders
	WHERE order_date >= '2026-01-01'
	GROUP BY region;
	```
	- If `order_date` is an appropriate sort key, Redshift can use the ordering of the data to **avoid scanning irrelevant portions of the table**.
	- This is conceptually similar to what you learned with Athena and partitioning, but they're not the same mechanism. Partitioning in S3/Athena allows entire partitions to be skipped based on how data is filtered in a query. Sort keys allow data **within a partition** to be skipped if the data is sorted and falls outside of the requested range.
- **Scenario**: You have a Redshift `orders` table containing **10 billion rows**. Most analytical queries filter by `order_date`, such as: `WHERE order_date BETWEEN '2026-01-01' AND '2026-01-31'`.
- `order_date` is currently **not** a sort key.
- **Why could adding `order_date` as a sort key improve query performance? What would be the downside of choosing a sort key that doesn't align with the workload's common query patterns**?
> 	order_date could act as a good sort key if analysts frequently filter data based on order_date. Using this column as a sort key would allow Redshift to skip scanning sections of data within a partition based on the requested range. Choosing a sort key that doesn't align with common query patterns eliminates this benefit.
	- In Redshift, "sections of the table" is more accurate than saying "sections of a partition" because Redshift's distribution and sorting are separate concepts.
	- Optimize physical data layout around the workload's actual access patterns.

### Distribution Key vs. Sort Key
- Distribution keys determine **which node stores a row**. The primary concern is: "How do I distribute data across nodes and minimize expensive data movement during joins?"
- Sort keys determine **how data is ordered within the storage of a node**. The primary concern is: "How can I efficiently locate/filter the data that queries commonly need?"
- Distribution and Sort keys can be used together, assuming those choices actually fit the workload and don't create problematic skew:
	```
	orders
	
	DISTKEY(customer_id)
	SORTKEY(order_date)
	```
- **Scenario**: Distribution + Sort Keys Together
- Suppose you have a Redshift `orders` table with:
	- 5 billion rows
	- `customer_id` is reasonably evenly distributed
	- Analysts frequently join `orders` to `customers` on `customer_id`
	- Analysts frequently filter orders by `order_date`
- **What distribution key and sort key would you consider for `orders`, and why would you choose each one**?
> 	I would choose customer_id as the distribution key because it evenly distributes data across nodes. I would choose order_date as the sort key because analysts frequently filter data based on order_date. This allows data within a node to be effectively queried based on order_date.
	- **`customer_id` as the distribution key:** The prompt tells us it's reasonably evenly distributed, and it matches the frequent join between `orders` and `customers`. This can reduce data redistribution during the join.
	- **`order_date` as the sort key:** Analysts frequently filter by `order_date`, so ordering the data around that column can make those range queries more efficient.
	- **Important Point**: The distribution key **doesn't** have to be the column with the highest cardinality. What matters is whether it **provides reasonably even distribution _and_ supports important access patterns** such as joins.

## Loading Data

- Suppose your company has a daily batch of sales data arriving in S3:
	```
	s3://sales/
	    2026-08-25/
	        part-001.parquet
	        part-002.parquet
	        ...
	```
- You need to load that data into a Redshift `sales` table every night.
- **Why might you prefer loading the data into Redshift in batches from S3 rather than having your application make an individual `INSERT` into Redshift for every sale**?
	- Think about:
		- Redshift's intended workload
		- Network/database overhead
		- The difference between OLTP and OLAP
		- The number of records involved
> 	Redshift is designed as an OLAP database that is designed for handling data in bulk, not performing individual inserts. Loading data in batches fits this type of workload better than individual inserts. It also saves network overhead because you would be making one request per batch instead of one request per row. If the batch size is large enough, this benefit becomes meaningful.
- A common architectural pattern is:
	```
	Application / OLTP systems
	          ↓
	         S3
	          ↓
	     Bulk loading
	          ↓
	      Redshift
	          ↓
	     Analytics / BI
	```
	- Important Points:
		- Redshift is an **OLAP** system designed for analytical workloads.
		- Bulk loading is much more appropriate than row-by-row inserts.
		- Batching reduces **network and request overhead**.
		- Larger batches allow Redshift to process the data more efficiently.
	- This also reinforces an important architectural principle: **Don't force an OLAP system to behave like an OLTP database**.
	- If an application is generating thousands of individual transactional writes per second, those writes should generally go to an appropriate operational datastore first. Redshift can then receive the data in bulk for analytical processing.

### Loading Data From S3
- There's an AWS-specific mechanism worth knowing here: the **`COPY` command**:
	```sql
	COPY sales
	FROM 's3://sales/2026-08-25/'
	...
	```
	- Instead of sending every row individually, Redshift can load a large collection of files from S3 in parallel. This fits naturally with Redshift's distributed architecture:
		```
		                 S3
		        ┌────────┼────────┐
		        ↓        ↓        ↓
		      Files    Files    Files
		        ↓        ↓        ↓
		     Redshift compute nodes
		        ↓        ↓        ↓
		          Parallel loading
		```
- **Scenario**: You have a Redshift cluster containing **5 billion orders**. You have already determined:
	- `customer_id` is evenly distributed.
	- Orders are frequently joined to `customers` using `customer_id`.
	- Analysts frequently filter orders by `order_date`.
	- Data arrives daily in large batches from S3.
- **Walk me through your overall design for this `orders` table. Specifically**:
	- Distribution key
	- Sort key
	- How you'd load the data
	- Why each choice makes sense for the workload
> 	I'd choose customer_id as the distribution key because it would evenly distribute data across all nodes. I'd use order_date as the sort key because analysts frequently filter orders by date. Finally, I'd load data in bulk, rather than loading individual records, because Redshift's distributed architecture is better suited at handling data in bulk.
		- Bulk loading also avoids the overhead of issuing individual database requests for every row.
- **Scenario 2**: You have 100 GB of Parquet data stored in S3. The analytics team runs **2–3 ad-hoc queries per week**. The queries don't have strict latency requirements and can take several minutes. There is no requirement for complex warehouse features, and the dataset is expected to remain relatively small.
- **Would you introduce Redshift? Or would you keep the data in S3 and use Athena**?
> 	I would keep the data in S3 and query it using Athena because there isn't a regular demand for intensive, low-latency queries that would justify using Redshift's distributed computing architecture.
	- Key Points:
		- Data is already in **S3 and Parquet**.
		- Query frequency is extremely low.
		- There isn't a strict latency requirement.
		- The workload doesn't justify maintaining a dedicated analytical warehouse.
		- Athena can query the data directly when needed.
- **Key Takeaway**: Redshift becomes more compelling when you have sustained analytical workloads with significant query volume, performance requirements, or warehouse-specific needs. Athena is often preferable for intermittent/ad-hoc queries directly against data in S3.

# ElastiCache / Redis

## Cache Hit Ratio

### Design Scenario
- You're building an e-commerce website. One product becomes extremely popular.
- The application looks like this:
	```
	Customer
	
	↓
	
	API
	
	↓
	
	DynamoDB
	
	↓
	
	Return Product
	```
	- Suddenly, traffic spikes to 200,000 TPS for the same product.
	- The product details typically don't change more than once a week.
- **How would you solve this problem**?
- Suppose we introduce a cache.
- The first request does this:
	```
	Cache
	
	↓
	
	Miss
	
	↓
	
	Database
	
	↓
	
	Populate Cache
	```
- The next 100,000 requests do this:
	```
	Cache
	
	↓
	
	Hit
	```
- **Why is this dramatically faster than querying the database every time**?
> 	Since the product details rarely change, I would place a cache in front of DynamoDB that could handle the repeated requests. Caches are optimized for fast and efficient reads, which would help reduce request latency compared to calling the database directly. Calling the database excessively for the same information, instead of using a cache, can lead to performance degradation and potential throttling.
	- When and application queries DynamoDB directly, the request may involve:
		- Network round trip.
		- Request parsing.
		- Authentication.
		- Routing.
		- Looking up the item.
		- Returning the result.
	- Even though DynamoDB is extremely fast, that's still a complete database request.
	- When using Redis to fulfill the request instead of DynamoDB, the data is already sitting:
		- In RAM.
		- In a very simple key-value structure.
		- Ready to return immediately.
	- Very little work is required.
	- That's why caches are measured in **microseconds**, while databases are often measured in **milliseconds**.
- Suppose someone says: "RAM is expensive. Let's store the cache on disk instead." **Would that still be a cache? Or would we lose most of the benefit**?
> 	Although RAM may be expensive, a disk-backed cache would still incur disk I/O, which is much slower than accessing data directly from memory. It may also introduce additional serialization/deserialization overhead, further increasing latency under heavy load.
	- Even if the data didn't need to be serialized, you'd still need to pay the cost of waiting for the storage device. **Disk I/O is the heavier cost burden, not serialization/deserialization**.
- **Why not just make the database faster instead of adding Redis**?
> 	Because caching addresses a different problem. Even if the database is highly optimized, repeatedly executing the same read wastes resources. A cache avoids unnecessary database work by serving frequently accessed data directly from memory, reducing both latency and database load.
- Why is Redis fast?
> 	Because it stores data in memory using efficient data structures, eliminating disk I/O and reducing the amount of work required to satisfy each request. That results in extremely low-latency lookups while also reducing load on the primary database.

### Performance Comparison
- Querying DynamoDB:
	```
	Application
	
	↓
	
	Network
	
	↓
	
	Database
	
	↓
	
	Storage lookup
	
	↓
	
	Return result
	```
	- Lots of work.
- Querying Redis:
	```
	Application
	
	↓
	
	Memory lookup
	
	↓
	
	Return result
	```
	- Much less work.
- Querying an in-process cache:
	```
	Application
	
	↓
	
	Local memory
	
	↓
	
	Return result
	```
	- Even less work.
- As we move the data closer to the application, we eliminate:
	- Network hops
	- Database work
	- Storage access
- Each step removes work from the request path.

### Knowledge Check
- Suppose you have a cache with a 95% cache hit ratio.
- Out of 100,000 requests, how many reach the database?
- Why is cache hit ratio one of the most important metrics for evaluating whether a cache is providing value?
> 	5,000 requests are reaching the database. cache hit ratio is one of the most important cache metrics because it tells you how effective the cache is at protecting the underlying data store from repeated requests.
	- Generally speaking, cache hit ratio is important because a higher ratio:
		- Lowers latency. More requests are served from memory.
		- Lowers database load.
		- Lowers cost. Fewer database reads means:
			- Less compute
			- Less throughput consumption
			- Lower infrastructure costs
- If your cache hit ratio suddenly drops from 95% to 40%, what do you investigate?
	- A strong engineer starts asking questions like:
		- Did the TTL become too short?
		- Are cache entries being invalidated too frequently?
		- Has the workload changed?
		- Are users requesting different data than before?
		- Is the cache undersized and evicting useful entries?

## Cache Invalidation

> There are only two hard things in Computer Science: cache invalidation and naming things.

### Design Scenario
- Suppose our product page is cached.
- Initially:
	```
	Database
	
	Nintendo Switch
	
	Price = $299
	```
- The cache contains:
	```
	Nintendo Switch
	
	Price = $299
	```
- Then someone updates the database.
	```
	Database
	
	Price = $249
	```
	- Unfortunately, the cached entry is now stale. The cache still contains: `Price = $299`.
	- Customers continue seeing the outdated price.
- **How would you prevent the cache from serving stale data? Can you think of one or more strategies**?
- Imagine the cache server crashes. It loses every cached entry.
- **What should the application do**?
	- Return an error?
	- Query the database?
	- Something else?
- Imagine 10,000 requests all arrive immediately after the cache is emptied.
- Every request asks for: `Nintendo Switch`.
- **What problem might that create? How could you reduce that problem**?
> 	When the price is updated in the database, a signal should be sent to the cache informing it of the update. This would tell the cache to invalidate the entry and either repopulate the updated entry immediately or upon the next cache miss. If a cache server crashes, it should be repopulated in the background while the application queries the database. Once the cache is restored, the application should resume querying the cache. The first cache miss should call the database while all other requests are told to wait. Once the cache is populated, the other requests should query the cache.
	- The invalidation signal being sent to the cache could come in the form of a DynamoDB stream.
	- Conceptually:
		```
		Update Database
		
		↓
		
		Publish Event
		
		↓
		
		Invalidate Cache
		
		↓
		
		Refresh Now
		
		or
		
		Refresh on Next Read
		```
- Question 1: There are two methods used to repopulated invalidated cache entries:
	- **Lazy Loading (Cache-Aside)**:
		```
		Invalidate
		
		↓
		
		Next Request
		
		↓
		
		Cache Miss
		
		↓
		
		Database
		
		↓
		
		Cache
		```
	- **Write-Through / Refresh**:
		```
		Database Updated
		
		↓
		
		Immediately Update Cache
		```
- Question 2:
	- The application queries the **database** while the cache is repopulated because the database **acts as the source of truth**, not the cache.
- Question 3:
	- Without coordination:
		```
		10,000 requests
		
		↓
		
		10,000 cache misses
		
		↓
		
		10,000 database queries
		```
	- Better approach:
		```
		10,000 requests
		
		↓
		
		One request rebuilds cache
		
		↓
		
		9,999 wait
		
		↓
		
		Cache populated
		
		↓
		
		All receive cached value
		```
		- This is commonly called:
			- Request coalescing
			- Single-flight pattern
			- Dogpile prevention
	- Instead of telling other requests to wait for the cache to be populated, other systems sometimes use:
		- Distributed locks
		- Request deduplication
		- Background refresh
		- Stale-while-revalidate
	- The important point is **preventing thousands of identical database queries**.
- **What happens if the cache goes down**?
> 	The application should gracefully fall back to the database. Performance may degrade temporarily, but correctness should not be affected because the database remains the source of truth.

## Cache Eviction

### Design Scenario
- Suppose your cache has enough memory for 1,000 products.
- But your application now has 10,000 products.
- Eventually, the cache becomes full. 
- Now a new product needs to be cached.
- Which existing item should be removed?
- **If you were designing the cache yourself, how would you decide which item to evict**?
- Product A receives 100,000 requests per day.
- Product B receives 1 request per year.
- The cache is full. **Which product would you remove? Why**?
- Now imagine Product A was really popular last Christmas, but hasn't been requested in the last 6 months.
- Meanwhile, Product C has suddenly become very popular this week.
- **Does your answer change? Why**?
> 	I would design the cache to remove the least frequently used item. Evicting the least frequently used item makes sense in this case because a popular product may not be queried while it is out of stock, but could receive a massive amount of requests once it is restocked. Since Product B only receives 1 request/year, I would event it instead of Product A. Using an LFU cache should evict Product A if Product C suddenly becomes more popular than Product A. However, setting a reasonable TTL would ensure a dormant product is evicted after a reasonable amount of time.
	- LRU would actually be more appropriate for the third scenario.
	- Imagine Product A gets 5,000,000 requests/day on Christmas. 6 months latter, Product B receives 100,000 requests/day while Product A has been mostly dormant.
	- An LFU cache would not evict Product A, an LRU cache would.
- Cache Policy Tradeoffs:
	- LFU
		- Good when:
			- Long-term popularity predicts future popularity.
			- Frequently used items stay valuable.
		- Examples:
			- Popular products
			- Frequently used configuration data
	- LRU:
		- Good when:
			- Recent activity predicts future activity.
		- Examples:
			- User sessions
			- Recently viewed products
			- Trending content
- Should I use LRU or LFU?
> 	It depends on whether historical popularity or recent activity is a better predictor of future access. If frequently accessed items remain valuable over long periods, LFU may be a better fit. If access patterns change rapidly, LRU often adapts more quickly.

### Knowledge Check
- Imagine a news website.
- Every hour:
	- Hundreds of new articles are published.
	- Yesterday's articles rapidly lose traffic.
	- Breaking news changes throughout the day.
- Would you choose an LFU or LRU cache? Why?
> 	I would use an LRU cache because recent activity is a better predictor of future activity than long-term popularity in this case. Articles rapidly lose traffic after they're read, so holding on to the historically most popular article wouldn't provide any meaningful benefit.
- Why doesn't Redis have one universally best eviction policy?
> 	Because different applications exhibit different access patterns. Some workloads have long-term hot data, making LFU effective. Others change rapidly, making LRU a better predictor of future requests. The optimal eviction policy depends on the application's workload rather than the cache implementation.

## Cache Write Strategies

### Design Scenario
- Suppose a customer updates their shipping address.
- The application currently looks like this:
	```
	Application
	
	↓
	
	Database
	
	↓
	
	Success
	```
- Now we've added Redis.
- The question becomes: When should Redis be updated?
- Option A:
	- The application writes to the database.
	- Then immediately updates the cache.
		```
		Application
		
		↓
		
		Database
		
		↓
		
		Cache
		```
- Option B:
	- The application writes to the cache.
	- The cache immediately writes to the database.
		```
		Application
		
		↓
		
		Cache
		
		↓
		
		Database
		```
- Option C:
	- The application writes to the cache.
	- The cache waits. Later, the cache asynchronously writes to the database.
		```
		Application
		
		↓
		
		Cache
		
		↓
		
		(wait)
		
		↓
		
		Database
		```
- **What are the advantages and disadvantages of each approach**?
- Suppose this is a banking application. **Which approach makes you the most comfortable? Why**?
- Now suppose this is an analytics platform ingesting 5 million events/sec. **Would your answer change? Why**?
> 	Option A ensures the cache is updated immediately after a successful database write and stays in sync with the database, but may end up writing a dormant item to the cache. Option B also ensures the cache and database stay in sync, but writes to the database take slightly longer. There is also a risk the cache could crash before writing to the database. Option C ensures the cache updates immediately, but further increases write latency to the database. For a banking application, I would prefer Option A because updates are written to the source of truth first, then immediately written to the cache to avoid inconsistencies. For an analytics platform ingesting data at a high rate, I would prefer Option C because it would buffer the amount of writes to the database.
	- The application waits until **both** writes succeed. So yes, the **overall operation** takes longer than writing only to the database, but not because the cache delays the database—it waits for both systems to complete before acknowledging success.
	- In **write-through**, the cache forwards the write immediately. That crash window is usually very small. The larger risk of losing data actually belongs to **Option C**.
	- Write-behind (Option C) is popular for:
		- Logging
		- Analytics
		- Metrics
		- Telemetry

### Summary
- **Cache-Aside**:
	```
	Read:
	
	Cache
	
	↓
	
	Miss
	
	↓
	
	Database
	
	↓
	
	Cache
	```
	- Best when:
		- Read-heavy
		- Not every item is read
- **Write-Through**:
	```
	Write Database immediately
	
	↓
	
	Update cache immediately
	```
	- Equivalently, write through the cache to the database immediately.
	- Best when:
		- Reads should immediately reflect writes.
		- Correctness matters.
- **Write-Behind**:
	```
	Write cache
	
	↓
	
	Return success
	
	↓
	
	Persist later
	```
	- Best when:
		- Massive write throughput
		- Eventual persistence acceptable
	- **Not ideal when data loss is unacceptable**.

### Knowledge Check
- Suppose you're building a service that collects:
	- CPU utilization
	- Memory utilization
	- Network throughput
- From **100,000 servers every second**.
- If one metric sample is occasionally lost, it's not a big deal because another arrives one second later.
- **Which write strategy would you choose**?
> 	I would choose the Write-Behind caching strategy because a small amount of data loss is acceptable and the buffering would lower the write throughput on the database. Cache-Aside and Write-Through would expose the database to the full write throughput of the service.
	- **Cache-Aside** is primarily a **read strategy**. Writes usually go directly to the database, and the cache is invalidated or updated afterward.
	- **Write-Through** is a **write strategy**, where every write synchronously reaches both the cache and the database.
- **Why don't banks use write-behind caching**?
> 	Because write-behind acknowledges the write before it has been durably stored in the database. If the cache fails before flushing its buffered writes, committed transactions could be lost. Banking systems prioritize durability and correctness over write throughput, so synchronous persistence is generally preferred.

| Strategy      | Prioritizes                |
| ------------- | -------------------------- |
| Cache-Aside   | Efficient reads            |
| Write-Through | Consistency and durability |
| Write-Behind  | Maximum write throughput   |

## Redis Data Structures

### Design Scenario
- **Scenario 1: User Profile**:
	- You're storing:
		```
		User
		
		Name
		
		Email
		
		Phone
		
		Address
		```
		- Would you rather store it as:
			- One giant JSON string.
			- A structure that stores each field separately.
		- **Why**?
- **Scenario 2: Unique Visitors**:
	- You need to answer: "How many unique users visited the homepage today?"
	- If the same person refreshes the page 100 times, should they only be counted 1 time?
	- **What kind of data structure would you choose**?
- **Scenario 3: Gaming Leaderboard**:
	- You need to display to the top 100 players, sorted by score.
	- Players constantly earn points.
	- The leaderboard is updated every few seconds.
	- **What kind of data structure would you design**?
- **Scenario 4: Shopping Cart**:
	- A customer has:
		```
		Nintendo Switch
		
		2
		
		Controller
		
		1
		
		Mario Kart
		
		1
		```
		- Would you rather store:
			- Four separate keys?
			- One object?
			- Something else?
		- **Why**?
> 	For Scenario 1, I'd rather store the user profile as a structure that stores each field separately. More often than not, a user's entire profile is not needed. Instead, certain fields, such as name, email, or phone, are needed. Storing the profile in a structure that stores each field separately would make retrieval faster. For Scenario 2, the user should only be counted once. To achieve this, I'd use a structure similar to a set, that only stores unique values. The only deciding factor would be what value to store. To prevent a user from being counted more than once if they refresh the page, session ID might be a good choice. For Scenario 3, I would design a cache with player ID as the partition key and score as the sort key. The data would be written to the cache and incrementally persisted to the data store in batches. For Scenario 4, I would store the shopping cart as a nested object. One attribute would be the customer ID and the second attribute would be an object mapping each item to its respective quantity. This would allow shopping carts to be queried by customer ID and have the items displayed using the nested object.
	- Scenario 3 asks about a **cache** design, not a database design. The partition and sort key are valid choices, but don't relate to a cache.
		- The most important operation is: Top 100 players
			- That means we only care about:
				- Fast score updates
				- Automatic sorting
				- Efficient ranking
			- The Redis Sorted Set was built specifically to meet these three requirements.
- **Why use a Hash instead of storing JSON**?
> 	Because hashes allow individual fields to be read and updated efficiently without replacing the entire object. They're a natural fit for records composed of multiple related attributes.

### Summary
- Scenario 1 would use a **Redis Hash** to store the user profile.
	- Conceptually:
		```
		User:123
		
		↓
		
		Name -> Calvin
		Email -> calvin@email.com
		Phone -> ...
		Address -> ...
		```
- Scenario 2 would use a **Redis Set** to store unique visitors.
	- Sets automatically prevent duplicates.
	- Examples of values that could be used to uniquely identify users include:
		- Session ID
		- User ID
		- Anonymous visitor cookie
	- The correct identifier depends on the business definition of "unique."
- Scenario 3 would use a **Redis Sorted Set** to store the leaderboard data.
	- Conceptually:
		```
		PlayerA -> 9500
		
		PlayerB -> 9300
		
		PlayerC -> 9100
		```
		- Redis automatically keeps the entries ordered by score.
- Scenario 4 would also use a **Redis Hash** to store the customer's shopping cart.
	- Something like:
		```
		Cart:123
		
		Nintendo Switch -> 2
		
		Controller -> 1
		
		Mario Kart -> 1
		```
		- The advantage is that individual quantities can be updated without replacing the entire cart.
		- The object isn't nested, but it still uniquely identifies shopping cart items and their respective quantities.

| Workload            | Redis Structure | Why                              |
| ------------------- | --------------- | -------------------------------- |
| User profile        | **Hash**        | Many related fields              |
| Shopping cart       | **Hash**        | Item → Quantity mapping          |
| Unique visitors     | **Set**         | Automatically removes duplicates |
| Leaderboard         | **Sorted Set**  | Maintains ordering by score      |
| Simple cached value | **String**      | Fast key-value lookup            |

### Knowledge Check
- Suppose you're building a chat application.
- Each user has:
	- An online/offline status.
	- A profile (name, avatar).
	- A list of friends.
	- A global leaderboard based on reputation points.
- For each piece of data, tell me **which Redis data structure you'd choose**:
	1. Online/offline status
	2. User profile
	3. Friend list
	4. Reputation leaderboard
> 	For an online/offline status, I'd use a Redis String because it offers fast, simple key-value lookup. For a user profile, I'd use a Redis Hash because it offers efficient key-value mapping. For a Friend list, I'd use a Redis Set if there was no sorting requirement or a Redis Sorted Set if there was a sorting requirement. For a reputation leaderboard, I'd use a Redis Sorted Set to maintain proper ordering.
	- Redis String:
		```
		User:123:Status
		
		↓
		
		"Online" (or "Offline")
		```
- **Why not store everything as Strings**?
> 	While everything could be serialized into a string, Redis data structures provide operations that are optimized for specific workloads. Using a Hash allows individual fields to be updated efficiently, a Set automatically enforces uniqueness, and a Sorted Set maintains ordering without requiring the application to sort the data itself.

| Workload           | Redis Structure | Reason                  |
| ------------------ | --------------- | ----------------------- |
| Cache a value      | String          | Simple key-value lookup |
| User profile       | Hash            | Many related fields     |
| Shopping cart      | Hash            | Item → Quantity mapping |
| Friend list        | Set             | Unique collection       |
| Sorted friend list | Sorted Set      | Ordered collection      |
| Unique visitors    | Set             | Prevent duplicates      |
| Leaderboard        | Sorted Set      | Ordered by score        |

## Pub / Sub

### Design Scenario
- You're building a chat application.
- Alice sends: "Hello!"
- Bob is currently online. The message should appear on Bob's screen almost instantly.
- **Option A**:
	- Bob's application repeatedly asks:
		```
		Any new messages?
		
		↓
		
		No
		
		↓
		
		Any new messages?
		
		↓
		
		No
		
		↓
		
		Any new messages?
		
		↓
		
		Yes
		```
		- At least once every second.
- **Option B**:
	- Instead, Alice's message is **published**.
	- Bob's application is already **subscribed**.
	- As soon as the message is published:
		```
		Alice
		
		↓
		
		Publish
		
		↓
		
		Bob immediately receives it
		```
	- **Which approach would you choose? Why**?
		- Think about:
			- Latency
			- Network traffic
			- Scalability
- Now, suppose Bob is offline.
- Alice sends 10 messages.
- Bob reconnects five minutes later.
- **Should Redis Pub/Sub automatically deliver those missed messages? Or is there a limitation here**?
- Now, imagine you're building:
	- A payment system
	- An order processing system
	- An inventory system
- **Would Redis Pub/Sub be your first choice? Or would you prefer a more durable messaging system? Why**?
> 	I would choose Option B because Bob's application doesn't need to keep asking if there are new messages. This saves network resources on both ends. It's also faster and more scalable. Instead of requesting messages once a second, Bob could receive Alice's messages in milliseconds. If Alice and Bob wanted to start a group chat, other members would simply need to subscribe to receive Alice's messages. If Bob goes offline, the messages should automatically be delivered because he is still subscribed. For an inventory system, Pub/Sub would be a good implementation for sending out inventory updates to multiple subscribes. For a payment and order processing system, Pub/Sub wouldn't be a good idea durability is a higher concern than quickly sending updates to multiple subscribers.
	- Redis Pub / Sub is only designed for **live communication**. If Bob goes offline, he won't receive any messages Alice sent during that time period. The messages are not stored or persisted by Redis.
	- If you needed **guaranteed delivery**, you'd typically choose something that persists messages until consumers process them, such as:
		- A message queue
		- An event streaming platform
		- **Redis Streams** (not Redis Pub / Sub)
	- **Pub / Sub is ephemeral**. Subscribers only receive messages while they're actively connected.
	- Durability is a higher concern for Payment and Order Processing. For payments:
		- You don't want to lose a message because a consumer briefly disconnected.
		- You need **acknowledgments, retries, and persistence**.
		- Pub/Sub intentionally doesn't provide those guarantees.
- Why not use Redis Pub/Sub for order processing?
> 	Because Redis Pub/Sub is designed for real-time message distribution, not guaranteed delivery. If a subscriber is offline when a message is published, the message is lost. Order processing requires durability, retries, and reliable delivery, so I'd choose a durable messaging system instead.

### Summary
| Redis Pub/Sub          | Durable Messaging            |
| ---------------------- | ---------------------------- |
| Very low latency       | Slightly higher latency      |
| No message persistence | Messages are persisted       |
| No replay              | Consumers can catch up       |
| Great for live events  | Great for critical workflows |
Always ask: **What guarantees does the business require**?

## Mock Interview Questions

1. You're designing the backend for a large online multiplayer game.
	- Requirements:
		- Player profiles are read constantly but change infrequently.
		- A live leaderboard displays the top 1,000 players.
		- Friends receive an instant notification when someone comes online.
		- Millions of gameplay events are collected every minute for analytics.
		- Players can resume an unfinished match within 30 minutes if they disconnect.
		- Match results affect player rankings and must never be lost.
	- **Walk me through your design**.
	- Specifically discuss:
		- Where you would use Redis.
		- Which Redis data structures you would choose.
		- Whether you'd use Pub/Sub.
		- Which write strategy fits the analytics pipeline.
		- Which operations should bypass Redis entirely.
		- Which components require durability versus speed.
		- Any tradeoffs you would consider.
> 	I'd cache player profiles in Redis because they're read frequently but updated infrequently, storing them as Redis Hashes. I'd use a Redis Sorted Set for the leaderboard because it efficiently maintains player rankings. I'd use Pub/Sub for online presence notifications because they're real-time but don't require durable delivery. Temporary match state would be stored in Redis with a 30-minute TTL so players can reconnect without leaving stale state indefinitely. For analytics events, I'd use a write-behind strategy to buffer the high volume of writes before persisting them. For match results, I'd bypass write-behind and use a write-through or direct database write because rankings and rewards require durable storage.
			- Profiles should be cached using a **Redis Hash** rather than a string because profiles naturally contain multiple fields.
			- If someone is offline, missing an "online" notification is inconsequential because another update will eventually arrive.
			- Temporary match state should be stored in Redis with a TTL of 30 minutes.

# Simple Notification Service (SNS) & Simple Queue Service (SQS)

## Overview

### Design Scenario
- Imagine you're building an e-commerce platform. Whenever an order is placed, the following systems need to be notified:
	- Email Service
	- Inventory Service
	- Billing Service
	- Analytics Service
- **Option A**: The Order Service directly calls each downstream service:
	```
	Order Service
	
	↓
	
	Email
	
	↓
	
	Inventory
	
	↓
	
	Billing
	
	↓
	
	Analytics
	```
- **Option B**: The Order Service publishes a single message. A messaging service distributes that message to all **interested** consumers:
	```
	Order Service
	
	↓
	
	Message
	
	↓
	
	Messaging Service
	
	↓
	
	Email
	
	Inventory
	
	Billing
	
	Analytics
	```
- **Which architecture would you choose and why**?
	- Think about:
		- Loose coupling
		- Scalability
		- Adding new downstream services
		- Failure isolation
> 	I would choose option be because it loosely couples the order service with the downstream serve. The order service only needs to worry about publishing the message to the message broker. The message broker is then responsible for distributing the message to all active subscribers. This allows the order service to scale easily as the number of downstream dependencies increases. It also isolates failures to the singe service(s) which failed to properly process the message using its respective processing logic.
		- SNS is technically called a pub/sub messaging service, not a message broker.
- Now imagine the Inventory Service goes offline for 30 minutes. Should:
	- The message be lost?
	- The Order Service wait?
	- The message be stored somewhere until Inventory comes back?
- **How would you design the service**?
> 	Ideally, messages should be retained for a practical period of time, allowing the inventory service to retrieve the messages it did not receive while it was offline. The order service wouldn't need to wait on the inventory service because each downstream dependency should be able to retrieve messages independently. I would design the message broker to retain messages for an appropriate amount of time, rather than storing messages in a separate service.
	- SNS should not provide message durability. SQS queues should act as subscribers which can send and retain messages bound for each downstream service. That's an extremely important SNS + SQS pattern.
	- Separate queues are needed because each consumer needs an **independent copy of the message**. If you put all four consumers on one SQS queue, they would compete for messages. A message consumed by Inventory would no longer be available to Billing.
- Suppose instead of four services needing the message, there's **only one** background worker responsible for resizing uploaded product images.
- **Would you still use the same messaging pattern? Or would you choose something different**?
	- Think about:
		- **One producer → Many consumers**
		- **One producer → One consumer**
> 	If only one downstream service needed the message, I would use a message queue instead of a message broker, since there is a one-to-one relationship between publishers and subscribers, instead of a one-to-many relationship.
			- Use SQS when you have a queue-based workload where messages are processed by workers independently.

### Mental Model
- SNS: "Broadcast this event to everyone interested."
	```
							 ┌→ SQS → Email
							 │
	Order Service → SNS ─────┼→ SQS → Inventory
							 │
							 ├→ SQS → Billing
							 │
							 └→ SQS → Analytics
	```
	- Provides fanout of SNS with durability of SQS.
- SQS: "Put this work somewhere so a worker can process it reliably."
	- SQS offers a fanout pattern similar to SNS. The key distinction is that **each message in the queue is processed once**. Unlike SNS, a copy of each message is not sent to each subscriber.
	- Instead, each subscriber asynchronously processes messages from the queue, allowing the queue to be cleared more quickly.
	- This is one of the major reasons **SQS is useful for decoupling producers from consumers**: the producer doesn't have to slow down simply because consumers temporarily can't keep up.
	- Think of one really long grocery line being handled by multiple cashiers. Each cashier doesn't repeatedly process the same customer's cart. Instead, each cashier handles one customer at a time, clearing the line more quickly than one lone cashier.
	- Services pulling messages from SQS need to be designed to be idempotent, in order to prevent a scenario where a message that is processed more than once creates **duplicate effects**.
	- SQS keeps sending a message to a consumer until the consumer deletes the message.
	- Important metrics:
		- Queue Depth: Number of messages waiting in the queue.
		- Age of Oldest Message: Provides an indication of the likelihood of messages expiring before they're processed.
		- Consumer Processing Rate: How quickly consumers are processing and deleting messages.
		- Consumer Errors: Number of message processing errors.
		- Message Visibility / In-Flight Counts: How long a message is visible to a consumer.

## Core Concepts

### Dead-Letter Queues
- A Dead-Letter Queue (DLQ) can be configured with an appropriate maximum receive count. This prevents poison messages from being retried indefinitely, which wastes consumer capacity and can prevent the system from efficiently processing healthy messages.
- The DLQ also gives the team a place to investigate and potentially **redrive failed messages** after resolving the underlying issue.

### Visibility Timeout
- After a worker receives a message, SQS temporarily hides it from other consumers. If processing succeeds before the timeout, the message is deleted and becomes permanently unavailable to other consumers.
- If a worker crashes before deleting a message or takes too long to process a message, the visibility timeout expires and the message becomes available to other consumers.
- **This is another way duplicate processing can occur**.

### Message Retention
- The SQS retention period determines how long a message can remain in a queue before being automatically deleted.
- This is distinct from the visibility timeout, which determines how long a message remains invisible to other consumers while it is being processed

# Kinesis Data Streams

## Overview

- The first distinction to understand is that Kinesis Data Streams is designed for **real-time streaming data**.
- A useful mental model is:
	```
	Producers
	   │
	   ├── Event
	   ├── Event
	   └── Event
	        ↓
	┌─────────────────────┐
	│ Kinesis Data Stream │
	│                     │
	│ Shard 1             │
	│ Shard 2             │
	│ Shard 3             │
	└─────────────────────┘
	        ↓
	   Consumers
	   ├── Lambda
	   ├── Flink
	   └── Custom apps
	```
- Unlike SQS, where the typical model is **a message being processed by one consumer**, Kinesis allows multiple independent consumers to process the **same stream of records**.
- The stream also retains records for a configurable period, allowing consumers to **re-read records** rather than permanently losing them after one consumer processes them.
- **Scenario**: Imagine an advertising platform receives **100,000 impression events per second**, with peaks of **500,000 events/sec**.
- You need to:
	- Process the events in near real time to update an analytics dashboard.
	- Have a separate consumer process the events for billing.
	- Store the events in S3 for historical analysis.
	- Allow a new analytics consumer to process **historical events** when it is deployed.
- Why might **Kinesis Data Streams** be a better fit than **SQS** for this workload?
> 	Kinesis is more appropriate for this use case than SQS because multiple consumers need to ingest the same events for different purposes. Additionally, historical events need to processed by an analytics consumer. Instead of using an SNS + SQS fanout pattern, Kinesis Data Stream allows the same events to processed by multiple consumers without being deleted.
	- Kinesis provides a **persistent ordered stream of records** that consumers track independently. A consumer can fall behind, catch up, or reread records within the stream's retention period.
	- Kinesis doesn't mean records are literally "never deleted." Records are retained for a configured retention period and then expire automatically.

## Shards & Partition Keys

- A Kinesis Data Stream is divided into **shards**:
	```
	             Stream
	                │
	       ┌────────┼────────┐
	       ↓        ↓        ↓
	    Shard 1  Shard 2  Shard 3
	```
- When a producer sends a record, it provides a **partition key**. Kinesis uses that partition key to determine which shard receives the record. This means your partition-key choice affects **how evenly the workload is distributed**.
- **Scenario**: Your advertising platform receives events containing:
	- `user_id`
	- `advertiser_id`
	- `campaign_id`
	- `event_type`
	- `timestamp`
- Your data stream has three shards. You initially choose `advertiser_id` as the partition key, but one advertiser is responsible for 60% of the traffic.
- What problem could this create in Kinesis, and what would you investigate when choosing a better partition key?
> 	Using advertiser_id as the partition key could lead to poor workload distribution among the shards of the data stream. One shard would receive about 60% of advertising events while others would receive relatively few events. When choosing a partition key, I'd look at how evenly the key distributes workload, not just data.
	- Choose partition keys based on expected traffic distribution, not simply cardinality or record distribution.
- If `advertiser_id` is the partition key and one advertiser generates 60% of traffic:
	```
	Advertiser A → 60% of events
	Advertiser B → 10%
	Advertiser C → 10%
	...
	             ↓
	       Kinesis routing
	             ↓
	Shard 1 → ████████████████████ 60%
	Shard 2 → ███                  10%
	Shard 3 → ███                  10%
	```
	- Shard 1 can become a **hot shard**, potentially limiting throughput even though the stream as a whole has plenty of capacity.

## Ordering

- There's another important reason partition-key selection matters. Kinesis guarantees **ordering within a shard, not across shards**.
- Suppose you're processing gameplay events:
	```
	Player 123:
	  login
	  purchase
	  level_up
	  logout
	```
	- You need those events to be processed **in that order**.
	- A player's `user_id` should be used as the partition key for the data stream since all of the player's events will be sent to the same shard, where ordering will be guaranteed.
- **Important Tradeoff**:
	- **Good partition key** → evenly distributes traffic.
	- **Ordering requirement** → related records must share a partition key/shard.
	- The partition key needs to balance **both workload distribution and ordering requirements**.

## Consumer Failure

- Let's say you have:
	```
	Kinesis
	   ↓
	Analytics Consumer
	```
	- The consumer successfully processes records through sequence number `1000`, but then crashes.
	- When it comes back online, you **don't want it to start at the newest record and permanently skip records 1001–1050**.
- How does Kinesis allow a consumer to resume processing from where it left off? What mechanism would you use to track the consumer's position in the stream?
	- A Kinesis consumer typically:
		1. Processes records.
		2. **Checkpoints** the sequence number it has successfully processed.
		3. If it crashes, resumes from the last checkpoint rather than starting over from the beginning.
	- With the **Kinesis Client Library (KCL)**, checkpointing is commonly managed through DynamoDB.
	- One important nuance: the consumer doesn't simply "send the sequence number to Kinesis." **Kinesis stores the stream records; the consumer/application is responsible for tracking its checkpoint**.
	- Checkpointing does not eliminate duplicate processing. If a consumer processes a record successfully but crashes **before checkpointing it**, that record may be processed again after recovery. Therefore, the downstream processing should ideally be **idempotent**.

### Slow Consumers
- Let's say your stream receives 100,000 events/sec, but the analytics consumer can only process 80,000 events/sec.
- The producer continues sending 100,000 events/sec. Over time, the consumer falls further and further behind.
- What problem does this create for the consumer, and what metrics or signals would you monitor to determine whether the consumer is falling behind?
> 	Since the consumer is lagging behind at a rate of 20,000 events/sec, it may never be able to process the events at the end of the stream before the retention period expires. A consumer's processing latency can be a good indicator of whether it is falling behind. If the consumer can't process events at the same rate they're being published to the stream, it will fall behind.
	- The bigger concern isn't just processing latency; it's **consumer lag**—how far behind the consumer is from the latest record in the stream.
	- Key Metrics:
		- **Consumer lag / iterator age** → directly tells you how far behind the consumer is.
		- **Consumer processing rate** → compare records processed/sec against records produced/sec.
		- **Incoming records/bytes** → determine whether producer throughput has increased.
		- **Retention period** → determine whether the consumer risks falling so far behind that records expire before being processed.

### Consumer Scaling
- Suppose your stream has **4 shards**, and your analytics consumer has **4 workers—one processing each shard**.
- You discover:
	- Shard 1 = 95% utilization
	- Shard 2 = 30%
	- Shard 3 = 25%
	- Shard 4 = 20%
- The overall stream still has plenty of unused capacity, but the consumer is falling behind.
- Would simply adding more consumer workers necessarily solve the problem? If not, what would you investigate?
> 	Adding more workers wouldn't necessarily solve the problem, especially if the consumer processing events from shard 1 is falling behind due to an issue with a downstream dependency. You'd need to first investigate why the consumer is falling behind by investigating metrics such as latency, memory utilization, and CPU utilization.
	- **The uneven shard utilization itself is a major clue**. Because **Shard 1 is the bottleneck**, adding workers to the other shards doesn't help.
	- Things to investigate:
		- Whether the **partition key is causing a hot shard**.
		- Whether records assigned to Shard 1 are inherently more expensive to process.
		- Consumer CPU/memory/latency for that shard.
		- Whether the downstream dependency is slower for those records.
		- Whether the shard itself has reached its throughput limit.
	- If the problem is a **hot partition caused by the partition key**, the solution may involve changing the partitioning strategy or increasing shard capacity—not simply adding consumers.
	- Before scaling horizontally, identify whether the bottleneck is actually the number of workers or a constrained resource downstream/underneath them.

### Hot Partitions
- You discover the consumer isn't actually slow. The problem is that **70% of incoming events are being routed to Shard 1**because of the chosen partition key. The other shards have plenty of capacity.
- What are two different approaches you could take to address this hot-shard problem?
> 	Since there's no issue with consumer performance and 70% of events are being routed to one shard, changing the partitioning strategy would be more effective than increasing shard capacity. Increasing available shard capacity could theoretically work, but would be a brute-force approach. It wouldn't solve the underlying traffic distribution problem. The partitioning strategy would need to be modified to more evenly distribute traffic while preserving necessary event ordering.
	- Traffic distribution **must always be balanced with required ordering**. Don't sacrifice required ordering merely to improve distribution.

### Multiple Consumers
- Suppose you have:
	```
	                 Kinesis
	                    │
	        ┌───────────┼───────────┐
	        ↓           ↓           ↓
	    Analytics     Billing      S3
	    Consumer      Consumer    Consumer
	```
	- All three consumers need to process the **same events independently**.
	- The Analytics consumer occasionally falls behind, but Billing must continue processing events in near real time.
- Why is Kinesis's independent consumer model useful here? What would happen if the Analytics consumer became very slow? Would it prevent the Billing consumer from continuing to process newer records?
> 	Kinesis's independent consumer model is useful here because each consumer can process events at varying rates, without disrupting one another. If analytics falls too far behind, it could fail to read certain events before their retention period expires. This would not impact any other consumer.
	- If Analytics falls behind, its lag increases independently. **Billing doesn't have to wait for Analytics** and can continue reading newer records.
	- The main consequence for Analytics is that it could eventually reach the stream's **retention boundary** and lose the ability to read records that have expired.
- Kinesis vs. SQS:
	- **SQS**: Messages are generally consumed from a queue by **competing** consumers.
	- **Kinesis**: Multiple consumers can independently read the same stream and maintain independent positions.

# Amazon Data Firehose

## Overview

- Kinesis Data Steams and Firehose involve streaming data, but they solve **different problems**.
- Mental Model:
	- Data Streams:
		```
		Producers
		    ↓
		Kinesis Data Streams
		    ↓
		Consumers
		 ├── Lambda
		 ├── Flink
		 ├── Custom application
		 └── ...
		```
		- You get control over how records are consumed and processed.
	- Data Firehose:
		```
		Producers
		    ↓
		Firehose
		    ↓
		Buffer / Batch
		    ↓
		Destination
		 ├── S3
		 ├── Redshift
		 ├── OpenSearch
		 └── ...
		```
		- Firehose is primarily a **managed delivery mechanism** for getting streaming data into destinations.
		- You don't manage shards or consumer applications in the same way.
- **Scenario**: Your application generates 50,000 click events/sec. The business wants all events continuously delivered to **S3** for later analytics.
- There is **no requirement for:**
	- Custom real-time processing
	- Multiple independent consumers
	- Replayable consumer positions
	- Complex stream transformations
- The primary requirement is simply: "Reliably get these events into S3 with minimal operational overhead."
- Would you choose **Kinesis Data Streams or Kinesis Data Firehose**?
> 	I would choose Kinesis Data Firehose because there is not requirement for custom real-time processing, independent consumers, or repayable consumer positions. The primary requirement is to reliably load events to S3 with minimal operational overhead. Firehose can optimize delivery to S3 by collecting events into appropriately-sized batches and writing them to S3 batch-by-batch, instead of event-by-event, which would help reduce costs associated with API calls.
	- **Firehose removes much of the operational burden**. You don't need to manage shards, consumers, checkpoints, or custom stream-processing infrastructure when the requirement is essentially: "Take streaming data and reliably deliver it to S3."
	- Firehose buffers records before delivering them to the destination, which is much more efficient than making an S3 write for every individual event.
- **Mental Model**:

| Requirement                               | Better fit                |
| ----------------------------------------- | ------------------------- |
| Custom real-time processing               | **Kinesis Data Streams**  |
| Multiple independent consumers            | **Kinesis Data Streams**  |
| Replay/control consumer position          | **Kinesis Data Streams**  |
| Simple streaming → S3 delivery            | **Kinesis Data Firehose** |
| Minimize stream infrastructure management | **Kinesis Data Firehose** |

## Delivery Latency

- Suppose you have a requirement: Click events must appear in S3 within approximately 60 seconds.
- You configure Firehose to deliver to S3.
- Why might Firehose **not deliver every event immediately** after it arrives? What tradeoff is Firehose making when it buffers records before delivering them to S3?
> 	Firehose might not deliver every event immediately after it arrives because events are only required to be available within 60 seconds, not immediately. The tradeoff being made when events are buffered is that delivery latency will be higher, but API costs will be reduced because S3 doesn't need to be called for every event.
- The basic flow is:
	```
	Events
	  ↓
	Firehose buffer
	  ↓
	Buffer reaches size/time threshold
	  ↓
	Batch delivery
	  ↓
	S3
	```
	- Firehose deliberately trades some **delivery latency** for **efficient batched delivery**.
	- The important point is that Firehose can buffer based on configured delivery conditions, so the destination doesn't need to receive one write per event.

## Transformations

- Suppose your incoming events look like:
	```
	{
	    "user_id": 123,
	    "event_type": "click",
	    "email": "calvin@example.com",
	    "timestamp": "..."
	}
	```
- Before storing the data in S3, you want to:
	- Remove the `email` field.
	- Convert the timestamp into a standardized format.
	- Add a derived field.
- You don't need complex distributed processing; the transformation is small and performed independently on each record.
- Would Firehose be capable of handling this type of transformation? When would you decide that the transformation is complex enough that you should instead use something like **Glue/Spark or another dedicated processing system**?
> 	Yes, Firehose is capable of handling the transformations because they don't require distributed processing. They're small and performed independently on each record. You would want to use a dedicated processing system when the data requires heavier transformations, such as joins or aggregations.
	- Firehose transformations are a good fit when each record can be transformed **independently** without needing significant state or distributed computation.
	- Once the transformation requires things like:
		- **Joins** between datasets
		- Large-scale **aggregations**
		- Complex multi-stage transformations
		- Significant distributed computation
		- **Stateful processing** across many records
	- You'd generally move the workload to something like **Glue/Spark** or another dedicated processing system.
	- The broader principle is: Firehose is primarily a delivery service with lightweight transformation capabilities, not a general-purpose distributed data-processing engine.

## Failure Handling

- Suppose you have:
	```
	Application
	    ↓
	Firehose
	    ↓
	S3
	```
- Suddenly, the S3 destination becomes temporarily unavailable. You **don't want the streaming records to simply disappear** while S3 is unavailable.
- What would you expect Firehose to do in this situation, and why is this different from simply having your application write directly to S3 for every event?
> 	I would expect Firehose to write the batch to a backup location or persist the data for a reasonable amount of time. This is different than having the application write directly to S3 because the application doesn't have to worry about the write being successful. That becomes Firehose's responsibility.
	- **Firehose takes responsibility for reliable delivery**, so the producer doesn't need to implement its own buffering, retry, and delivery logic around every S3 write.
	- Firehose **doesn't necessarily write to a backup location when delivery fails**. The exact failure behavior depends on the destination and configuration. The important concept is that Firehose **buffers and retries delivery**, rather than immediately losing records when the destination has a transient failure.

# Database Migration Service (DMS)

## Overview

- **Scenario**: A company has a production **PostgreSQL** database running on-premises. They want to migrate it to **Amazon RDS for PostgreSQL**. The database is large, and the application **cannot afford to be offline for several hours** while the migration occurs.
- How would you approach this migration using **AWS Database Migration Service (DMS)**?
> 	I would choose a recent checkpoint in the PostgreSQL database as a point from which I'd perform an initial full load into RDS. Once the initial full load is complete, I'd use the database transaction logs to perform CDC and incrementally load data from the initial checkpoint. Once RDS has caught up with the PostgreSQL database and both databases are confirmed to contain the same data, I would cutover to RDS.
	- This is the standard **full load + CDC** migration pattern.
	- The important part is that **CDC captures changes occurring while the initial load is running**. That prevents changes made during the potentially long full-load process from being lost.
	- DMS needs to establish the appropriate **source position/transaction-log position** so changes aren't missed or duplicated between the full load and CDC phases. You don't simply "choose a recent checkpoint."
	- The final step is arguably the most important: Don't cut over merely because CDC is running. Validate that the target has caught up and that the source and target data are consistent.

## Full Load vs. Change Data Capture (CDC)

- Suppose the initial full load takes **6 hours**. During those 6 hours, the application continues writing to PostgreSQL:
	```
	10:00 ─────────────────────────────── 16:00
	       Full load running
	
	Writes:
	10:30 → Order A updated
	11:45 → Order B inserted
	13:20 → Order C updated
	15:50 → Order D inserted
	```
- If you performed **only the full load**, the RDS database would reflect the source as it existed when the full load captured the relevant data—not necessarily the current state at 16:00.
- What role does **CDC** play during this six-hour full load? Why is it important that DMS starts CDC from an appropriate position so that changes aren't missed between the initial load and CDC processing?
> 	CDC acts as a sort of "bookmark" in the transaction log that tells DMS where to start performing incremental loads using CDC once the initial full load is complete. It's important that DMS starts CDC from an appropriate position so that no changes are missed or duplicated between the initial load and CDC processing.
	- The key to choosing an appropriate position is to avoid a **gap**. During the full load, changes continue occurring. DMS needs to retain/process those changes and then apply them to the target in the **correct sequence**.
	- Choosing the wrong CDC position could result in:
		- **Missing changes** → target becomes inconsistent with source.
		- **Duplicate changes** → the same logical change gets applied twice.
	- This is another place where **idempotency and data validation** become important.

## CDC Lag

- Your migration has been running successfully:
	- Full Load: Complete
	- CDC: Running
- However, you notice the following:
	```
	Source DB current position:  15:00
	DMS CDC position:            14:20
	```
	- The gap continues increasing. The application is still actively writing to the source database.
- What does this indicate, and what would you investigate to determine **why DMS is falling behind**?
> 	DMS could be falling behind because it is not properly scaled and/or because RDS is not properly scaled. If DMS is not properly scaled, it won't be able to efficiently read the transaction logs and write the changes to RDS. If RDS is not properly scaled, it would act as a bottleneck for DMS even if it was processing events from the transaction log efficiently.
	- If **DMS itself** can't keep up, the CDC position falls behind because DMS isn't reading/processing the source changes quickly enough.
	- If **RDS** is the bottleneck, DMS may be reading changes efficiently but can't apply them to the target quickly enough.
	- To determine where the bottleneck is, the following should be investigated:
		- **DMS replication instance** → CPU, memory, storage, network utilization.
		- **CDC source latency** → is DMS falling behind while reading PostgreSQL's transaction logs?
		- **Target latency** → are changes piling up because RDS writes are slow?
		- **RDS metrics** → CPU, I/O, connections, storage, and other signs of resource pressure.
		- **DMS task metrics/logs** → errors, throughput, and CDC latency.
	- Don't assume the replication service is the bottleneck just because replication is behind. Trace the entire pipeline to find where throughput is being constrained.

## Data Validation

- Suppose CDC has caught up and you're preparing to cutover the application:
	```
	Source → DMS → RDS
	             ↓
	        CDC lag ≈ 0
	```
- However, the migration team says: "DMS reports that the migration completed successfully, so we're ready to switch over."
- Would you consider **zero CDC lag** sufficient evidence that the migration is correct? What kinds of validation would you perform before cutting over to RDS?
> 	I would not consider zero CDC lag as sufficient evidence that the migration was successful. I would also confirm the data in both databases actually match by comparing row counts and checksums.
	- Validation can occur at multiple levels, including:
		- **Row counts** → catch missing or extra records.
		- **Checksums/hashes** → detect differences in actual data.
		- **Representative record comparisons** → verify important tables/records.
		- **Schema validation** → confirm tables, columns, types, indexes, constraints, etc. are consistent where required.
		- **Application-level validation** → ideally verify that the application can perform its critical operations correctly against RDS.

## Schema Changes

- Your production PostgreSQL database has 200+ tables. While the migration is running, developers occasionally make schema changes, such as:
	- `ADD COLUMN`
	- `ALTER COLUMN`
	- `CREATE TABLE`
	- `DROP COLUMN`
- The target RDS database needs to remain compatible with these changes.
- Why can **schema evolution** be challenging during a DMS migration? What would you want to consider when designing the migration so that a schema change doesn't cause the CDC process or target database to become inconsistent?
> 	Schema evolution during a DMS migration can be challenging because certain schema changes can require a lot time and resources to properly execute, which could cause CDC lag to increase and cause the source and target databases to become inconsistent. Schema changes can also be challenging because you'd need to consider whether the change constitutes a breaking or non-breaking change. For example, dropping, renaming, or changing a column's data type could represent a breaking change, while adding an optional column would not necessarily represent a breaking change.
	- Schema changes need to be coordinated between the source, DMS task, target schema, and **consuming application**.

| Change                              | Potential impact     |
| ----------------------------------- | -------------------- |
| Add optional column                 | Usually non-breaking |
| Add required column without default | Potentially breaking |
| Rename column                       | Breaking             |
| Drop column                         | Breaking             |
| Change data type                    | Potentially breaking |

# Lambda

## Overview

### Design Scenario
- Your application allows users to upload profile pictures to S3. Whenever an image is uploaded, you need to:
	1. Resize it.
	2. Generate a thumbnail.
	3. Store the results back in S3.
- The workload is highly unpredictable:
	- Normal: 5 images/sec
	- Occasional Spikes: 5,000 images/sec
- The processing of each image takes about 2 seconds.
- **Would Lambda be a good fit?** Explain why or why not.
	- Think about:
		- Server management
		- Scaling
		- Workload predictability
		- Processing duration
		- Whether you need the compute resources running continuously
> 	Lambda would be a good fit because the unpredictable nature of the workload would make server management, especially manually up and downscaling, very complicated and time consuming. Since each image only takes about 2 seconds to process, it would be unlikely for you to hit any performance limitations. Compute resources would only need to run while images are being processed and could automatically shut down after an appropriate idle period.
			- Lambda provisions the execution environment as needed, so the application doesn't need to manage continuously running servers.
			- The fact that each invocation only takes 2 seconds doesn't automatically mean the system can handle **5,000 images/sec**without constraints. Lambda also has **concurrency limitations**.
			- For example, if you suddenly receive thousands of events, Lambda may need to execute thousands of concurrent invocations. If concurrency exceeds the applicable limit, requests can be throttled.
			- Furthermore, the fact that Lambda can scale quickly doesn't matter if downstream dependencies can't handle the traffic spike.
- Now suppose you have a nightly data-processing job. Every night:
	```
	S3
	 ↓
	Process 500 GB of data
	 ↓
	Transform
	 ↓
	Write results
	```
- The processing takes 45 minutes and the job only needs to run once a day.
- **Would you use Lambda for this? If not, what characteristics of the workload make Lambda less attractive**?
> 	I would only used Lambda if the data could be effectively partitioned and processed concurrently by multiple Lambdas. If this was not possible, Lambda would not be a good choice because of its 15-minute execution time limit.
	- 500 GB is a substantial amount of data, so even if you _can_ partition it, Lambda may not be the best tool.
	- We'd need to consider:
		- Number of partitions/invocations
		- Data movement
		- Coordination between functions
		- Memory and execution limits
		- Cost
		- Whether a distributed processing engine would be more efficient
	- That's where something like **AWS Glue/Spark** can become a better fit.

### Mental Model
- That's where something like **AWS Glue/Spark** can become a better fit.
- For example:
	```
	S3 upload | SQS message | API request
	   ↓      |       ↓     |        ↓
	Lambda    |    Lambda   |     Lambda
	```
- Be more cautious when you see:
	- Hours of computation.
	- Large-scale distributed processing.
	- Long-running **stateful** workloads.
	- Specialized compute requirements.

## Lambda + SQS

- Suppose we have:
	```
	Order Service
	      ↓
	     SQS
	      ↓
	   Lambda
	```
- The Lambda function processes each order. Suddenly, the queue receives 100,000 messages.
- **Would you manually launch more Lambda instances to process the backlog? What potential problem could occur if Lambda scales up very aggressively and every invocation makes a request to DynamoDB**?
> 	SQS can act as a durable storage layer for messages that are waiting to be processed as the Lambda function processes the load. You could horizontally scale to process messages in the queue more quickly, but only up to a certain point. Once the read/write capacity of the DynamoDB table is reached, it becomes a bottleneck.
	- SQS absorbs the backlog; Lambda scales consumers to process it; DynamoDB ultimately limits how far you can scale.
	- If 100,000 messages arrive, you don't need to provision 100,000 servers yourself. Lambda can **increase concurrency** to process the queue.
	- Lambda concurrency is roughly the number of invocations executing simultaneously.
	- Lambda can increase concurrency automatically, or you can manually control concurrency to avoid overwhelming downstream dependencies.
- **Backpressure**:
	- If the downstream system can't process work as quickly as Lambda can generate it, you need some mechanism to prevent the upstream system from overwhelming it.
	- SQS naturally helps here because the backlog can remain in the queue. Instead of forcing Lambda to process everything immediately, you can deliberately limit concurrency and let SQS temporarily accumulate messages.
	- The one major tradeoff is higher queue latency in exchange for protecting the downstream system.
- As with any other SQS consumer, it's important to make Lambda invocation functions idempotent, incase the Lambda crashes after processing the message, but before deleting it.

## Cold Starts & Provisioned Concurrency

- Suppose you have an API backed by Lambda. Normally, it receives 100 requests/sec. Once every morning at 9:00 AM, traffic spikes to 20,000 requests/sec.
- The API has a strict P99 latency SLA of 200 ms.
- Some Lambda invocations experience significantly higher latency when a new execution environment needs to be initialized.
- What could cause that additional latency, and how could you design the system to reduce it during the predictable 9 AM spike?
> 	The additional latency could be associated with server startup time. This could be prevented by provisioning concurrency ahead of time, so that's fully prepared to absorb the traffic load.
- When Lambda needs a new execution environment, there can be initialization overhead before your function actually starts processing the request. This is called the **cold start** problem.
- Provisioning concurrency ahead of known traffic spikes helps avoid the cold start problem by allowing Lambda to initialize the execution environment before it's actually needed.
- **Provisioned concurrency is not the same thing as increasing Lambda's concurrency limit**. They solve different problems.
	- Concurrency Limit: How many executions am I allowed to run simultaneously?
	- Provisioned Concurrency: How many execution environments should already be initialized and ready to respond?

## Lambda Retries & Failure Handling

- A Lambda function processes an SNS event. The function fails because a downstream API is temporarily unavailable.
- **What should happen?** Should Lambda
	- Immediately give up?
	- Retry the invocation?
	- Keep retrying forever?
	- Put the failed event somewhere for later investigation?
- **What would you need to consider to make sure retries don't create duplicate side effects**?
> 	Lambda should retry an appropriate number of times before sending the message to a durable storage layer, such as a DLQ. To ensure retries don't produce duplicate effects, I'd ensure the processing logic in the invocation function is idempotent.
- The exact retry/DLQ behavior depends on **what invokes Lambda**.
- For example, Lambda's behavior differs between:
	- SNS → Lambda
	- SQS → Lambda
	- EventBridge → Lambda
	- Direct synchronous invocation
- 

## Summary

| Concept                 | Key takeaway                                  |
| ----------------------- | --------------------------------------------- |
| Serverless              | No server management                          |
| Short-lived workloads   | Good Lambda use case                          |
| 15-minute limit         | Long-running work may need another solution   |
| Concurrency             | Number of simultaneous executions             |
| Downstream bottlenecks  | More Lambda ≠ unlimited scalability           |
| SQS integration         | Queue absorbs backlog                         |
| Idempotency             | Protect against duplicate processing          |
| Visibility timeout      | Prevent premature redelivery                  |
| Cold starts             | Initialization adds latency                   |
| Provisioned Concurrency | Keep environments initialized ahead of demand |

# Glue

## Overview

- The most important thing to understand about Glue is that it's primarily a **managed data integration / ETL service**. A common architecture looks like:
	```
	S3
	 ↓
	Glue Job
	 ↓
	Transform / Clean / Join
	 ↓
	S3 / Redshift / other destination
	```
- Glue is especially useful when you're dealing with **batch-oriented data processing at scale** and don't want to manage your own Spark cluster.
- Under the hood, Glue ETL jobs commonly use **Apache Spark**.
- **Scenario 1 - Small Dataset**:
	- Imagine you have an ETL job that runs once a night:
		```
		S3
		 ↓
		5 GB of Parquet data
		 ↓
		Transform a few columns
		 ↓
		Write result to S3
		```
	- **Would you use AWS Glue for this? Why or why not**?
> 	Although the dataset is 5GB, if the transformations that need to be run are fairly simple and can be processed very quickly, Lambda could be a good fit.
		- The **5 GB size alone doesn't mean Glue is necessary**. The more important questions are:
			- How complex is the transformation?
			- How quickly does it need to run?
			- Can it be handled by a single process?
			- Is distributed processing actually providing value?
			- What are the startup and compute costs?
		- If the transformations are simple enough to process efficiently without distributed compute, Glue could be unnecessary overhead.
		- Lambda _could_ be an option if the processing fits within Lambda's execution, memory, and other limits.
		- **The decision to use Lambda shouldn't be based soley on "5GB is small."** Five GB of data could still justify distributed processing if, for example, the **transformation involves expensive joins or aggregations**.
- **Scenario 2 - Large Dataset**:
	- Now imagine **500 GB of Parquet data** stored in S3. Every night, you need to:
		1. Read all 500 GB.
		2. Join it with another **100 GB dataset**.
		3. Perform several aggregations.
		4. Remove invalid records.
		5. Write the results back to S3 as **partitioned** parquet.
	- The job currently takes about **20 minutes** when run on a Spark cluster.
	- **Would you choose Lambda or Glue? What characteristics of this workload make one more appropriate than the other**?
> 	The primary characteristics of the workload that make Glue a more appropriate choice than Lambda are the volume of data and complexity of transformations that need to be performed. Spark's distributed processing framework is well-suited for this kind of workload.

### Glue Job
- A Glue Job is the **compute** that actually performs ETL transformations.
- For example:
	```
	S3
	 ↓
	Glue Job
	 ↓
	Transform
	 ↓
	S3
	```
- A Glue ETL job commonly runs on Spark code.

### Glue Data Catalog
- The Glue Data Catalog is the **metadata repository** describing your data.
- For example:
	```
	Database: sales
	
	Table: orders
	
	Columns:
	    order_id
	    customer_id
	    order_date
	    amount
	
	Location:
	    s3://bucket/orders/
	```
- The Catalog **doesn't contain** the actual 500 GB of data. It contains information **about** the data.

### Glue Crawler
- A crawler can inspect data sources and infer things such as:
	- Schema
	- Columns
	- Data types
	- Partitions
- It can then populate/update the Glue Data Catalog.
- Conceptually:
	```
	S3
	 ↓
	Crawler
	 ↓
	Glue Data Catalog
	 ↓
	Athena / Glue / other services
	```
- **Interview Scenario**:
	- Suppose you have this S3 dataset:
		```
		s3://sales/orders/
		    year=2025/
		        month=01/
		        month=02/
		        ...
		    year=2026/
		        month=01/
		        month=02/
		        ...
		```
	- An analyst wants to query it using Athena.
	- **What role could the Glue Data Catalog play here? What role would a Glue Crawler play**?
> 	The Glue Crawler inspects the S3 dataset and produces metadata, which is stored in the Glue Data Catalog and used by Athena or other serves to query the data.

## Glue Performance

- Suppose you have a Glue Spark job processing **2 TB of Parquet data**. You look at the Spark UI and see:
	```
	Executor 1: ████████████████████ 95% CPU
	Executor 2: ██                   10% CPU
	Executor 3: ██                   8% CPU
	Executor 4: █                    5% CPU
	...
	```
- The job is taking much longer than expected.
- **What does this pattern suggest to you? What would you investigate to determine why one executor is doing dramatically more work than the others**?
> 	I would look to see how the data is partitioned and compare it to the data access patterns within the Glue Job. The fact that one executor is performing significantly more work than the others suggests a hot partition.
	- The key issue isn't necessarily that you don't have enough executors. You may have plenty of compute available, but **the work isn't distributed evenly across them**.
	- What to investigate:
		- How the data is partitioned.
		- The distribution of records across partitions.
		- The partition key and its cardinality.
		- Whether one or a few key values account for a disproportionate amount of the data.
		- Whether a particular join or aggregation is causing the skew.
- Now suppose the Spark UI instead shows:
	```
	Executor 1: ████████████████ 90%
		Executor 2: ████████████████ 92%
	Executor 3: ████████████████ 89%
	Executor 4: ████████████████ 91%
	...
	Executor 50: ███████████████  88%
	```
- **What does this suggest? Would you investigate data skew, or would you start looking somewhere else**?
> 	This suggests the data load is evenly distributed, but nearing the performance limitations of the current Spark cluster. Before scaling, I'd look at the Spark UI to try and identify the bottleneck, then consider optimizations such as filtering before aggregations or performing broadcast joins for relatively small tables.
	- If every executor is similarly busy, **data skew is much less likely**. The cluster is actually receiving work fairly evenly, so I'd investigate whether the workload itself is computationally expensive or whether there are Spark-level inefficiencies.
	- Troubleshooting sequence:
		- **Inspect the Spark UI** to identify where the time is being spent.
		- Look for expensive stages, excessive shuffling, large joins, etc.
		- Optimize the job before simply throwing more compute at it.
		- Scale the cluster if the workload genuinely requires more resources.
- Now imagine your Glue Job processes **1 TB of data** stored in S3. The job is taking much longer than expected.
- You inspect the S3 layout and discover that there are 1,000,000 files with an average size of 1 MB. The actual amount of data is not particularly large for the Spark cluster, and the transformations themselves are straightforward.
- **What problem does this S3 layout create? What would you do to improve the performance of the Glue job**?
> 	This S3 layout creates the small files problem, where Spark expends more resources scheduling and orchestrating tasks than actually performing tasks. Improving the performance of the Glue job could involve repartitioning the data so that it is spread across fewer files, or compacting the files if the partitioning strategy can't be changed.
	- The two core solutions are correct:
		- **Repartition** the data when you have control over how the output is written.
		- **Compact existing files** when the underlying partitioning strategy is otherwise appropriate.
	- The goal is generally to have **reasonably sized files** rather than either too many small files or too few large files. Too many small files creates excessive metadata and scheduling overhead, while too few files leads to insufficient parallelism.

### Worker Sizing
- Let's say you've optimized the job:
	- No major data skew
	- No small-files problem
	- Filters are pushed early
	- Appropriate broadcast joins are being used
	- The workload is still legitimately compute-intensive
- Your Glue job is currently configured with **10 workers**, and the job takes 60 minutes.
- You increase it to **20 workers**, and the job takes 32 minutes.
- You increase it to **40 workers**, and it takes 18 minutes.
- You increase it to **80 workers**, and it takes 17 minutes.
- **What does this tell you? Would you continue increasing the number of workers? might doubling the compute resources eventually produce little or no additional performance improvement**?
> 	This tells me that the job is benefiting from increased parallelism associated with a larger cluster size, up to a certain point. The trend shows diminishing returns, where doubling cluster size the first time cuts execution time by almost 30 minutes, while doubling it a second time only reduces execution time by less than 15 minutes. Continuing this trend indefinitely would result in very few tasks being assigned to each worker, meaning individual workers would be underutilized during the execution of the job.

## Glue vs. Lambda

- **Lambda**: Best suited for:
	- Short-lived processing
	- Event-driven workloads
	- Small/independently processable units of work
	- Highly variable workloads
- **Glue**: Best suited for:
	- Large datasets
	- Batch ETL
	- Complex transformations
	- Joins and aggregations
	- Distributed Spark processing

## Glue vs. Athena

- Imagine you have data in S3:
	```
	S3
	 ↓
	Parquet
	 ↓
	Several TB
	```
- An analyst asks: "Give me the total sales for each month in 2026."
- There is **no need to transform or persist the data**. They simply need to query it.
- **Would you use a Glue Job or Athena**?
> 	Since there is no need to transform the data and it is already stored in Parquet format, Athena would be appropriate since it is designed for querying processed data, while Glue is designed to transform unprocessed data.
	- Glue can absolutely transform **already processed/structured data**.
	- A more precise distinction is that Glue is primarily for **data integration and transformation**, while Athena is primarily for **interactive SQL querying** of data stored in S3.
- Now imagine you have **10 TB of Parquet data** in S3. An analyst runs:
	```sql
	SELECT
	    customer_id,
	    SUM(amount)
	FROM sales
	WHERE year = 2026
	GROUP BY customer_id;
	```
- The table contains data from **2018 through 2026**, and the data is physically partitioned by `year` and `month`.
- **What could you do to make this Athena query more efficient and reduce the amount of data Athena needs to scan**?
> 	If analysts commonly filter by year instead of month, query performance could be improved by partitioning the data by year.
	- In this case, the data is **already partitioned** by `year`, so Athena would use **partition pruning** to avoid scanning irrelevant partitions.
- Now, suppose the `sales` table has **100 columns**, but the analyst only needs `customer_id`, `amount`, and `year`.
- Athena is querying **Parquet** files.
- **Why does using Parquet help Athena here, and why would this query generally be more efficient than querying the same data stored as CSV**?
> 	Unlike CSV, Parquet is column-based storage format, meaning only the specific columns being queried need to be scanned during execution.
	- Partition pruning and column pruning work together to reduce the amount of data that needs to be scanned when data is stored in Parquet format.
	- Parquet also supports **compression and efficient encoding**, which can further reduce the amount of data Athena needs to read.
- Finally, suppose you have a **5 TB events dataset** in S3.
- The current S3 storage layout stores all 1,000,000 small **CSV** files under one `events` partition.
- Analysts primarily query by `event_date` and `application_name`. The queries are becoming expensive and slow.
- **If you were responsible for improving this data platform, what changes would you consider to the storage layout and query architecture**?
> 	Since the queries are becoming slow and expensive with the data stored in CSV format, I'd consider switching to Parquet format, as well as partitioning the data based on application_name and event_date, given that is primarily how analysts filter the data when performing queries. Using this partitioning data allows unrelated data to be skipped while scanning data. Parquet also offers efficient compression and encoding, further reducing the amount of data that needs to be scanned.
	- Also mention **file compaction**. You don't want to simply convert 1,000,000 CSV files to 1,000,000 Parquet files.
	- You might not want to consider partitioning by `application_name`, depending on the cardinality of the column and how evenly it would distribute data across partitions.

# Athena

## Overview

- Athena is a serverless **query engine** that lets you run SQL directly against data stored in S3. You don't need to load the data into a database first.
- Athena is particularly useful when:
	- Data is already in **S3**
	- Query volume is relatively **low or unpredictable**
	- You don't need a **continuously running** database
	- You want minimal infrastructure management
	- You're primarily performing **analytical** queries
- **Parquet and partitioning can dramatically reduce the amount of data Athena needs to scan**.

## Athena vs. Redshift

- Your company has:
	- **10 TB of historical event data**
	- Data already stored in **Parquet on S3**
	- Analysts run approximately **10 moderately complex queries per day**
	- Queries don't require extremely low latency
	- Query demand may increase somewhat in the future
- Would you initially choose **Athena or Redshift**? What factors would you consider before deciding that Redshift is worth introducing?
> 	I would likely choose Athena because the data is already stored in S3 in Parquet format. Assuming the data is effectively partitioned, this would help reduce the amount of data Athena is required to scan during a query. Furthermore, the queries are moderately complex and don't require extremely low latency. The somewhat unpredictable nature of query demand also makes Athena a better fit.
	- **Data is already in S3** → Athena can query it directly.
	- **Parquet** → column pruning and compression can reduce scanned data.
	- **Effective partitioning** → partition pruning can further reduce the scan.
	- **Moderate query complexity** → Athena is capable of handling it.
	- **No strict low-latency requirement** → there's less justification for a dedicated warehouse.
	- **Unpredictable demand** → Athena's serverless, pay-per-query model can be attractive.
- Key Principle: Don't introduce a continuously running analytical warehouse unless the workload actually benefits enough from its performance to justify the additional cost and operational considerations.

## Query Cost

- Imagine you have a **5 TB Parquet dataset** in S3. The data is partitioned by year, month, and day. An analyst runs this query:
	```sql
	SELECT customer_id, COUNT(*)
	FROM events
	WHERE year = 2026
	GROUP BY customer_id;
	```
- Why would the partitioning strategy significantly affect the cost and performance of this Athena query? And what would happen if the data **wasn't partitioned at all**?
> 	The partitioning strategy affects query cost and performance because it determines how much data needs to be scanned in order to execute the query. If the data wasn't partitioned at all, Athena would need to query a much larger portion of the data.
	- Because the query filters on `year = 2026`, Athena can skip the partitions for other years instead of scanning the entire dataset.
	- That improves both **query performance and cost**, because Athena's pricing is based largely on the **amount of data scanned**.
	- Without partitioning, Athena would have to scan the **relevant columns** across the much larger dataset to determine which rows satisfy `year = 2026`.
	- Partitioning eliminates entire partitions from the scan, while Parquet columnar storage eliminates unnecessary columns from the scan.

## Partition Design

- Suppose you have **10 TB of event data** with these columns:
	- `event_date`
	- `application_name`
	- `user_id`
	- `event_type`
	- `region`
	- `payload`
- Analysts commonly run queries such as:
	```sql
	WHERE event_date BETWEEN ...
	  AND application_name = ...
	```
	- `region` is only occasionally used as a filter. You need to decide how to partition the data.
- What partitioning strategy would you choose, and why? What problem could arise if you partitioned by **`user_id`** instead?
> 	I would partition by event_date and application_name because this is the most common query pattern. Partitioning by user_id could create an excessive amount of small partitions, leading to an excessive amount of scanning and metadata overhead.
	- Small partitions don't necessarily lead to "excessive scanning." The bigger concern is the **small-files / partition-overhead problem** and the possibility that the partition layout doesn't provide enough benefit relative to its management cost.

## Parquet vs. Partitioning

- Suppose your data looks like:
	```
	10 TB total
	↓
	Partitioned by event_date
	↓
	Stored as Parquet
	```
- An analyst typically runs:
	```sql
	SELECT user_id
	FROM events
	WHERE event_date = '2026-08-27';
	```
- Explain **two different ways** the combination of partitioning and Parquet makes this query more efficient.
> 	Partition pruning eliminates scanning event_date partitions that are irrelevant to the query. Column pruning eliminates scanning irrelevant columns within a partition. Storing files in Parquet format, combined with an appropriate partitioning strategy based on common access patterns reduces the amount of irrelevant data scanned during query execution.
- **Scenario: Partitioning Gone Wrong**:
	- Suppose you have a dataset:
		```
		S3
		└── events/
		    ├── application_name=A/
		    │   ├── year=2026/
		    │   │   ├── month=01/
		    │   │   │   ├── day=01/
		    │   │   │   │   ├── file1.parquet
		    │   │   │   │   ├── file2.parquet
		    │   │   │   │   ├── file3.parquet
		    │   │   │   │   └── ...
		```
	- After several months, the data lake contains **millions of very small Parquet files**.
	- Analysts complain that Athena queries are becoming slower even though they're scanning relatively little data.
- Why could having millions of small Parquet files hurt Athena query performance? What would you consider doing to improve the situation?
> 	Having millions of small Parquet files hurts query performance because there are costs associated with opening and closing each file while scanning relevant data. Small partitions also create a lot of metadata. To mitigate these issues, I'd either consider changing the partitioning strategy or compacting files to produce more appropriately-sized files.
	- **File-management overhead:** Athena/Spark has to deal with many individual files, so the overhead of opening, scheduling, and coordinating them can become significant relative to the amount of actual data being processed.
	- **Metadata overhead:** Millions of files create substantially more metadata to manage and inspect. This metadata needs to be inspected when producing the query's execution plan.
	- Compact the existing files into fewer, appropriately sized Parquet files would be a more effective, immediately solution than repartitioning.
	- After compacting, The pipeline should be inspected to determine why it is producing so many small files in the first place. If the partitioning strategy is causing it, redesigning the partitioning scheme could help, so future writes produce healthier file sizes.

## Query Optimization

- An analyst tells you: "This Athena query used to take 10 seconds, but now it takes 3 minutes." The query scans data from S3 and hasn't changed.
- Walk me through how you would investigate why the Athena query became slower.
> 	I'd investigate how the data is stored first. If the data is stored as JSON or CSV instead of Parquet, I'd consider switching to Parquet if visual inspection wasn't a major requirement. This would allow for optimizations such as column pruning and predicate pushdown. Next, I'd look at the partitioning strategy and compare it against the query's access pattern. If the two are misaligned, I'd consider changing the partitioning strategy or asking the analyst if their query could be modified to match the existing pattern. This would allow for better partition pruning. I'd also look at the query's execution plan to see if the query could be optimized in any way, such as filtering early. Next, I'd look at file sizes. If each partition contains an excessive number of small files, I'd consider compaction as a solution.
	- **Predicate pushdown** is more accurately associated with the storage engine/file format being able to avoid reading rows that don't satisfy predicates; **partition pruning** is what skips entire partitions.
- **Scenario: Cost Explosion**:
	- Your company has an Athena query that analysts run regularly.
	- Yestarday:
		- Dats Scanned: 200 GB
		- Cost: low
		- Runtime: 15 seconds
	- Today:
		- Data Scanned: 4 TB
		- Cost: significantly higher
		- Runtime: 4 minutes
	- The SQL query hasn't changed. Yesterday's query contained: `WHERE event_date = '2026-08-26'`, but today's version doesn't include the `event_date` filter.
- Why would removing that filter have such a dramatic effect on both **cost and performance**, and what Athena/S3 design principle does this demonstrate?
> 	Removing the filter would have a dramatic effect on query cost and performance because it effectively eliminates the benefit of partition pruning, assuming there aren't any other filters in the query. This demonstrates the need to align S3 partitioning strategy with access patterns used in Athena.
	- Partitioning only provides a major benefit when queries **actually filter on the partition columns**.
	- When designing an S3 data lake for Athena, you need to consider **how analysts will actually query the data**, not just how the data happens to be structured.

## Athena vs. Glue

- You have raw data arriving in S3:
	```
	CSV files
	  ↓
	Need to:
	- clean malformed records
	- join with customer data
	- aggregate transactions
	- convert to Parquet
	  ↓
	S3
	  ↓
	Athena
	```
- Why would you use **Glue before Athena** here rather than having Athena perform all of the work directly? What role would Athena play **after** the Glue job completes?
> 	You would use Glue before Athena because the required transformations are fairly complex. Glue primarily uses Spark under the hood to perform data transformations. Taking advantage of Spark's distributed processing framework would allow the transformations to be executed much more effectively than using SQL statements. After Glue transforms the raw CSV data and coverts it to Parquet format, Athena would simply act as a query engine. Glue can also scan the processed data and provide metadata that would help Athena optimize queries.
	- **Glue is doing the transformation work**, while **Athena is doing the interactive querying**.
	- Glue Data Catalog provides metadata about the data that Athena can use to **understand and query** the datasets.
	- The catalog itself doesn't necessarily "optimize" Athena's queries; **partitioning, columnar formats, and query predicates** are what provide the major scan optimizations.

# Step Functions

## Overview

- The simplest way to think about Step Functions is that it is a tool used to orchestrate workflows made up of multiple steps.
- Suppose order processing involves:
	```
	Validate Order
	      ↓
	Charge Payment
	      ↓
	Reserve Inventory
	      ↓
	Create Shipment
	      ↓
	Send Confirmation
	```
- You *could* put all of the logic into a Lambda Function:
	```
	Order Lambda
	 ├─ Validate
	 ├─ Charge
	 ├─ Reserve
	 ├─ Ship
	 └─ Notify
	```
	- This creates a large, tightly-coupled function.
- Instead, Step Functions can orchestrate separate, independent components:
	```
	             Step Functions
	                   │
	        ┌──────────┼──────────┐
	        ↓          ↓          ↓
	    Validate    Payment    Inventory
	        ↓          ↓          ↓
	              Shipment
	                   ↓
	              Notification
	```
- The state machine keeps track of **where the workflow is**, allowing individual steps to succeed, fail, retry, or branch independently.
- **Scenario**: You're building a data pipeline:
	1. Extract data from API
	2. Validate the data
	3. Transform the data
	4. Load it into Redshift
	5. Send a notification when complete
- The transformation step fails occasionally because the external API sometimes returns malformed records.
- **Why might Step Functions be preferable to putting the entire workflow into a single Lambda function**?
> 	Step Functions would be preferable to a single Lambda function because each step can be turned into a task, where each task is a part of a larger workflow. This makes failure isolation and retries significantly easier to manage. When one step fails, you know exactly which one failed. Retrying the step doesn't involve retrying any of the other steps, avoiding unnecessary repeated work. Lambda functions also offer automatic retries, but the entire workflow would need to be retried, which would be especially wasteful if only the last step failed.
	- When a Lambda function is retried, the entire workflow isn't necessarily retried. It depends on the Lambda's invocation source.

## Sequential vs. Parallel Work

- Step Functions becomes particularly useful when a workflow isn't simply a straight line.
- Suppose you're validating an order. You need to:
	```
	             Validate
	                ↓
	        ┌───────┴───────┐
	        ↓               ↓
	   Charge Payment   Reserve Inventory
	        ↓               ↓
	        └───────┬───────┘
	                ↓
	          Create Shipment
	```
- Payment and inventory reservation don't depend on each other, so they can potentially happen **in parallel**. Step Functions supports this kind of branching.
- **Why might running these two tasks in parallel be preferable to running them sequentially? What would you need to consider before deciding that the two operations are safe to execute concurrently**?
> 	Running the two tasks in parallel is preferable to running them sequentially because concurrent execution is faster. Before deciding if concurrent execution is appropriate, you'd need to evaluate dependency between the tasks and whether they use shared resources.
	- Dependencies: If inventory reservation requires successful payment authorization, you can't safely run them independently.
	- Shared Resources / Concurrency: If both operations modify the same resource, concurrent execution could introduce race conditions or inconsistent state.
	- Failure Semantics: You also need to consider what happens if one succeeds and the other fails. For example, if `Charge Payment` succeeds, but `Reserve Inventory` fails, there would need to be a way to identify and correct the inconsistent state. This is why **error handling and retries** are important considerations when designing a Step Functions workflow.

## Retry vs. Catch

- Suppose your workflow looks like:
	```
	Validate
	   ↓
	Charge Payment
	   ↓
	Reserve Inventory
	   ↓
	Create Shipment
	```
	- The `Reserve Inventory` task occasionally fails because DynamoDB experiences a temporary throttling event.
	- You **don't** want the entire workflow to fail immediately. A **transient failure** may succeed a few seconds later.
- **How would you design the Step Function to handle this**?
> 	I would design the Step Function to retry the task an appropriate number of times. I'd also ensure the task itself is idempotent so that inventory can't be reserved more than once for the same order. After a set amount of retries, the failure can be assumed permanent rather than transient. Once the task has permanently failed, it should trigger a separate task that issues a refund.
	- The configuration of the retry policy is also important. Ideally, you'd use **exponential backoff** to avoid overwhelming a downstream service.
	- You can't necessarily roll back a payment and inventory reservation as one atomic transaction across independent services, you may need to perform a compensating operation when a later step fails.

## Step Functions vs. SQS

- Suppose you have an image-processing system:
	```
	Upload Image
	     ↓
	Process Image
	     ↓
	Generate Thumbnail
	     ↓
	Extract Metadata
	     ↓
	Notify User
	```
	- You have **millions of images**, and each image can be processed independently.
- **Would you use **Step Functions**, **SQS**, or potentially both**? Explain what role each service would play and why.
> 	I would use SQS as a durable storage location for image processing requests and Step Functions as an orchestration layer that executes independent tasks in parallel. Requests for workflows that failed could be sent to a DLQ so they can be inspected and retried.
	- Step Functions is appropriate when you actually have a **workflow with multiple dependent or coordinated steps**. It can orchestrate those operations, including retries, branching, parallel execution, and compensation.
	- Step Functions **isn't primarily a replacement for a queue** when you have millions of independent jobs. If every image is completely independent, SQS + workers/Lambda may be sufficient.
	- You'd only introduce Step Functions when **each image has a multi-step workflow that needs orchestration**.

## Step Functions vs. SNS

- **Scenario**: Suppose an order is successfully created. Three independent services need to know about:
	- Inventory
	- Email
	- Analytics
- The services don't need to execute in a particular order, and the order service shouldn't need to know about each downstream service.
- **Would you use Step Functions or SNS here? Why**?
> 	Since each service is independent and doesn't require orchestration, I'd use SNS. If message durability was a concern, I'd use an SNS + SQS fanout pattern so that each queue holds copies of the same messages so that they can be independently consumed by each downstream service.
	- Each downstream service gets its **own queue**, so it can:
		- Process messages independently
		- Consume at its own rate
		- Retry failures independently
		- Retain messages while temporarily offline
		- Use a DLQ for repeatedly failing messages
	- Most importantly, the Order service **doesn't need to know that these three consumers exist**.
	- Step Functions would be more appropriate if the **order itself had a workflow** with dependencies:
		- First, Validate Order
		- Then, Charge Payment
		- Then, Reserve Inventory
		- Finally, Create Shipment
	- That's an **orchestration**. SNS serves as an **event distribution** layer.
- **Mental Model**:
	- SNS: "Who needs to know about this event?"
	- Step Functions: "What steps need to happen, and in what order?"

## Step Functions vs. SNS vs. SQS

- **Scenario 1**: You have an order-processing system:
	- Requirement 1: When an order is created, **Inventory, Analytics, and Email** all need to receive an `OrderCreated` event.
	- Requirement 2: The inventory workflow is:
		1. Reserve Inventory
		2. Charge Payment
		3. Create Shipment
		- Payment **should only occur** if inventory reservation succeeds.
	- Requirement 3: The Analytics service may be offline for several hours and **must not lose events**.
	- Requirement 4: The Email service can process messages much more slowly than the other consumers.
- **Design the AWS messaging/orchestration architecture. Which of SNS, SQS, and Step Functions would you use, where would you use them, and why**?
> 	I would use an SNS + SQS fanout pattern. SQS queues would feed the Analytics and Email services since they operate independently. Retention policies for these queues would need to be set appropriately, according to service reliability and processing speed. Another SQS queue would feed a Step Functions workflow that orchestrates inventory reservation, payment, and shipment creation. Inventory reservation and payment would be setup to run sequentially, with payment depending on inventory reservation. Shipment creation could occur in parallel. If the sequential portion of the workflow fails, the shipment would be cancelled.
	- The first part is right:
		- **SNS** handles the one-to-many fanout.
		- **Separate SQS queues** give Analytics and Email independent buffering and consumption.
		- Analytics can be offline for hours because its queue retains the events.
		- Email can process slowly without blocking Analytics or Inventory.
	- Since the first two parts of the inventory workflow must be coordinated, it makes sense to only create the shipment once those two parts succeed together. It shouldn't be run in parallel.
	- You don't necessarily need an SQS queue in front of the Step Functions workflow. SQS is providing durable asynchronous buffering; Step Functions is orchestrating the multi-step order workflow. Don't add an SQS queue merely because Step Functions exists. Add it when you actually need **buffering, backpressure, independent consumption, or retry isolation**.

# Identity and Access Management (IAM)

## Overview

- You have a Lambda function that processes files from `s3://customer-data/`.
- The Lambda only needs to:
	- Read objects from `customer-data/incoming/`
	- Write processed objects to `customer-data/processed/`
- It does **not** need to:
	- Delete objects
	- Access other S3 buckets
	- Modify bucket configuration
	- Read objects from `customer-data/archive/`
- **How would you design the Lambda's IAM permissions? What does least privilege mean in this scenario**?
> 	I would design the Lambda's permissions policy such that it grants access to the specific S3 prefixes needed to read and write objects and only has permission to perform the needed actions on those buckets. For example, the function would only be given read permission to customer-data/incoming/ and write permission to customer-data/processed. It would not be given read and write permission to both buckets.
	- Designing a permissions policy using the principle of least privilege reduces the potential impact if the Lambda is compromised or contains a bug.
	- `customer-data/incoming/` and `customer-data/processed/` are S3 **prefixes**, not buckets. Be careful not to mix the two up during an interview.

## Identity-Based vs. Resource-Based Policies

- **Scenario**: Suppose you have Account A, which contains a Lambda function, and Account B, which contains an S3 bucket. The Lambda in **Account A** needs to read objects from an S3 bucket owned by **Account B**.
- **What additional IAM consideration does this introduce? Specifically, is giving the Lambda's execution role `s3:GetObject` permission necessarily enough for the cross-account access to work**?
> 	Since the two resources exist in different accounts, A resource-based policy must be created in Account B with the necessary S3 permissions and attached to an IAM role. Additionally, a trust policy must be created, granting the Lambda permission to assume that role. Giving Lambda's execution role s3:GetObject permission isn't enough for the cross-account access to work.
	- Instead of creating a trust policy, S3 supports a **bucket policy**, which can directly grant the Lambda's execution role permission to access objects in the bucket owned by Account B.
	- Using a bucket policy instead of a trust policy means the Lambda doesn't need to assume another role.
	- The configuration would look like:
		- Lambda's execution role → has the appropriate identity-based permissions.
		- Account B's S3 bucket policy → allows that role from Account A to access the specified objects.
	- Using a trust policy instead of a bucket policy would mean the Lambda would need to actually assume the role it's being granted permission to assume. In the two possible options, the bucket policy is a resource-based policy, while the trust policy is an identity-based policy.
	- **Important Takeaway**: **Resource-based policies**, such as an S3 bucket policy is attached to the **resource**, not the role.
- **Bucket Policy vs. IAM Policy**:
	- **Scope:** IAM policies control what an identity can access across multiple AWS services. Bucket policies control who can access one specific S3 bucket and its contents (what it can do with the bucket).
	- **Principal Element:** IAM policies do not need a `Principal` field because the user or role is already the target. Bucket policies must define a `Principal` to specify who gets access.
	- **Cross-Account Access:** Bucket policies are **mandatory** to allow external AWS accounts to access your S3 data.
	- How They Work Together:
		- **Same-Account Requests:** Access is granted if **either** the IAM policy or the bucket policy allows it (as long as there is no explicit deny).
		- **Cross-Account Requests:** Access requires an explicit **allow from both** the bucket policy in the resource account and the IAM policy in the client account.
		- **Deny Wins:** An explicit `Deny` in either policy blocks the request completely.

## Assigning Roles

- **Scenario**: You have a developer who needs to deploy a Lambda function.
	- The Lambda needs an execution role that allows it to:
		- Read from S3
		- Write to DynamoDB
		- Publish to SNS
	- The developer should be able to deploy and update the Lambda, but **should not be able to give the Lambda an arbitrary administrator role**.
	- **What IAM permission would you use to control which execution roles the developer is allowed to assign to the Lambda? Why is this important from a least-privilege perspective**?
> 	The `aim:PassRole` permission grants the ability to assign roles to IAM principals. This is important for least-privilege access because you don't want users bypassing restrictions set by IAM policies by granting themselves or each other permissions they don't need or shouldn't have.
		- The key security issue is that **creating/updating a Lambda isn't inherently dangerous if the developer can only attach approved execution roles**. Without `iam:PassRole` restrictions, someone could potentially deploy a Lambda with a highly privileged role and effectively turn that Lambda into a way to access resources they shouldn't have access to.
		- `iam:PassRole` controls which IAM roles a principal can assign to an AWS service such as Lambda. The developer's `iam:PassRole` permission should be restricted to only the approved Lambda execution roles. This prevents the developer from bypassing least-privilege controls by deploying a service with a more privileged role.

## IAM User vs. IAM Role

- **Scenario**: Suppose a developer needs to deploy infrastructure to AWS from their laptop. You have two possible approaches:
	- **Option A**: Create an IAM user with a permanent access key and secret access key. This gives the developer permanent credentials.
	- **Option B**: Use temporary credentials obtained through an IAM role. When the developer **assumes** the role, they are granted **temporary** permissions associated with the role.
- **Which approach would you prefer, and why are IAM roles with temporary credentials generally considered safer than long-lived access keys**?
> 	I would prefer assigning temporary credentials. This is generally considered safer than long-lived access keys because temporary credentials don't require long-term management. They don't need to be modified or deleted when a developer switches teams or leaves the company. Additionally, long-lived access keys pose a greater security threat when leaked than temporary credentials that can only be used once.
	- Temporary credentials still need to be managed. More precisely, temporary credentials **expire automatically**, so you don't have to manually rotate/delete a permanent credential when someone's access changes.
	- Temporary credentials **aren't necessarily single-use**. Instead, they're valid for a **limited period of time** and can be used during that period.
	- Temporary credentials also provide better centralized control because access is governed by the role and its policies.

# Key Management Service (KMS)

## Overview

- AWS KMS is primarily a service for **creating and managing encryption keys** and controlling who can use those keys.
- Think about an S3 bucket containing sensitive customer data. The important distinction is that KMS **isn't generally where you store the encrypted data**. Instead, AWS services such as S3, DynamoDB, and EBS can use KMS keys to perform encryption operations.
- **Scenario**: You have an S3 bucket containing customer data. Your security requirements state:
	- Data must be encrypted **at rest**.
	- Only a specific application should be able to decrypt it.
	- Developers should not automatically have access to the decrypted data.
	- Access to the encryption key should be auditable.
- **Why might you use KMS rather than having the application generate and manage its own encryption keys**?
> 	It would be better to abstract encryption key management to a separate service, such as KMS. The application should only need to worry about doing what it was designed to do. KMS natively integrates with S3, offering encryption at rest. KMS keys can be managed with key policies, which outline which entities are allowed to use a given key and what they're allowed to do with it. KMS also integrates with CloudTrail, allowing key owners to audit key access.
	- Key Points:
		- **Separation of responsibilities:** the application doesn't need to implement and maintain its own key-management system.
		- **AWS integration:** KMS integrates directly with services such as S3 for encryption at rest.
		- **Access control:** KMS key policies determine who can perform operations such as using or managing a key.
		- **Auditing:** KMS API activity can be recorded through CloudTrail.
	- KMS doesn't mean developers can never access decrypted data. **Access to the key and access to the underlying data are separate authorization decisions**.

## Symmetric vs. Asymmetric Keys

- For most AWS data-at-rest encryption use cases, you'll encounter **symmetric KMS keys**.
- A symmetric key conceptually looks like:
	```
	Plaintext
	   ↓
	Encryption key
	   ↓
	Ciphertext
	```
	- The same underlying key is used for **encryption and decryption**.
- Asymmetric cryptography uses **key pairs**. A public key encrypts data, while a private key decrypts it. This is useful for certain scenarios involving public/private key cryptography and digital signatures, but it isn't generally what you'd use to encrypt large amounts of S3 data.
- **Scenario**: Your application needs to encrypt **10 GB of customer data** before storing it in S3.
- **Would you have the application send all 10 GB directly through KMS to perform encryption? Or would you use a different approach**?
> 	I would configure the S3 bucket to use KMS to encrypt data at rest as it is uploaded to the bucket, instead of encrypting the data directly through KMS.
	- You generally **don't send the 10 GB payload to KMS for encryption**. Instead, you configure S3 to use a KMS key for server-side encryption.

## Evelope Encryption

- The basic idea is that KMS manages a **key-encryption key (KEK)**, while the actual data is encrypted using a separate **data encryption key (DEK)**.
- Conceptually:
	```
	                 KMS
	                  │
	             Master/KEK
	                  │
	                  ↓
	             Encrypts DEK
	                  │
	                  ↓
	Application/S3 ──→ DEK ──→ Encrypts data
	                            │
	                            ↓
	                         Ciphertext
	```
	- This allows the large amount of data to be encrypted using a symmetric data key rather than repeatedly sending the entire dataset through KMS.
- **Why is envelope encryption more practical than using a KMS key to directly encrypt every byte of a large dataset**?
> 	By allowing the encrypting a DEK once and allowing it to be used to perform the encryption, KMS only needs to worry managing data encryption keys. It doesn't need to worry about actually encrypting the data. This improves scalability, as KMS doesn't need to be called to encrypt every piece of data.
	- KMS doesn't actually "manage the data encryption keys" in the sense of performing all DEK operations itself. The **DEK handles the bulk data encryption**, while KMS protects/manages the higher-level key material.
	- Improved Answer:
		- Envelope encryption improves scalability because the data encryption key performs the actual encryption of the data, while KMS is used to protect the data encryption key. This means the large payload doesn't need to be sent through KMS for every encryption operation, reducing KMS overhead and allowing the system to efficiently encrypt large amounts of data.
- `kms:encrypt` / `kms:decrypt` grants an application permission to call KMS to perform data encryption / decryption on its behalf. Only 4 KB of data can be encrypted / decrypted per API call.
- `kms:GenerateDataKey` grants an application permission to perform **envelop encryption** to encrypt **large amounts of data** locally before sending it to storage.
	- When You Need It:
		- **Encrypting local data:** When your application needs a plaintext data key to encrypt files, database records, or objects locally (such as before uploading them to Amazon S3 or DynamoDB).
		- **Writing data to AWS services:** When custom applications or AWS services (like SQS, Step Functions, or custom data stores) need to generate a new data key to protect data at rest using a **customer-managed** KMS key.
		- **Performing client-side encryption:** When your code calls the AWS SDK to request a unique data key, it receives both a plaintext and an encrypted copy from KMS, uses the plaintext copy to lock the data, and then discards the plaintext copy. **The encrypted copy of the data key must be stored directly alongside the encrypted data**.
			- KMS does not store or keep track of the data keys it generates for you, this encrypted key is your only way to decrypt the data later.
- Encrypted Data Key Lifecycle:
	1. **Packaging:** Your application packages the encrypted data key and the encrypted payload together into a single file or database entry.
	2. **Storage:** You upload this package to your storage layer (like Amazon S3, DynamoDB, or a local hard drive).
	3. **Metadata Attachment:** In services like Amazon S3, this encrypted key is often stored in the object's **metadata fields** or headers.
	4. **Retrieve:** Your application reads the file and pulls out the encrypted data key.
	5. **Decrypt Key:** Your application sends _only_ that encrypted data key back to AWS KMS using the `kms:Decrypt` API.
	6. **Decrypt Data:** AWS KMS decrypts the key and returns the plaintext data key to your application.
	7. **Final Read:** Your application uses that plaintext key to decrypt the actual file locally.
- By storing the encrypted key with the data, the package becomes **self-contained**. You do not need a separate database just to map files to their respective encryption keys.

## Key Policies vs. IAM Policies

- Suppose you have a customer-data KMS key called `CustomerDataKey`. You want:
	- Application A → allowed to decrypt
	- Application B → denied
	- Developers → denied
	- Security team → allowed to administer the key
- KMS uses **key policies** to control access to the key. You can also use **IAM policies** to grant permissions involving KMS keys, but KMS has an important property: The key policy is a resource-based policy attached directly to the key, similar to an S3 bucket policy.
- **Scenario**: An application has the following IAM policy:
	```
	Allow:
	    kms:Decrypt
	    Resource: CustomerDataKey
	```
	- However, the KMS key's key policy does **not** allow that application/role to use the key.
- **Will the application necessarily be able to decrypt the data? And what role does the KMS key policy play in determining whether access is allowed**?
> 	If "does not allow" means an explicit denial, then the application won't be able to decrypt the data because an explicit denial always wins. If "does not allow" means the key policy doesn't specify permissions, then the permissions granted by the IAM policy would be sufficient. In this way the KMS key policy can act as a source of truth for who is allowed to access keys and what they're allowed to do with them, since the key policy is attached to the key directly.
	- With KMS, **the key policy is central to authorization**, but whether an IAM policy alone is sufficient depends on what the key policy says.
	- A KMS key policy can be configured to allow IAM policies to control access. In that case, an IAM policy alone would be enough to grant decrypt permissions.
	- If the key policy is written so that it **doesn't enable IAM policies to grant access**, then the IAM policy by itself isn't enough.
	- Improved Answer:
		- KMS authorization depends on both the key policy and IAM policies. The key policy determines how the key can be accessed and can either directly grant access or enable IAM policies to grant access. An explicit deny overrides an allow.

## Key Administrators vs. Key Users

- Key administrators need to:
	- Rotate the key
	- Modify the key policy
	- Disable/enable the key
	- Schedule deletion
- Key users only need to:
	- Encrypt data
	- Decrypt data
- You generally **don't want the application role to administer the key**.
- Why is it important to separate **key administration permissions** from **key usage permissions**? What could go wrong if the application role had both administrative permissions and `kms:Decrypt`?
> 	Key administration and usage permissions need to be separated to properly enforce least-privilege permissions. If they were not separate, users would be able grant themselves permissions they shouldn't have. If an application role had administration and usage permissions, the key could easily be compromised if the application were hacked.
	- The general principle is: An application that needs to use a key should not automatically have permission to administer that key.

## Single-Region vs. Multi-Region Keys

- Suppose an application operates in `us-east-1` and `us-west-2`. Both regions need to encrypt and decrypt the same category of data.
- You have two options:
	- Single-Region Key: Only exists and works in one AWS region, such as `us-east-1`.
	- Multi-Region Key: Provides **releated** KMS keys in multiple AWS regions that share the same underlying key material and key ID.
- **Scenario**: Your application is deployed active-active across **us-east-1 and us-west-2**, and encrypted data may need to be processed in either region.
- Why might a **Multi-Region KMS key** be preferable to using two completely independent KMS keys, one in each region?
> 	A Multi-Region KMS key would be preferable for this use case because data encrypted in one region using a Single-Region key can only be decrypted in that region. Once data is encrypted, it's essentially "stuck" in that region until it's decrypted. With Multi-Region keys, the same key material can be replicated across different regions, allowing data encrypted in one region to be decrypted in another region.
	- Multi-Region keys shouldn't automatically be used when data moves from one region to another. They should specifically be used when **encrypted** data needs to move from one region to another. If data that moves between regions can still be effectively encrypted and decrypted in one region, Single-Region keys are a better choice.
	- Don't introduce distributed infrastructure unless the requirements actually require it.

## S3 + KMS Access

- Suppose you have:
	```
	Application Lambda
	       ↓
	S3 customer-data bucket
	       ↓
	KMS CustomerDataKey
	```
	- The Lambda needs to:
		- Read encrypted objects from S3
		- Decrypt those objects
		- Process them
		- Write newly encrypted objects back to S3
- What permissions would you give the **Lambda execution role**, and what permissions would you give the **developer**?
> 	I would give the lambda permission to encrypt and decrypt data using the KMS key. I would also grant it access to get objects from the s3 bucket and write encrypted objects back to s3. I would only grant the developer permissions needed to access the Lambda's source code and modify the Lambda.
	- The Lamda needs S3 and KMS permissions:
		- `s3:GetObject`
		- `s3:PutObject`
		- `kms:Decrypt`
		- `kms:Encrypt`
	- These permissions should also be scoped to the specific bucket/prefix and KMS key rather than granting broad access.
	- The developer only needs permissions necessary to **deploy and modify the Lambda**. They **don**'t need:
		- `s3:GetObject`
		- `kms:Descrypt`
	- The developer's ability to modify the Lambda does create an important **indirect privilege-escalation consideration**, though. If the developer can deploy arbitrary Lambda code _and_ assign an execution role with `kms:Decrypt`, they could potentially use the Lambda to access customer data. That's why the `iam:PassRole` restriction is important: the developer should only be able to pass approved execution roles.

# Secrets Manager

## Overview

- **Scenario**: Your application needs to connect to an RDS PostgreSQL database. You could put the credentials directly in the application, but this creates security and operational problems.
- Why is **AWS Secrets Manager** preferable to hardcoding database credentials in application code or configuration?
> 	Using Secrets Manager is preferable to hardcoding database credentials in application code or configuration because Secrets Manager acts as a secure, centralized repository for sensitive information. When credentials need to created, updated, or deleted, you only need to do so in one place. Access to secrets manager can be governed and monitored using IAM and CloudTrail. If credentials are ever compromised, they only need to be updated in one place. Instead of hardcoding secrets directly in an application's code or configuration, an application would make an API call to retrieve the necessary secret and potentially cache it for repeated use.
	- Why Secrets Manager is Better:
		- **Centralized management** → credentials aren't scattered across source code/configuration.
		- **IAM-controlled access** → applications only get access to the secrets they need.
		- **Auditing** → access can be monitored through CloudTrail.
		- **Rotation** → credentials can be changed without modifying application source code.
		- **Reduced blast radius** → if credentials are compromised, you can rotate the secret rather than hunting through application code/configuration.
	- **Important Security Principle**: The application should have permission to retrieve the **specific secret it needs**—not broad permission to retrieve all secrets in the account.

## Credential Rotation

- Suppose your application uses:
	```
	Lambda
	  ↓
	Secrets Manager
	  ↓
	RDS PostgreSQL
	```
- One day, the rotation occurs while thousands of Lambda invocations are actively running.
- What could go wrong if Lambda instances cache the database credentials? How would you design the application so that credential rotation doesn't cause widespread failures?
> 	If a Lambda instance caches database credentials for too long, the cached credentials could become stale when the credentials are rotated. To ensure credential rotation doesn't cause widespread failures, I'd design the application to periodically poll Secrets Manager for updated values.
	- Instead of only relying on periodic polling, the application should be designed to detect authentication failures caused by credential rotation and retrieve the current secret before attempting to re-authenticate.
	- Periodic polling, even if done multiple times per second, could still lead to authentication failures caused by stale credentials.
	- **Another important consideration**: If an application uses connection pooling, **existing database connections may still have the old credentials** while newly created connections use the new credentials. The application needs to handle that transition gracefully rather than assuming refreshing the secret automatically fixes every existing connection.

## Secrets Manager vs. Parameter Store

- Suppose you need to store:
	- `DATABASE_PASSWORD`
	- `API_KEY`
	- `THIRD_PARTY_TOKEN`
- You could use **AWS Systems Manager Parameter Store** or **Secrets Manager**.
- What characteristics of the data or operational requirements would make **Secrets Manager** the better choice?
> 	For sensitive information such as passwords, API keys, and tokens, Secrets Manager is typically the better choice because it offers built-in, automatic rotation for services such as RDS, or custome rotation via Lambda. Secrets manager also offers native support for cross-account access and cross-region replication.
	- Secrets Manager is particularly appropriate when you're dealing with **credentials that need lifecycle management**, especially when automatic rotation is valuable.
	- Parameter Store is often a good fit for configuration values and simpler parameters, while Secrets Manager is purpose-built for sensitive credentials and their lifecycle, including rotation.
- Key Differences:
	- **Cost:** Parameter Store standard tier is **free**, whereas Secrets Manager charges per secret per month plus a small fee per 10,000 API calls. Advanced Parameter Store tiers cost a fraction of Secrets Manager.
	- **Rotation:** Secrets Manager features **built-in automatic rotation** for databases like RDS and custom rotation via Lambda. Parameter Store requires manual updates or custom external scheduling.
	- **Sharing:** Secrets Manager supports **cross-account access** natively using resource-based policies. Parameter Store does not support direct cross-account access out of the box.
	- **Data Types & Size:** Parameter Store handles plain text and encrypted strings up to 4KB (Standard) or 8KB (Advanced). Secrets Manager is tailored for structured credentials and larger secrets up to 10KB–64KB.
- Use Cases:
	- Parameter Store:
		- Non-sensitive application settings, feature flags, environment names, and service URLs.
		- Low-cost or free parameter storage where manual or CI/CD updates are sufficient.
	- Secrets Manager:
		- Highly sensitive credentials like database passwords, API keys, and OAuth tokens.
		- Assets that require scheduled, automated rotation or multi-region replication.

## Summary

- **Scenario**: You have three Lambda functions:
	- Lambda A = Production DB
	- Lambda B = Analytics DB
	- Lambda C = Third-Party API
- Secrets Manager Contains:
	- `prod-db-credentials`
	- `analytics-db-credentials`
	- `third-party-api-key`
- A security review discovers that **all three Lambda execution roles have permission to retrieve all three secrets**.
- What's wrong with this design? How would you redesign the IAM permissions to follow **least privilege**?
> 	This design violates the principle of least privilege. Each Lambda execution role should only have access to the secrets needed to execute properly. I would redesign the IAM permissions for each execution role by narrowing the scope to the specific secret(s) needed, instead of all secrets in the account.
	- This limits the **blast radius** if one Lambda or its execution role is compromised.

# CloudWatch

## Metrics

- CloudWatch is AWS's primary monitoring and observability service. The first concept to nail down is the distinction between **metrics, logs, and alarms.**
- A **metric** is a numerical measurement recorded over time.
- For example, an application might publish:
	- `CPUUtilization`
	- `RequestCount`
	- `ErrorCount`
	- `Latency`
	- `QueueDepth`
- Metrics are useful because they let you understand the **health and behavior of a system without inspecting individual requests**. They also allow you to visualize how system health and behavior change over time.
- **Scenario**: You operate an API service with a requirement that p99 latency must remain below 200 ms.
- The service currently publishes:
	- Average latency
	- p50 latency
	- p95 latency
	- p99 latency
	- Request count
	- Error count
- Which metric would you primarily monitor to determine whether the service is meeting its latency requirement, and why would **average latency** be a poor choice?
> 	I would monitor p99 latency because it is directly related to the SLA. Observing P50 latency would be inappropriate because it gives you a sense of how the application operates under average conditions, not how it would operate under extreme load.
	- P50 latency describes the **median** latency, not the average latency. It tells you that 50% of requests are faster than that value and 50% are slower.
	- The important distinction is:
		- P50 = typical request
		- P95 = tail of the distribution
		- P99 = extreme tail
	- When you set a P99 latency SLA, you're essentially saying: "99% of request latencies should fall below this target."

## Metrics vs. Logs

- Suppose your API suddenly experiences a spike in errors. 
	- A CloudWatch **metric** can tell you: "Error rate increased from 0.2% to 8%." It doesn't necessarily tell you **why**.
	- Your application **logs** might contain:
		```
		2026-08-26 16:32:01 ERROR
		OrderService
		DynamoDB ProvisionedThroughputExceededException
		customer_id=...
		```
	- Logs provide the **detailed context** surrounding individual events.
- Suppose you notice that your API's error rate suddenly increased. Would you primarily use **CloudWatch Metrics** or **CloudWatch Logs** to investigate the root cause?
> 	Both can be helpful. Metrics tell you how close the error rate is to breaching any defined SLA, which can tell you how quickly corrective action needs to be taken. Metrics can also tell you when the error rate started to increase, which can help narrow down your log search. Logs can provide insights into what specifically is causing the spike in error rates.
	- Metrics can also be used to **automatically trigger alarms**, while logs are generally more useful for investigation.

## Alarms

- An alarm watches a metric and transitions between states based on a configured threshold.
- For example:
	```
	p99 Latency
	    │
	300 │             ●
	    │           ●
	200 │───────────●──────── Threshold
	    │        ●
	100 │ ●  ●
	    │
	    └────────────────────→ Time
	
	              ↓
	          ALARM STATE
	```
- You might configure the alarm as follows: If p99 latency exceeds 200 ms for 3 consecutive evaluation periods, enter the `ALARM` state.
- The alarm can then trigger an action, such as:
	- Sending an SNS notification
	- Triggering an Auto Scaling policy
	- Invoking another AWS integration
- **Scenario**: Your API has a requirement:
	- p99 latency must remain below 200 ms.
	- You configure:
		- Threshold: 200 ms
		- Evaluation periods: 3
		- Datapoints to alarm: 3
	- The observed values are:
		- Period 1: 100 ms
		- Period 2: 215 ms
		- Period 3: 230 ms
		- Period 4: 190 ms
- **Would the alarm enter `ALARM` state? Why is requiring multiple evaluation periods potentially better than immediately alarming whenever a single datapoint exceeds 200 ms**?
> 	The alarm would not enter ALARM state because the SLA was only breached for two evaluation periods. Using an appropriate number of evaluation periods prevents the alarm from being flakey in the event of transient spikes.
	- Evaluation Periods (N) and Datapoints to Alarm (M) work together in Amazon CloudWatch to create an M-out-N alarm, defining the total window checked versus how many individual failures are required to trigger an alert.
	- In this example, 3 out of 3 datapoints would be needed to trigger the alarm, but only 2 out of 3 were recorded.

### Alarm Configuration
- Suppose your service has a P99 latency SLA of 200 ms.
- You want to alert the on-call engineer when there's a **sustained latency problem**, but you don't want to page them for a brief spike.
- You configure the alarm as follows:
	- Threshold: 200 ms
	- Evaluation periods: 5
	- Datapoints to alarm: 3
- This means the on-call engineer will be paged when 3 out of 5 datapoints break the threshold.
- Why might `3 out of 5` be preferable to `5 out of 5` for an on-call alert? What is the tradeoff compared with `1 out of 5`?
> 	3 out of 5 is preferable to 5 out of 5 because if the service is truly underperforming, but experiencing transient of relief, a 5 out of 5 configuration might not trigger the alarm when it's appropriate. On the other hand, a 1 out of 5 configuration could trigger an alarm in a healthy service that experiences transient latency spikes.
	- Alarm thresholds and evaluation periods should be chosen based on the behavior of the workload and the operational cost of false positives versus delayed detection.

### Composite Alarms
- Suppose your API has an error rate of 8%. That sounds pretty bad, but also imagine:
	- Error rate = 8%
	- CPU = 35%
	- Latency = Normal
	- Traffic / Request count = Normal
- The errors might be caused by a small number of invalid client requests rather than an unhealthy service.
- You might instead want to page the on-call engineer only when **multiple indicators suggest the service itself is unhealthy**. For example, page the on-call when the error rate and latency alarms transition into an `ALARM` state.
- A **CloudWatch Composite Alarm** combines the states of multiple underlying alarms using logical conditions.
- **Scenario**: You have:
	- `HighErrorRate` alarm
	- `HighLatency` alarm
	- `HighCPU` alarm
- You want to page the on-call engineer only when: `HighErrorRate AND (HighLatency OR HighCPU)`
- Why might a composite alarm be preferable to simply paging whenever `HighErrorRate` enters the `ALARM` state? What benefit does this provide for reducing **alert fatigue**?
> 	Depending on the nature of the service, a high error rate by itself may not be a reliable indicator of service health. The composite alarm will trigger when a high error rate is combined with either high latency or high CPU utilization. Using composite alarms can help reduce alarm fatigue by appropriately narrowing the scope that defines when an alarm is triggered.

## Dashboards

- Suppose you're operating an SQS → Lambda → DynamoDB pipeline.
- You have the following metrics:
	- SQS Queue Depth
	- Lambda Invocation Count
	- Lambda Error Rate
	- Lambda Duration
	- DynamoDB Throttled Requests
	- DynamoDB Consumed Capacity
- An engineer looking at these individually might have difficulty understanding how the system is behaving as a whole. A dashboard can put the relevant metrics together.
- **Scenario**: You notice that **SQS queue depth is steadily increasing**.
- What would you investigate next using the other metrics?
- Specifically, how could you distinguish between:
	1. **Lambda doesn't have enough processing capacity**
	2. **Lambda is processing messages slowly because DynamoDB is the bottleneck**
	3. **DynamoDB itself is throttling requests**
> 	I would investigate Lambda Duration and DynamoDB Throttling. These metrics could give me a sense of how Lambda and DynamoDB are working together to process messages in the queue. If Lambda Duration is increasing and/or elevated, but DynamoDB Throttling is level and normal, it would indicate Lambda doesn't have enough processing capacity. If Lambda Duration is level and normal, but DynamoDB Throttling is increasing and/or elevated, it could indicate DynamoDB is the bottleneck. If Queue Depth and Lambda Duration are level and normal, but DynamoDB Throttling is increasing and/or elevated, it would indicate DynamoDB is throttling requests.
		- Lambda Capacity Problem:
			- **SQS queue depth ↑**
			- **Lambda duration ↑**
			- **DynamoDB throttling → normal**
			- Investigate Lambda's own processing capacity—CPU, memory, concurrency, downstream calls, etc.
		- DynamoDB Bottleneck:
			- **SQS queue depth ↑**
			- **Lambda duration ↑**
			- **DynamoDB throttling ↑**
		- DynamoDB Throttling:
			- If **Lambda duration remains normal while queue depth increases**, that would suggest the Lambda may not actually be waiting on DynamoDB long enough for throttling to affect its duration, or that the throttling is occurring on a path that isn't reflected in the measured duration.
		- Lambda **concurrency** is also a useful metric to look at. If queue depth is increasing while Lambda concurrency is already near its configured limit, that strongly suggests you've hit a Lambda concurrency bottleneck.

## Log Insights

- Suppose your Lambda logs contain entries such as:
	```
	2026-08-26T17:10:01 ERROR
	request_id=abc123
	operation=UpdateOrder
	error=DynamoDB.ThrottlingException
	duration_ms=842
	
	2026-08-26T17:10:02 ERROR
	request_id=def456
	operation=UpdateOrder
	error=DynamoDB.ThrottlingException
	duration_ms=911
	```
- You have **millions of log entries**, so manually searching through them isn't practical.
- You want to know: Which errors occurred most frequently during the last 30 minutes?
- How would **CloudWatch Logs Insights** help you answer this?
	- CloudWatch Log Insights would help efficiently search logs by enabling you to use SQL-style syntax to query the log data.
	- Logs Insights lets you **query and aggregate large volumes of CloudWatch logs** instead of manually inspecting individual entries.
	- Log Insights uses the **CloudWatch Logs Insights query language**, which is SQL-like in some ways, but isn't actually SQL.
- For this particular problem, you'd want to:
	- Filter logs to the last 30 minutes.
	- Filter for error entries.
	- Group/count them by the error type.
	- Sort by frequency.
- This lets you quickly discover something like:
	```
	DynamoDB.ThrottlingException    12,431
	TimeoutException                 3,812
	ValidationException              1,204
	```
	- Rather than manually searching millions of records.
- **Scenario**: Your API started returning a large number of `500` errors around **17:00**.
	- You want to determine: Which API endpoint is generating the most 500 errors, and what downstream dependency is associated with those failures?
	- Your logs contain fields such as:
		- `timestamp`
		- `status_code`
		- `endpoint`
		- `request_id`
		- `dependency`
		- `error_type`
	- How would you approach this investigation using **CloudWatch Logs Insights**?
> 	I would filter by status_code and timestamp, group by by endpoint, and count request_id to determine which API is producing the most errors. Once I knew which API was causing the errors, I'd filter by status_code, timestamp, and endpoint, the group by dependency and count request_id to determine which dependency is causing the failures.
		- **`count(*)`** would typically be more direct than counting `request_id`, assuming every log record represents one request/error. However, `count(request_id)` is still a valid approach.

## Summary

- **Scenario 1**: You operate this pipeline:
	```
	                SQS
	                 ↓
	              Lambda
	                 ↓
	             DynamoDB
	```
- At 2:00 PM, your monitoring shows:
	```
	SQS Queue Depth       ↑↑
	Lambda Duration       ↑
	Lambda Errors         ↑
	DynamoDB Throttles    ↑
	```
- CloudWatch Log Insights shows: `DynamoDB.ProvisionedThroughputExceededException` appearing frequently in the Lambda logs.
- Walk me through how you would investigate this incident using **CloudWatch Metrics, Alarms, Dashboards, and Logs Insights**.
> 	I'd look at the dashboard to determine if there is a relationship between the various metrics. I'd also check alarm history to see if any alarms are active or were recently triggered. Reviewing the metrics and dashboards would allow me to use log insights to narrow my search to the specific service or services that are unhealthy, as well as narrow down the log search based on timestamp. Using the results of the Log Insights query would allow me to identify the likely bottleneck causing the issues.
- Before concluding what the bottleneck is based on the results of the Log Insights query, the hypothesis **should be verified**. A good debugging workflow would look like:
	1. Check the dashboard
		- Establish when the queue depth started increasing.
		- Correlate Lambda duration/errors with DynamoDB throttling.
		- Determine whether the symptoms started at roughly the same time.
	2. Check alarm history
		- Identify which alarms triggered and when.
		- This helps establish the incident timeline and whether the current behavior crossed predefined thresholds.
	3. Use Log Insights
		- Narrow the log search to the incident window.
		- Since DynamoDB throttling is increasing, search for `ProvisionedThroughputExceededException`.
		- Aggregate by relevant fields such as table, operation, or partition key if those are available in the logs.
	4. **Confirm the bottleneck**
		- If DynamoDB throttling correlates with increased Lambda duration and queue depth, DynamoDB is a strong candidate for the bottleneck.
		- Then investigate _why_ DynamoDB is throttling—insufficient capacity, a hot partition, an inappropriate capacity mode, etc.
	5. Take corrective action
		- Depending on the cause, you might adjust DynamoDB capacity, address a hot partition, or reduce/reshape the workload.
		- Then use the same CloudWatch metrics to verify that queue depth and Lambda latency return to normal.
- Key Interview Principle:
	- Metrics tell you where and when the problem is; Logs Insights helps you determine why; alarms tell you when predefined conditions have been breached; dashboards let you correlate the system's behavior.
- **Scenario 2**: You're deploying a new **SQS → Lambda → DynamoDB** data-processing pipeline.
- You need monitoring that can detect:
	- Messages accumulating faster than Lambda can process them
	- Lambda processing failures
	- Excessive Lambda latency
	- DynamoDB throttling
	- A situation where the system is technically functioning but becoming unhealthy
- What **CloudWatch metrics and alarms** would you create for this pipeline? For each major component, tell me **what you'd monitor and what kind of condition would cause an alarm**.
> 	For Lambda, I'd look at processing latency and concurrency to determine how well Lambda is handling the load. I'd also check error counts to see if there is a spike in processing failures. For SQS, I'd look at queue depth and the age of the oldest message to determine how severe the backlog is. For DynamoDB, I'd look at throttling metrics to determine if the database is being overwhelmed. When reviewing alarm history, I'd look for SQS queue depth, Lambda duration, Lambda errors, and DynamoDB throttling.
	- Lambda:
		- **Duration** → are individual messages taking too long?
		- **Concurrency** → are we approaching Lambda's concurrency limit?
		- **Errors** → are messages repeatedly failing?
		- **Also consider Lambda throttling**, which specifically tell you Lambda is rejecting invocations because concurrency capacity has been reached.
	- SQS:
		- **ApproximateNumberOfMessagesVisible** → backlog size
		- **ApproximateAgeOfOldestMessage** → how long messages are waiting
	- DynamoDB:
		- **Throttled requests** are the key metric for detecting capacity pressure.

# CloudTrail

## Overview

- A useful mental model for distinguishing between CloudWatch and CloudTrail is:
	- **CloudWatch tells you what your systems are doing.** 
	- **CloudTrail tells you what actions were taken against your AWS account/resources.**
- In CloudWatch, you might see:
	```
	Lambda errors ↑
	DynamoDB throttling ↑
	CPU utilization ↑
	```
	- This tells you **something is happening** to the system.
- In CloudTrail, you might see:
	```
	Who:     arn:aws:iam::123456789:role/Developer
	Action:  DeleteTable
	Resource: OrdersTable
	Time:    14:32 UTC
	```
	- This helps answer: "Who performed an AWS API action, what action did they perform, and when?"
- **Scenario**: You notice that an S3 bucket's configuration unexpectedly changed.
	- Your CloudWatch dashboard looks completely normal: CPU, Latency, and Error Count are all normal.
	- However, someone changed the bucket policy and the application can no longer access the bucket.
- Why would **CloudTrail** be more useful than CloudWatch for investigating this incident? And what information would you look for in the CloudTrail event?
> 	CloudWatch primarily tells you how a system is behaving. CloudTrail primarily tells you what actions have been taken against your AWS account or the resources within it. CloudTrail would be a more useful investigative tool in this scenario because all metrics are normal, but the bucket policy was changed. This modification made the bucket inaccessible to the application. CloudTrail can tell you who performed this action and when.
	- Useful CloudTrail Information:
		- **Who** performed the action (`userIdentity`)
		- **What API action** was performed (`eventName`, e.g. `PutBucketPolicy`)
		- **When** it occurred (`eventTime`)
		- **Which resource/account/region** was involved
		- **Where the request came from**, such as source IP or AWS service
		- **How the request was made**, such as console, CLI, SDK, or another AWS service
	- This gives you the audit trail needed to determine whether the change was intentional, accidental, or potentially malicious.

## Management Events vs. Data Events

- CloudTrail can record different categories of activity. Two important categories are management and data events.
- **Management Events**:
	- These involve **management/control-plane operations**, such as:
		- `CreateBucket`
		- `DeleteBucket`
		- `PutBucketPolicy`
		- `CreateRole`
		- `UpdateFunctionConfiguration`
		- `CreateTable`
	- These are generally about **creating, modifying, or deleting AWS resources/configuration**.
- **Data Events**:
	- These concern **operations performed on the actual data inside a resource**.
	- For example, S3 data events can include:
		- `GetObject`
		- `PutObject`
		- `DeleteObject`
	- These are **object-level** operations, which is why they're considered data events. The management events are **bucket-level** events.
- **Scenario 1**: Your security team wants to investigate: "Who accessed sensitive customer files in our S3 bucket during the last 24 hours?"
- They don't care who changed the bucket configuration. They specifically want to know **who read individual objects**.
- Would you primarily need **management events or data events**?
> 	Since the primary concern isn't how modified the bucket configuration, you would primarily need to look at data events, such as GetObject, to determine who read the individual objects. Management events would primarily tell you how the bucket was modified.
	- **Management events** → changes to AWS resources and configuration.
	- **Data events** → operations performed against the actual data within supported resources.
- **Scenario 2**: Your security team receives an alert: An IAM role that normally operates from your AWS infrastructure suddenly performed several sensitive API calls from an unfamiliar source IP address.
- CloudTrail shows:
	```
	Principal:  arn:aws:iam::123456789:role/ApplicationRole
	Source IP:  203.0.113.50
	Actions:
	    PutBucketPolicy
	    CreateAccessKey
	    GetObject
	```
- How would you use CloudTrail to investigate this incident?
> 	I'd look at CloudTrail management events related to changing the configuration of the ApplicationRole, since this role is being used by the unfamiliar IP address to perform the sensitive API calls. I'd also look at data events to determine if sensitive data could have potentially been exposed.
- Proper Investigation Sequence:
	1. Examine the CloudTrail event itself:
		- `eventName`
		- `eventTime`
		- `userIdentity`
		- `sourceIPAddress`
		- `userAgent`
		- affected resource
	2. Investigate the role's recent management activity:
		- Look for changes to the role or its policies.
		- Pay particular attention to actions such as `PutRolePolicy`, `AttachRolePolicy`, or changes to the trust policy.
		- This could reveal how an attacker obtained or expanded access.
	3. Investigate the suspicious API activity:
		- Look at the `PutBucketPolicy`, `CreateAccessKey`, and other sensitive calls.
		- Determine what resources were affected and whether the calls were successful.
	4. Investigate potential data exposure:
		- Use S3 **data events** to look for `GetObject` activity around the same period.
		- Determine which objects were accessed and by which principal.
	5. Establish a timeline:
		```
		Role/policy modified
		        ↓
		Suspicious API access
		        ↓
		Bucket policy modified
		        ↓
		S3 objects accessed
		```
		- This helps distinguish **initial compromise → privilege escalation → malicious activity → potential data exposure**.
- The fact that the unfamiliar IP is using the application role doesn't necessarily mean the **role itself was modified**. The credentials could have been compromised without changing the role. So both **changes to the role** _and_ **how/when the role was assumed** should be investigated.
- CloudTrail can record the `AssumeRole` event, which can help trace the chain:
	```
	IAM User / Service
	       ↓
	   AssumeRole
	       ↓
	ApplicationRole
	       ↓
	S3 / DynamoDB / etc.
	```
	- Role assumption is important when investigating CloudTrail activity because it tells you who had access to the role. You can compare this to the role's trust policy to see if the assumption is valid or invalid. Only looking at the PutBucketPolicy event doesn't necessarily tell you who assumed the role that grants those permissions.
	- The `AssumeRole` event should be examined and the caller should be compared against the role's trust policy to determine if the role assumption was expected and authorized.

## Retention and Centralization

- Imagine your company has:
	- Account A = Production
	- Account B = Development
	- Account C = Security / Logging
- The security team wants to ensure that an attacker who compromises the production account **cannot simply delete or modify the audit logs** needed to investigate the incident.
- How would you design CloudTrail logging so that audit records are protected from someone who compromises the production account?
> 	I'd configure CloudTrail to deliver logs from the production account into a centralized S3 bucket in a dedicated security/logging account. Access to that bucket would be restricted to the security team and other explicitly authorized personnel. This protects the audit trail from an attacker who gains administrative access to the production account, because they wouldn't automatically have permission to modify or delete the centralized logs.
- The important architectural pattern is:
	```
	Production Account
	       │
	       │ CloudTrail
	       ↓
	┌──────────────────────┐
	│ Security/Log Account │
	│                      │
	│  Central S3 Bucket   │
	│  CloudTrail Logs     │
	└──────────────────────┘
	       ↑
	       │
	  Restricted access
	```
	- This is essentially **separation of concerns**: the account generating the activity shouldn't have unrestricted control over the evidence used to audit that activity.

# API Gateway

## Overview

- API Gateway is a managed front door for APIs. A common architecture is:
	```
	Client
	   ↓
	API Gateway
	   ↓
	Lambda / ECS / other backend
	   ↓
	Application logic
	```
- API Gateway can handle concerns such as:
	- Routing requests
	- Authentication/authorization
	- Validation
	- Rate limiting/throttling
	- Request/response transformation
	- Monitoring
	- API lifecycle/version management
- **Scenario**: You have a serverless application:
	```
	Mobile App
	    ↓
	Lambda
	    ↓
	DynamoDB
	```
- The Lambda function currently has a public endpoint that clients invoke directly. You want to introduce API Gateway.
- What advantages does putting **API Gateway in front of Lambda** provide compared with exposing the Lambda endpoint directly?
> 	API Gateway can act as a secure front door that provides authentication, validation, and throttling. By routing requests through API Gateway before sending them to the Lambda for processing, the Lambda function and downstream services and data can be protected from malicious activity.
	- **API Gateway doesn't inherently make the backend immune to malicious traffic**. It provides mechanisms to authenticate, authorize, validate, and throttle requests, which can substantially reduce unwanted load and protect backend resources.

## Throttling

- Suppose your API typically handles 1,000 TPS traffic. Suddenly a buggy client starts sending 50,000 TPS traffic. Without throttling, those requests could overwhelm your Lambda functions and downstream DynamoDB table.
- How would **API Gateway throttling** help in this situation? Why is throttling useful even when Lambda itself can automatically scale?
> 	API Gateway can help in this situation by denying requests from clients whose request rate exceeds an established threshold. Throttling is useful even if the Lambda function can handle the load because this does not mean any downstream dependencies can also scale to handle the increased load.
	- API Gateway provides a **protective layer in front of horizontally scalable backends**. Lambda may be able to scale rapidly, but that doesn't mean every downstream dependency, such as DynamoDB, can scale at the same rate.
	- Throttling doesn't necessarily mean API Gateway permanently **denies** requests. Depending on the configuration and API Gateway behavior, requests exceeding the rate/burst limits can be **throttled/rejected**, typically resulting in a `429 Too Many Requests`response.
- API Gateway can throttle requests at various levels, depending on the configuration:
	- **Account/Region-level throttling:** A shared limit across your APIs in an AWS account/region. This acts as a global safety limit.
	- **Stage/API/method-level throttling:** You can set more specific limits for particular APIs, stages, or methods.
	- **Usage plans + API keys:** You can throttle **per client/API key**, with limits such as requests per second and burst capacity.
- **API Gateway does not inherently throttle every client independently**. The default throttling is primarily a shared/account-level mechanism, while **per-client throttling requires usage plans/API keys (or another client-specific mechanism)**.

## Authentication vs. Authorization

- Suppose you have a `POST /orders` API. Only authenticated users should be able to call it. Furthermore, User A should only be able to access **their own orders**, while an administrator can access orders belonging to any user.
- What's the difference between **authentication** and **authorization** in this scenario? Where would you expect API Gateway to participate in this process?
> 	Authentication verifies who is making the request, while authorization verifies they have the required permissions. API Gateway can perform authentication, while backend services can perform authorization through least-privilege IAM policies.
	- Authentication can **sometimes** be performed using least-priviledge IAM roles, but there are certain scenarios IAM can't handle. For example, determining whether `User A` is allowed to call `GET /orders/123` depends on whether **order 123 belongs to User A**. That's **application-level authorization**, and you'd typically need the backend to enforce that ownership rule.
	- Authentication verifies the caller's identity, while authorization determines what that identity is permitted to access. API Gateway can participate in authentication and authorization using mechanisms such as authorizers. The backend should still enforce resource-level authorization, such as verifying that a user actually owns or is allowed to access a requested resource.

## Rate Limiting vs. Authentication

- Suppose you have a **public API** that allows unauthenticated users to search products. You want to prevent a single client from consuming excessive resources. You decide to impose a limit of 100 requests per minute per API key.
- What is the purpose of **API keys and usage plans** in API Gateway? How is this different from using authentication to determine whether someone is allowed to access the API at all?
> 	The API keys and usage plans allow API Gateway to apply the limit to a specific user or key, rather than globally for all requests. This is differs from using authentication because authentication only determines who someone is, it doesn't necessarily tell you how many requests they've made.
	- **API key / usage plan** → identifies a client/application for purposes of **usage controls**, such as quotas and throttling.
	- **Authentication** → establishes the identity of the caller.
	- **Authorization** → determines what that authenticated identity is allowed to access.
	- **An API key should not generally be treated as a strong authentication mechanism by itself.** It's primarily useful for identifying and controlling API consumers.

## Backend Protection

- Suppose you have application consisting of:
	```
	Mobile App
	    ↓
	API Gateway
	    ↓
	Lambda
	    ↓
	DynamoDB
	```
	- A developer accidentally exposes the Lambda function's **Function URL** publicly, allowing clients to bypass API Gateway entirely.
- Why is this a security and architecture problem? How would you ensure that clients **must go through API Gateway** rather than directly invoking the Lambda?
> 	I would prevent the Lambda from being publicly invokable and configure it so that only API Gateway has permission to invoke it. The Lambda's resource-based policy can restrict `lambda:InvokeFunction` to the API Gateway service/principal associated with the API. That way, even if someone knows the Lambda's endpoint, they can't bypass the API Gateway controls.
	- Using resource-based policies to restrict how the Lambda can be invoked is better than implementing a custom signature policy, where API signs requests before sending them to Lambda, then Lambda validates the signature before processing the request.
	- Using resource-based policies is much simpler and uses existing AWS architecture.

## API Gateway vs. Load Balancer

- Suppose you have a backend service running on **ECS**:
	```
	Clients
	   ↓
	???
	   ↓
	ECS Service
	```
- You need:
	- HTTP/HTTPS routing
	- TLS termination
	- Health checks
	- Load balancing across ECS tasks
- You **don't** necessarily need API-specific features such as API keys, usage plans, request transformation, or API lifecycle management.
- Would **API Gateway** or an **Application Load Balancer (ALB)** be the more natural choice here? What characteristics of the workload would make you choose one over the other?
> 	An Application Load Balancer would be a more natural choice because the workload requires health checks, TLS termination, and load balancing across tasks, not just request routing. Furthermore the service doesn't require API-specific features.
	- An **ALB** is the more natural choice when the primary requirement is distributing HTTP/HTTPS traffic across a fleet of backend services.
	- ALB is primarily a load balancer for backend compute, while API Gateway is primarily an API management/front-door service.

|Requirement|ALB|API Gateway|
|---|---|---|
|Load balance ECS tasks|✅|Not its primary purpose|
|Health checks|✅|Not the same role|
|TLS termination|✅|✅|
|Basic HTTP routing|✅|✅|
|API keys / usage plans|❌|✅|
|API-specific authorization|Limited|✅|
|Request/response transformation|Limited|✅|
|Serverless API front door|—|✅|