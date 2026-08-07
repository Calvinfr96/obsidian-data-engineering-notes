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

## Mock Interview

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