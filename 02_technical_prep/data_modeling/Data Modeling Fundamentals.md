## 1. Normalization vs. Denormalization

- Imagine you're building a database for an online store. You have customers placing orders. One way to store the data is:

| Order ID | Customer Name | Customer Email                          | Customer Address | Product  | Price |
| -------- | ------------- | --------------------------------------- | ---------------- | -------- | ----- |
| 1001     | John Smith    | [john@email.com](mailto:john@email.com) | 123 Main St      | Laptop   | 1200  |
| 1002     | John Smith    | [john@email.com](mailto:john@email.com) | 123 Main St      | Mouse    | 30    |
| 1003     | John Smith    | [john@email.com](mailto:john@email.com) | 123 Main St      | Keyboard | 90    |
- The problem with this approach is that John's information (email and address) appears in every order. If John changes any of this information, all of the order records must also be updated.
	- If one row is missed during this update, now your database has inconsistent address information. This is called an **update anomaly**.

### Normalization
- **Normalization** reduces redundancy by storing each **fact** exactly once. Instead of one wide table, you'll have two, more narrow tables:

| Customer ID | Name       | Email                                   | Address     |
| ----------- | ---------- | --------------------------------------- | ----------- |
| 42          | John Smith | [john@email.com](mailto:john@email.com) | 123 Main St |

|Order ID|Customer ID|Product|Price|
|---|---|---|---|
|1001|42|Laptop|1200|
|1002|42|Mouse|30|
|1003|42|Keyboard|90|
- Now, John's address exists in one place and only one row needs to be updated.
- Normalization minimizes:
	- Duplicate data
	- Storage requirements
	- Update anomalies
	- Insert anomalies
	- Delete anomalies
- Most transactional databases (OLTP systems) are highly normalized. For example:
	- Banking
	- E-commerce
	- Inventory
	- HR systems
- One major tradeoff of Normalization:
	- When you want data from two normalized tables, such as "John's recent orders," you need to join the Customers and Orders table.
	- This costs CPU and I/O.

### Denormalization
- Now imagine you're building a dashboard that is read millions of times. Customer information rarely changes. Instead of performing a join in every query, you keep all relevant information in one, wide table:

|Order ID|Customer Name|City|Product|Price|
|---|---|---|---|---|
|1001|John Smith|Seattle|Laptop|1200|
|1002|John Smith|Seattle|Mouse|30|
- All data is together and queries become much faster.
- Data Engineers and Data Analysts use denormalized tables because storage is often cheaper than computation. Instead of paying for repeated joins, you pay for extra storage.
- Modern analytics platforms make this tradeoff frequently.

### Tradeoffs
- Normalization:
	- Pros:
		- Minimal duplication
		- Easier updates
		- Better consistency
		- Smaller storage footprint
	- Cons:
		- More JOINs
		- Slower analytical queries
		- More complex reads
- Denormalization:
	- Pros:
		- Faster reads
		- Fewer JOINs
		- Simpler analytical queries
		- Better reporting performance
	- Cons:
		- Duplicate data
		- More storage
		- Harder updates
		- Risk of inconsistent copies
- Real-World Example:
	- If you update your address in your Amazon profile, should they go back and update the address in all of your previous orders?
		- Probably not. Old orders should preserve the address that was valid when the purchase occurred.
		- This is **intentional denormalization**. Historical data is more important than duplicate data.
		- This is a great example of **business requirements** driving design.
- **Key Takeaway:** Normalization optimizes writes; denormalization optimizes reads.
	- This isn't universally true, but it's a good rule of thumb.

### Data Engineering Perspective
- Normalization and Denormalization aren't mutually exclusive. They both have their place in different parts of a system.
	- A transactional database is highly normalized:
		```
		Orders
		
		Customers
		
		Products
		```
	- An analytics warehouse might contain all of the following data in one table:
		- Customer region
		- Product category
		- Store name
		- Promotion
		- Sales amount
	- This is a highly denormalized structure. You have one table with minimal joins.
	- It's structured this way because dashboards need to answer questions quickly.

### Star Schema
- The star schema is one of the most common warehouse designs:
	```
	          Customers
	              │
	Products ── Sales ── Dates
	              │
	          Stores
	```
	- The sales table is wide and stores measurements such as:
		- Revenue
		- Quantity
		- Profit
	- The surrounding tables are relatively narrow and only contain descriptive information.

### Common Examples
- **Data Engineering Example:** Suppose you're building a daily revenue dashboard.
	- The dashboard shows the following metrics:
		- State
		- Product category
		- Customer segment
	- Instead of joining multiple tables for every database refresh, one denormalized fact table is maintained.
	- Most warehouses choose this model because reads vastly outnumber writes.
- **Amazon Example:** Caches, such as the one used to store creative assets, often store data that is contained in another system.
	- This is also a form of denormalization.
	- The purpose of a cache isn't to eliminate duplication, it's to **optimize read performance**.
- Sometimes duplicating data is the right design decision when it improves performance or availability.

### Common Interview Questions
1. Suppose you're designing the backend for a banking application. Customers update their contact information regularly, and correctness is critical. Would you favor a normalized or denormalized schema? Why?
> 	I would maintain normalized tables for customer and account information and a denormalized reporting table or materialized view that the dashboard queries. Using a hybrid approach optimizes the underlying tables for writes and reduces data duplication and the potential for malformed updates. Meanwhile, the table used to create the dashboard is optimized for reads because repetitive joins aren't required.
2. Now suppose you're building a business intelligence dashboard that executives use every morning. The dashboard joins customer, product, store, promotion, and sales data. The underlying data is updated once overnight but queried thousands of times during the day. Would you normalize or denormalize the reporting tables? Why?
> 	I would denormalize the reporting tables because they need to be optimized for reads not writes. Joining all of the tables for each read would be especially inefficient considering the underlying tables are only updated once a day.
3. Storage is cheap, so why don't we always denormalize?
	- Cost isn't everything. Imagine a Customers table with 100 million customers. Now duplicate customer information into the following tables:
		- Orders
		- Payments
		- Returns
		- Shipments
		- Support tickets
		- Marketing events
	- Suddenly, you're paying for 5TB of storage, instead of 100GB
	- Storage may be relatively inexpensive, but:
		- ETL pipelines become more expensive
		- Data transfers increase
		- Update logic becomes more complex
		- Consistency becomes harder
	- **Duplicate data only when the performance benefits outweigh the operational costs**.
4. Should I normalize or denormalize?
	- It depends on the workload you're optimizing for:
		- Frequent writes → normalize.
		- Heavy analytical reads → often denormalize.
		- Many production systems use **both**: a normalized operational database and a denormalized analytical layer.
	- This demonstrates technical knowledge and practical experience.

## 2. Partition Keys

- Imagine you have a table that contains **2 billion** rows. and someone runs the following query:
	```sql
	SELECT *
	FROM Orders
	WHERE OrderDate = '2026-07-01';
	```
- If your database stores these 2 billion rows together, it must scan the **entire table** looking for matching rows.
- Instead of scanning the entire table, partitions allow the database to divide a table into smaller pieces and query those pieces separately:
	```
	Orders
	
	├── 2026-06
	
	├── 2026-07
	
	├── 2026-08
	
	├── 2026-09
	```
	- Now, the same query is much faster because it only scans the `2026-07` partition of the table. The other partitions are ignored. This is called **partition pruning**.
- A **partition key** is the attribute used to decide where data is stored. Examples include:
	- Date
	- Customer ID
	- Country
	- Region
	- Device Type
	- Account ID
- The partition key determines which partition receives each row.

### Choosing a Good Partion Key
- The partition key should:
	- Evenly distribute data.
	- Match common query patterns.
	- Avoid creating too many tiny partitions.
	- Avoid creating one gigantic partition.
- Notice there are multiple goals. That's why choosing a partition key is a design decision.
- For example:
	- If most queries ask `Last 7 Days`, partitioning by date makes perfect sense.
	- If analysts commonly ask `WHERE Country = 'Canada'`, partitioning by country may be benificial.
	- If every query looks like `WHERE CustomerID = 12345`, partitioning by customer ID may reduce the amount of data scanned.
- Bad Partition Key:
	- If you partition a Customers table by gender, all you've done is create two giant partitions.
	- On the other hand, if you partition an Orders table by timestamp, now you have millions of tiny partitions. The metadata overhead becomes enormous. This is called the **small files problem** in data lake systems.

### Tradeoffs
- Large Partitons:
	- Pros:
		- Less metadata
		- Easier management
	- Cons:
		- More data scanned
		- Slower queries
- Small Partitions:
	- Pros:
		- Excellent prunning
		- Fast selective queries
	- Cons:
		- Metadata overhead
		- Too many files
		- Scheduling overhead
		- Poor performance in distributed systems
- The best partition key aligns with your most common access patterns while distributing data evenly.

### Partitioning vs. Indexing
- Many candidates confuse partitioning with indexing. They're different:
	- An index helps find data quickly **within** stored data.
	- A partition determines **where data is stored**.
- A partition may still have a lot of indexes. The two techniques compliment each other.

### Common Examples
- **Amazon Example:** Imagine Amazon tracking deliveries.
	- Queries often ask:
		- Yesterday's deliveries
		- This week's deliveries
		- Last month's deliveries
	- Partitioning by delivery date makes sense in this case. However, partitioning by package weight doesn't make sense because no one queries the data this way.
- **Cache Warming Project Example:** Suppose the cache contained billions of ad creatives.
	- You wouldn't store them all together and you also wouldn't store them randomly.
	- You'd likely partition them in a way that matches how they're retrieved—perhaps by region, customer, or campaign—so requests can efficiently locate the relevant data without scanning everything.
	- **The exact key depends on the application's access patterns**.
- **Spark Example:** Suppose you have:
	```
	sales/
	
	2025/
	
	2026/
	
	2027/
	```
	- When you query `WHERE Year = 2026`, Spark only reads `sales/2026/`, instead of the entire sales directory. This dramatically reduces I/O.
- Most warehouses partition by:
	- Date
	- Ingestion date
	- Event date
- This is because analytical queries almost always filter by time.

### Common Interview Questions
1. You're designing a table that stores **10 years of web clickstream events**. Analysts almost always query: `Last 7 Days`. What partition key would you choose? Why?
> 	I would partition by day because this aligns with common data access patterns.
	- Partitioning by year and month would create very large partitions.
	- While partitioning by month seems like a good compromise between day and year, partitioning by month leads to unnecessary data scanning if analysts are only querying the last 7 days.
	- Additionally, partitioning by day creates about 3,650 partitions for 10 years worth of data. This might seem like a lot, but it's actually fairly reasonable for many modern data warehouse and data lake technologies.
	- **The partition size should be driven by the query pattern and the capabilities of the storage engine—not by a fixed rule like "monthly is always best**.
	- In many real-world analytics systems, **daily partitioning** is the standard because time-based queries are so common.
2. An engineer suggests partitioning a customer table by **first name** because "it's easy." Would you agree? Why or why not?
> 	I would only agree if this choice of partition key aligns with common data access patterns. Partitioning by first name would create uneven partitions, as some names are more popular than others. Many small partitions would exist for rare names while few large partitions would exist for very common names.

### Partition Key Evaluation Framework
1. Does it match query patterns?
	- Example: `WHERE OrderDate = ...`
	- A good partition key would be `OrderDate`.
2. Does it distribute data evenly?
	- Good Example: Customer ID
	- Bad Examples:
		- First Name
		- Gender
		- Country
3. Is the partition size appropriate?
	- Too large? Slow scans.
	- Too small? Metadata overhead.
- How do you choose a partition key?
> 	I evaluate three things: whether the key aligns with common query patterns, whether it distributes data evenly to avoid skew, and whether it creates partitions of an appropriate size for the storage system.

## 3. Hot Partitions

- A hot partition is one of the most common failure modes in a distributed data system.
- A partition key can be **correct**, but still create a terrible system. This is because choosing **where** data goes is only half the problem. You also need to think about **how much** data and traffic a partition receives.
- A **balanced distribution** occurs when all partitions do roughly the same amount of work.
- A **hot partition**  is one that receives a **disproportionately large amount of traffic** compared to the others. This could cause one server to overload while the others remain idle.
	- A good partition key cannot prevent a partition from becomming hot.
- One of the biggest lessons in distributed systems is: "The system is only as fast as its busiest partition."
- Scaling doesn't solve hot partitions because it's meant to solve an **insufficient resource issue**. A hot partition creates a **resource distribution issue**.
- Engineers use the following metrics to detect hot partitions:
	- Request rate
	- CPU usage
	- Queue length
	- Latency
	- Storage growth
	- Consumer lag (Kafka)
	- Read/write throughput
- One partition behaving differently than the others is an indicator of a hot partition.
- Hot partitions can be prevented by using:
	1. **Better Partition Key:** Instead of using `Country`, use `CustomerID`. This creates a more even traffic distribution.
	2. **Composite Key:** Instead of `CustomerID`, use `CustomerID + Date`. Now, one large customer's events are spread over multiple partitions based on time.
	3. **Hash Partitioning:** Instead of `CustomerID`, use `Hash(CustomerID)` as the partition key. The hash function spreads data more evenly across partitions. **This is one of the most common strategies in distributed databases**.
	4. **Key Salting:** 
		- Suppose Customer 42 is extremely active. Instead of storing based on `CustomerID = 42`, store data based on:
			```
			42-0
			
			42-1
			
			42-2
			
			42-3
			```
		- This spreads requests across multiple partitions.
		- At a later point, the results are aggregated. This technique is called **salting**. It's commonly used in Spark and other distributed processing systems.

### Tradeoffs
- Hot partitions don't just affect performance. They also affect:
	- Auto-scaling
	- Resource utilization
	- Cost
	- Reliability
	- Availability
- An overloaded partition often becomes the first point of failure.
- The goal of partitioning isn't just to distribute data—it's to distribute work.

### Common Examples
- Imagine Amazon Prime Day:
	- Suppose all orders are partitioned by `Product Category`.
	- If one category is especially popular, most of the web traffic will flow towards that partition, making it a hot partition.
	- If that partition fails under the load, customers will think the entire site is broken, even though the other partitions are perfectly healthy.
- Hot partitions appear everywhere, including:
	- Kafka
	- Spark
	- Hive
	- BigQuery
	- Snowflake
	- DynamoDB
	- Cassandra
	- HBase

### What is Kafka?
- Apache Kafka is an open-source, distributed event streaming platform used to collect, store, and process large volumes of real-time data. Its main functions are publishing and subscribing to data streams, storing data safely on disk, and processing real-time feeds quickly.
- **Key Components:**
	- **Producers**: Apps or devices that send data messages into Kafka.
	- **Topics**: Categories or channels where Kafka stores related messages.
	- **Brokers**: Individual servers that form a Kafka cluster to hold and manage the data.
	- **Consumers**: Apps or services that read and process the messages from topics.
- **Why People Use It:**
	- **Real-Time Speed**: Handles massive data flows with very low delay.
	- **Data Replay**: Keeps saved data for a set time, so systems can re-read or replay past events.
	- **Scalability**: Spreads data across many computers to grow easily as traffic increases.

### Hot Partition vs. Poor Partition Key
- The distinction is subtle.
	- A poor partition key, such as `Gender`, is bad from the **beginning**.
	- A hot partition, such as `CustomerID`, is normally good. When one customer becomes unusually active, it creates a hot partition even though there was nothing wrong with the partition key before.

### Common Interview Questions
1. Your team partitions a table by **CustomerID**. One enterprise customer begins generating **60% of all requests**. Performance degrades significantly. Would you change the partition key? Why or why not?
> 	I wouldn't immediately change the partition key because CustomerID is still a reasonable choice for most customers. I'd first determine whether the skew is temporary or persistent. If it's a long-term issue caused by a consistently high-volume customer, techniques like key salting or redesigning the partitioning strategy could help distribute the workload more evenly.
	- When a hot partition occurs, the answer isn't to immediately introduce key salting or hashing. You must first determine if the traffic anomaly is temporary or persistent.
2. An interviewer says, "Hash partitioning guarantees you won't have hot partitions." Do you agree? Why or why not?
> 	Hash partitioning more evenly distributes keys among partitions, but it doesn't necessarily guarantee you won't have hot partitions.
	- Hashing distributes **keys**, not **traffic**. One unusually active customer can be assigned to the same partition **with or without** hashing. If the hashing is based purely on something like `CustomerID`, hashing doesn't solve the **traffic distribution** problem.
	- Hashing reduces the probability of hot partitions caused by uneven key distribution, but it cannot prevent hot partitions caused by skewed workloads.
3. If adding more servers doesn't improve throughput, what would you investigate?
	- A great answer includes:
		- Uneven partition sizes
		- Hot partitions
		- Data skew
		- Uneven traffic distribution
		- Load imbalance
	- That demonstrates you understand that **horizontal scaling only works when work is distributed effectively**.

## 4. Sharding vs. Partitioning

- Sharding and partitioning are often used interchangeably, but they have different meanings.
- Imagine you have a database that grows to 10 billion rows in size.
	- One server can no longer:
		- Store all the data
		- Handle all the traffic
		- Process all the queries
	- You need to split the data somehow. The question becomes, "How do you split it?"
		- Partitioning and sharding solve similar problems, but at different levels.
- Partitioning:
	- Dividing a table into smaller pieces called partitions.
	- The database decides which partition stores each row.
	- Everything can still live on **one database server**.
	- Partitioning is mainly about:
		- Organization
		- Faster queries
		- Easier maintenance
	- Partitioning does not necessarily concern scaling across different machines.
- Sharding:
	- Splits data across **multiple servers**, instead of just one server.
	- Distributes the **workload** across multiple machines. This is horizontal scaling.
	- Sharding is implemented because one server has limits. Eventually, you'll hit:
		- CPU limits
		- Memory limits
		- Storage limits
		- Network limits
	- Adding partitions inside one server does not resolve these issues. Adding more servers does.
- Every shard contains multiple partitions. Not every partition is a shard.
	- The hierarchy looks like this: `Cluster > Shards > Partitions`.
	- A shard may internally partition its own data.
	- For example, suppose you have three shards. All shards partition data based on month, but each shard only stores part of the customer base.
	- Large production systems often combine partitioning with sharding.

### Common Sharding Keys
- Just like partition keys:
	- CustomerID
	- Region
	- AccountID
	- TenantID
- Common Example:
	```
	Hash(CustomerID)
	
	↓
	
	Shard
	```

### Tradeoffs
- Partitioning:
	- Pros:
		- Simpler
		- Easier to manage
		- Faster queries
		- Lower operational complexity
	- Cons:
		- Limited by one machine
		- Can't scale indefinitely
- Sharding:
	- Pros:
		- Nearly unlimited horizontal scaling
		- More storage
		- More throughput
		- Better fault isolation
	- Cons:
		- More operational complexity
		- Cross-shard queries are harder
		- Rebalancing shards can be expensive
		- Distributed transactions become more difficult

### Common Examples
- **Amazon Example:**
	- Suppose all customer accounts lived on one database.
	- Eventually, as the customer base grows, the server can't keep up with the demand.
	- Instead:
		```
		Shard 1
		
		North America
		```
		```
		Shard 2
		
		Europe
		```
		```
		Shard 3
		
		Asia
		```
		- Within each shard, orders can still be **partitioned** by month.
		- That's sharding and partitioning working together. They're not mutually exclusive.

### Data Engineering Perspective
- You'll encounter both concepts depending on the technology:
	- **BigQuery, Snowflake, Hive, Spark, Delta Lake**: You'll primarily think in terms of **partitions**, because they optimize how analytical data is stored and scanned.
	- **Cassandra, DynamoDB, MongoDB, Vitess, CockroachDB**: You'll often think about **sharding**, because data is distributed across many machines to scale reads and writes.
- As a Data Engineer, it's useful to recognize whether you're optimizing **query efficiency** (partitioning), **system scalability** (sharding), or both.
- Helpful Analogy:
	```
	Data
	
	↓
	
	Partitioning
	(Split within a database)
	
	↓
	
	Sharding
	(Split across databases/servers)
	```
	- When some asks "What's the difference?" You can explain:
> 	Partitioning divides data into logical pieces to improve organization and query performance, while sharding distributes those pieces across multiple servers to scale storage and throughput.

### Common Interview Questions
1. If partitioning improves query performance, why not just keep adding more partitions instead of sharding?
> 	Partitioning organizes data and reduces the amount scanned, but it doesn't remove the physical limits of a single machine. Once CPU, memory, storage, or network become bottlenecks, you need sharding to distribute both the data and the workload across multiple servers.
	- This answer demonstrates you understand the different problems each technique solves.
2. Can a sharded database still have hot partitions?
	- Yes. Imagine a shard containing 100 customers. If one customer generates 95% of traffic, one partition within that shard will still become overloaded.
	- Sharding solved the capacity issue. It didn't automatically solve the workload imbalance.
		- Think about a kitchen with multiple stoves.
		- Each stove represents a shard. On each stove, a stove top or burner represents a partition.
		- If one burner is turned up to the maximum setting, it's using up most of the gas / electricity being supplied to the stove, leaving few resources left to heat up the remaining burners.
		- Having multiple stoves solved the capacity issue. Within each stove, the gas / electricity could still be distributed unevenly.
	- You still need good partition keys and good workload distribution.
3. Your PostgreSQL database has grown to **3 TB**, but CPU, memory, and storage are all still well within the limits of a single server. Queries filtering by date have become slow. Would you recommend **partitioning**, **sharding**, or both? Why?
> 	Since the server still has sufficient CPU, memory, and storage, I wouldn't introduce the operational complexity of sharding. The bottleneck is query performance, not machine capacity. I'd partition the table by a key that aligns with common access patterns, such as `OrderDate`, allowing the database to prune irrelevant partitions and reduce the amount of data scanned.
	- Partitioning doesn't necessarily evenly distribute data. For example, partitioning by order date can cause uneven partitions if one month had more orders than another month. The pruning is the real benefit of partitioning.
4. Your company's customer database is now **50 TB**, and a single server can no longer store all the data or handle peak traffic. Would partitioning alone solve the problem? If not, what would you recommend, and why?
> 	Partitioning would not solve the issue even if data was evenly distributed among all partitions. The combined load of all partitions would still be an issue. Scaling horizontally and introducing sharding would increase storage and compute capacity. Partitioning within each shard can further improve query performance and help distribute the workload within that shard.
	- This wording acknowledges that data skew is still possible.
5. Your analytics queries are slow. Should you normalize, partition, or shard?
	- It depends on what's causing the slowdown:
		- **Too much data being scanned?** → Partitioning.
		- **One server has reached its hardware limits?** → Sharding.
		- **Too many expensive joins?** → Consider denormalization for analytical workloads.
		- **Frequent updates and data consistency are the priority?** → Normalize.

## 5. Serialization & Data Formats

### Serialization
- Imagine you have a simple object stored in memory:
	```
	Customer
	
	Name: Alice
	Age: 32
	Prime: True
	```
	- Your program understands this object because it exists in memory as language-specific data structures.
	- What happens if you want to:
		- Send it over the network?
		- Save it to disk?
		- Publish it to Kafka?
		- Write it to S3?
		- Send it to another service written in Java?
	- The other system can't directly read your application's memory. You need a common representation. This is what serialization does.
- Serialization is the process of converting an in-memory object into a format that can be stored or transmitted.
	- Later, another program performs the reverse operation. Deserialization:
		```
		Object
		
		↓
		
		Serialization
		
		↓
		
		Bytes
		
		↓
		
		Network / Disk
		
		↓
		
		Bytes
		
		↓
		
		Deserialization
		
		↓
		
		Object
		```
- For example, suppose you're writing Python:
	```python
	customer = {
	    "name": "Alice",
	    "age": 32
	}
	```
	- Inside Python, this is a dictionary.
	- A Java application doesn't understand Python dictionaries.
	- Neither does Kafka.
	- Neither does a file.
	- Instead, the Python dictionary is serialized to something like:
		```json
		{
		  "name": "Alice",
		  "age": 32
		}
		```
	- Now, every system that understands JSON can understand the Python dictionary.
- Computers ultimately store all data as bytes. Even JSON-serialized objects become bytes.
	- Networks transmit bytes.
	- Disks store bytes.
	- Memory caches store bytes.
	- Serialization converts high-level objects into bytes.
	- Serialization also provides a **language-agnostic** representation of data, making it possible for systems that operate using different programming languages to communicate.
- You'll encounter serialization almost everywhere:
	- REST APIs
	- Kafka
	- RabbitMQ
	- SQS
	- Files
	- S3
	- Databases
	- Redis
	- Spark
	- Flink
- Anytime data moves between systems, serialization is involved.
- **It's important to recognize that serialization and encoding are not synonymous**.
	- Serialization answers: How do I convert this object into a transferable or storable representation?
	- Encoding answers: How are those bytes represented?
	- For example:
		```
		Object
		
		↓
		
		JSON Serialization
		
		↓
		
		UTF-8 Encoding
		
		↓
		
		Bytes
		```

### Serialization Formats
- Not all formats are created equal. Good serialization formats aim to strike a balance between:
	- Small Size:
		- Faster network transfer
		- Lower storage cost
		- Better cache utilization
	- Fast Serialization:
		- Writes objects quickly
		- Important for high-throughput systems
	- Fast Deserialization:
		- Reads objects quickly
		- Important when millions of messages are read every second
	- Human Readability:
		- Sometimes, engineers need to inspect data.
		- Readable formats allow for easier debugging
		- Compact binary formats allow for better performance
		- Tradeoff is necessary
	- Schema Support:
		- Can the format accurately describe:
			- Field names
			- Types
			- Optional fields
- Binary formats sacrifice readability for small size.
- **Data Engineering Example:**
	- Suppose every event is 2KB and you process 2 billion events /  day.
	- That's roughly 4TB in storage and network transfer every day.
	- When you switch to a binary format that reduces the event size by 40%, you save about 1.6TB.
	- At large scale, serialization choices have real operational and financial impact.

### Tradeoffs
- Human-readable formats
	- Pros:
		- Easy debugging
		- Easy APIs
		- Easy development
	- Cons:
		- Larger
		- Slower
		- More storage
- Binary formats:
	- Pros:
		- Small
		- Fast
		- Efficient
	- Cons:
		- Difficult to inspect manually
		- Usually require schemas or tooling

### Common Formats
- JSON:
	- Human-readable
	- Very common
	- Large
	- Flexible
- Avro:
	- Binary
	- Schema-based
	- Excellent for streaming
	- Supports schema evolution
	- Data pipelines
	- Kafka
	- Analytics
- Protobuf:
	- Binary
	- Very compact
	- Very fast
	- Common in microservices, RPC, and gRPC
- Parquet:
	- Columnar storage
	- Optimized for analytics
	- Not intended for message passing
	- **Parquet is not primarily meant for network communication. It's a storage format optimized for analytical queries.**

### Common Interview Questions
1. Why do distributed systems use serialization?
> 	Because applications store objects in language-specific memory structures, but networks and storage systems operate on bytes. Serialization converts those in-memory objects into a portable format that can be transmitted, stored, and reconstructed by other systems, even if they're written in different programming languages.
2. Why can't two microservices simply send Java objects to each other?
> 	Java objects only exist in the memory of the process that created them. Since networks and storage systems operate on bytes, the object must first be serialized into a portable byte representation. The receiving service can then deserialize those bytes back into a Java object. This also enables interoperability if other services are written in different languages.
3. Suppose you're processing **billions of events per day**. Why might a binary serialization format be preferred over a human-readable format like JSON?
> 	Binary formats are generally preferred because they're more compact and require less network bandwidth and storage. They also reduce disk I/O and improve cache utilization. Since binary formats contain less data to parse, serialization and deserialization are often faster as well, allowing the system to process more events per second.
4. Why not use JSON everywhere?
> 	JSON is human-readable and easy to debug, which makes it a great choice for APIs and development. At large scale, however, binary formats reduce storage requirements, network bandwidth, and CPU overhead, improving overall throughput.

## 6. JavaScript Object Notation (JSON)

- Imagine two applications need to exchange data. One is written in Java and the other is written in Python.
- The two applications need a serialization format that is:
	- Human-readable
	- Language-independent
	- Easy to generate
	- Easy to parse
- JSON satisfies all of these requirements.
- JSON is popular for several reasons:
	1. Human Readable:
		```json
		{
		  "customerId": 123,
		  "name": "Alice"
		}
		```
		- You can immediately understand it.
		- No special tools required.
		- This makes debugging much easier.
	2. Language Independent:
		- JSON is understood by every major programming language, including: Python, Java, Go, JavaScript, C#, and Rust.
		- This makes it ideal for communication between services.
	3. Flexible:
		- Suppose version 1 sends:
			```json
			{
			  "name": "Alice"
			}
			```
		- Tomorrow, version 1 sends:
			```json
			{
			  "name": "Alice",
			  "email": "alice@email.com"
			}
			```
		- Many JSON parses simply ignore fields they don't care about. This flexibility makes development easy.
	4. Great for APIs:
		- This is why REST APIs almost always use JSON.
		- Request:
			```http
			GET /customers/123
			```
		- Response:
			```json
			{
			  "customerId": 123,
			  "name": "Alice",
			  "prime": true
			}
			```
		- Easy for both humans and web applications.

### Problems
JSON's enhanced readability comes at a cost:
1. Large Messages:
	```json
	{
	  "customerId": 123,
	  "customerName": "Alice",
	  "customerAge": 32
	}
	```
	- Field names are repeated with each message.
	- When millions or billions of messages are sent, those repeated field names consume a surprising amount of bandwidth and storage.
	- Binary formats avoid this overhead.
2. Pasing Cost:
	- Before your application can use JSON, it must be capable of parsing text, which is expensive.
	- Binary formats are often faster to deserialize because they are already encoded in a machine-friendly representation.
3. Weak Typing:
	- JSON supports a limited set of data types, such as generic numbers and strings.
	- JSON cannot tell the difference between an int, long, or float.
	- JSON cannot tell the difference between a string and a date or timestamp.
	- Applications must agree on an interpretation.
4. No Built-in Schema:
	- Suppose one service sends:
		```json
		{
		  "customerId": 123
		}
		```
	- Another service sends:
		```json
		{
		  "customerID": 123
		}
		```
	- JSON by itself won't catch the mistake (different keys for the same value).
	- Without schema validation, these errors can silently propagate through a system.

### Data Engineering Perspective
- Suppose you're processing 5 billion events every day. Each JSON message contains `customerID` repeated millions or billions of times. That's a lot of unnecessary data.
- At this scale:
	- Storage costs rise.
	- Network costs rise.
	- Parsing costs rise.
- This is why large data pipelines often choose formats such as **Avro** or **Protobuf** instead.

### Tradeoffs
- JSON is excellent when:
	- Humans need to inspect data.
	- Ease of integration matters.
	- Data volumes are moderate.
	- Public APIs are involved.
	- Development speed is more important than absolute efficiency.
	- Examples:
		- REST APIs
		- Configuration files
		- Small event payloads
		- Debugging
		- Logging
- JSON struggles in environments such as:
	- Kafka pipelines
	- Event streaming
	- Data lakes
	- Petabyte-scale storage
	- High-frequency messaging
	- In these environments, storage and netowrk bandwidth costs become significant.
- Suppose you're sending:
	```json
	{
	  "customerId": 123,
	  "name": "Alice",
	  "country": "USA"
	}
	```
	- Every message repeats `customerId`, `name`, and `country`.
	- One million messages send one million copies of this field name.
	- Binary formats usually store this metadata much more efficiently.
- Pros:
	- Human-readable
	- Easy to debug
	- Language-independent
	- Universally supported
	- Flexible
- Cons:
	- Larger messages
	- Slower parsing
	- No built-in schema enforcement
	- Weak typing
	- Less efficient for large-scale data processing

### JSON vs. In-Memory Objects
- Remember, your application works with objects.
- JSON is simply one way of serializing that object into text:
	```
	Object
	
	↓
	
	JSON
	
	↓
	
	Bytes
	
	↓
	
	Network
	
	↓
	
	JSON
	
	↓
	
	Object
	```

### Real-World Example
- Imagine a developer makes a request to Amazon's Product API:
	```http
	GET /products/123
	```
	- JSON is an excellent choice for this workflow because the payload is easy to inspect, and the request volume for an individual client is manageable.
	- JSON is self-describing because the field names are included in the payload.
	- Binary formats often separate the schema from the data, making them smaller but less human-readable.
- Now imagine Amazon's internal analytics pipeline processing billions of clickstream events per day.
	- Using JSON for every event would consume far more storage, network bandwidth, and CPU than a compact binary format.
- Different workloads lead to different format choices.

### Common Interview Questions
1. Why is JSON still so widely used if binary formats are faster?
> 	Because developer productivity often matters more than raw performance. JSON is human-readable, universally supported, and easy to debug, making it an excellent choice for APIs and integrations. For high-throughput systems where bandwidth, storage, and CPU efficiency become critical, binary formats are usually preferred.
2. You're designing a **public REST API** that third-party developers will integrate with. Would you choose JSON or a binary format like Protobuf? Why?
> 	I would choose JSON because it's human readable and easier to debug than binary formats, making it an ideal choice for an API used mainly by third-party developers. JSON is also universally supported, making it easier for developers using different programming languages to integrate with the API.
3. Your Kafka pipeline now processes **8 billion events per day**, and network bandwidth and storage costs have become significant. Would you continue using JSON? Why or why not?
> 	At eight billion events per day, operational efficiency becomes much more important than human readability. I'd likely move to a binary format such as Avro or Protobuf because they produce smaller payloads, reducing storage costs and network bandwidth. They're also generally faster to serialize and deserialize, improving throughput. While debugging becomes more difficult, the scalability benefits outweigh that tradeoff at this volume.
- In engineering interviews, the strongest answers typically begin with "it depends...", provided you elaborate further. For example:
	1. **Normalization:** It depends on read / write patterns.
	2. **Partition Keys:** It depends on access patterns.
	3. **Sharding:** It depends on whether the bottleneck is query performance or machine capacity.
	4. **Serialization:** It depends on whether developer productivity (JSON) or operational efficiency (Avro or Protobuf) is more important.

## 7. Avro

- If JSON is optimized for **humans**, Avro is optimized for **machines processing enormous amounts of data**.
	- Instead of storing field names in every message, like JSON, Avro stores them once in a **schema**.
	- Once the field names are stored in the schema, it only sends the values.
	- This works because the receiver **already knows the schema**. There's no need to repeat field names in every message.
- A schema is simply a contract describing the structure of the data.
	- Example:
		```json
		{
		  "type": "record",
		  "name": "Customer",
		  "fields": [
		    {
		      "name": "customerId",
		      "type": "long"
		    },
		    {
		      "name": "name",
		      "type": "string"
		    },
		    {
		      "name": "country",
		      "type": "string"
		    }
		  ]
		}
		```
		- This tells every application:
			- What fields exist
			- Their names
			- Their types
		- Once everyone agrees on a schema, the actual messages become much smaller. This creates huge storage and bandwidth savings.
- Unlike JSON, Avro **isn't meant to be read by humans**. Instead:
	```
	Customer Object
	
	↓
	
	Avro Serialization
	
	↓
	
	Compact Binary Data
	```
	- This is much smaller and easier to transfer. Even when two services are running on the same machine and use the same programming language, they're both running in separate processes, so the data still needs to be transferred.
- Avro is popular in Data Engineering because Data Engineers are constantly moving enormous amounts of structured and unstructured data.
	- Think about:
		- Kafka
		- Spark
		- Hadoop
		- Hive
		- Flink
	- These systems don't care if humans can read every message. They only care about:
		- Throughput
		- Storage
		- Network bandwidth
		- Compatibility
	- Avro performs well in all four categories.

### Schema Evolution
- Schema evolution is Avro's superpower. Suppose Version 1 contains:
	```
	Customer
	
	customerId
	name
	```
- Later the business wants:
	```
	Customer
	
	customerId
	name
	email
	```
- Old applications continue working when the schema updates because Avro supports schema evolution.
	- Old consumers **ignore new fields**.
	- New consumers can provide **default values** for missing fields.
	- This allows produces and consumers to evolve independently.
- Example:
	- Producer V2:
		```
		customerId
		name
		email
		```
	- Consumer V1:
		```
		customerId
		name
		```
		- Consumer V1 will only understand `customerId` and `name`.
		- Nothing breaks. The consumer simply ignores `email`.
		- No deployment coordination is required, which very valuable in large organizations.

### JSON vs. Avro
- JSON:
	```json
	{
	  "customerId": 123,
	  "name": "Alice"
	}
	```
	- Pros:
		- Human readable
		- Easy debugging
		- Flexible
	- Cons:
		- Large
		- Field names repeated
		- Weak typing
		- No built-in schema
- Avro:
	- Binary
	- Pros:
		- Small
		- Fast
		- Strong schema
		- Excellent schema evolution
		- Great for streaming
	- Cons:
		- Not human readable
		- Requires schema management
		- Harder to inspect manually
- Why don't we just compress JSON?
	- Compression reduces size.
	- However, it doesn't other issues:
		- schema enforcement
		- strong typing
		- schema evolution
		- parsing efficiency
	- **Avro still has advantages beyond messaage size**.
- Avro is sometimes confused with compressed JSON. It's not. Key features that Avro provides include:
	- Binary encoding
	- Schemas
	- Strong typing
	- Schema evolution
	- These are all **architectural features**, not compression.

|Feature|JSON|Avro|
|---|---|---|
|Human-readable|✅|❌|
|Binary|❌|✅|
|Compact|❌|✅|
|Built-in schema|❌|✅|
|Strong typing|Limited|✅|
|Schema evolution|Poor|✅|
|Great for APIs|✅|Sometimes|
|Great for Kafka|Okay|✅|
- Avro doesn't just try to improve JSON by making it slightly faster. It was designed with a different set of priorities:
	- Machines instead of humans
	- Data pipelines instead of APIs
	- Long-term compatibility instead of flexibility
- Understanding those design goals is much more important than memorizing the feature list.

### Data Engineering Example
- Imagine the following:
	```
	Order Service
	
	↓
	
	Kafka
	
	↓
	
	Inventory
	
	↓
	
	Fraud Detection
	
	↓
	
	Analytics
	```
	- Every service reads the same events.
	- With JSON:
		- Large messages.
		- Repeated field names.
		- More bandwidth.
	- With Avro:
		- Compact binary messages.
		- Shared schema.
		- Consumers know exactly what to expect.
		- This is why Avro and Kafka are commonly paired.
- **Real-World Example:**
	- Suppose Uber generates millions of ride events.
	- Every event contains:
		- Driver ID
		- Rider ID
		- Timestamp
		- GPS coordinates
		- Fare
	- The structure rarely changes.
	- Avro would be a great choice here because:
		- Every event has the same schema
		- Billions of events are transmitted
		- Storage efficiency matters
		- Backward compatibility matters

### Common Interview Questions
1. Why is Avro commonly used with Kafka?
> 	Kafka often carries large volumes of structured events. Avro's compact binary representation reduces storage and network costs, while its schema support and schema evolution capabilities allow producers and consumers to evolve independently without breaking existing applications.
2. Your Kafka pipeline currently uses JSON. Network bandwidth and storage costs are becoming a concern, but the event structure is well-defined and changes infrequently. Would you recommend switching to Avro? Why?
> 	Yes, I would still recommend switching to Avro in this case. The fact that the event structure is well-defined and fairly static makes it perfect for Avro. The producer and consumer agree on the common schema and messages are sent in compressed, binary format. If the schema does change, Avro allows the producer and consumer to evolve independently.
3. Your manager says: **"JSON is easier to debug, so we should always use JSON instead of Avro."** How would you respond?
> 	I would mention that switching to Avro offers a significant reduction in storage and network bandwidth costs. JSON is definitely easier to debug, and that's one of its biggest strengths. However, debugging is only one requirement. If the system processes billions of events with a stable schema, the savings in storage, bandwidth, and CPU often outweigh the reduced readability. Engineers can still inspect Avro data using tooling when necessary. This would be preferable compared to sending large JSON messages between services, especially if the schema is well-defined and fairly static.
4. If Avro requires a schema, isn't that less flexible than JSON?
	- Yes, that's often why Avro is considered a better choice. At some point, flexibility becomes a liability.
	- Imagine one user sends:
		```json
		{
		  "customerId": 123
		}
		```
	- While another user sends:
		```json
		{
		  "customerID": 123
		}
		```
	- JSON accepts both. Your analytics pipeline quietly breaks.
	- Avro wouldn't accept both. The schema catches the inconsistency before it propagates.
	- In many production systems, consistency is favored more than flexibility. This is one of the primary reasons Data Engineers like schema-based platforms.

## 8. Protocol Buffers (Protobuf)

- Imagine Google has thousands of services. Every request between services travels over the network. For example:
	```
	Web Service
	
	↓
	
	Authentication Service
	
	↓
	
	Recommendation Service
	
	↓
	
	Billing Service
	```
	- Every millisecond matters.
	- Every byte matters.
	- Google wanted a serialization format with the following properties:
		- Extremely compact
		- Extremely fast
		- Strongly typed
		- Language independent
	- That's why they created Protocol Buffers.
- Avro and Protobuf are similar in that both are:
	- Binary
	- Schema-based
	- Compact
	- Cross-language
- Despite these similarities, Avro and Protobuf optimize different workloads.

### Protobuf Schema
- A Protobuf schema looks like:
	```proto
	message Customer {
	  int64 customer_id = 1;
	  string name = 2;
	  string country = 3;
	}
	```
	- The numbers assigned to each key are **field tags**, not values.
		- Instead of transmitting `customer_id`, Protobuf transmits `1`.
		- Instead of transmitting `name`, Protobuf tansmits `2`.
		- Instead of transmitting `country`, Protobuf transmits `3`.
		- These tiny numbers make the serialized data extremely compact.

### Generated Code
- Another major difference between Avro and Protobuf is that Protobuf doesn't work directly with schemas. Instead:
	```
	Schema
	
	↓
	
	Code Generator
	
	↓
	
	Java Class
	
	Python Class
	
	Go Struct
	```
	- Now your application works with native language objects. For example:
		```java
		Customer customer =
		    Customer.newBuilder()
		        .setCustomerId(123)
		        .setName("Alice")
		        .build();
		```
		- Developers love this because its type-safe and IDE-friendly.
- Protobuf is fast for several reasons:
	- Binary encoding
	- Small messages
	- Efficient parsing
	- Generated code
	- Minimal allocations
- All of these benefits reduce CPU Utilization.
- This is why Protobuf is used in low-latency systems.

### Common Use Cases
- Protobuf is most famously used in gRPC APIs. This is because these types of APIs prioritze:
	- Low latency
	- High throughput
	- Efficient communication
- This is exactly what Protobuf was built for.

### Protobuf vs. JSON
- JSON:
	```json
	{
	  "customerId": 123,
	  "name": "Alice"
	}
	```
	- Easy to read.
- Protobuf:
	- Binary bytes.
	- Humans can't read it.
	- Applications deserialize it instantly.

### Protobuf vs. Avro
- Commonalities
	- Both are binary.
	- Both have schemas.
- Avro is optimized for:
	- Kafka
	- ETL
	- Data pipelines
	- Analytics
	- Schema evolution
- Protobuf is optimized for:
	- RPC
	- APIs
	- Microservices
	- gRPC
	- Low latency
- Avro optimizes **data at rest and streaming events**.
- Protobuf optimizes **service-to-service communication**.

### Schema Evolution
- Protobuf supports schema evolution as well. For example:
	- Original:
		```proto
		message Customer {
		  int64 customer_id = 1;
		}
		```
	- Later:
		```proto
		message Customer {
		  int64 customer_id = 1;
		  string email = 2;
		}
		```
	- Older applications simply ignore unknown fields. Very similar to Avro.
	- However, Avro generally offers more flexibility for evolving schemas in long-lived data pipelines. That's one reason it's so popular with Kafka.

### Tradeoffs
- Avro's schema evolution model is generally better suited to long-lived datasets.
	- Imagine storing 20 petabytes of analytical data.
	- Engineers years later may need to read historical records.
- Protobuf shines when systems communicate in real time.
- **Real-World Example:**
	- Suppose a customer clicks 'Buy Now.' The request flows through:
		```
		Checkout
		
		↓
		
		Inventory
		
		↓
		
		Payment
		
		↓
		
		Fraud Detection
		```
	- Every service call should be:
		- Tiny
		- Fast
		- Efficient
	- Protobuf is an excellent fit.
	- Now imagine:
		- Every completed purchase becomes an analytics event.
		- Those events are written into Kafka.
		- Eventually loaded into a data lake.
		- That's where Avro often becomes the better choice.
- Protobuf:
	- Pros:
		- Extremely compact
		- Very fast
		- Strong typing
		- Generated code
		- Great developer experience
		- Excellent for RPC
	- Cons:
		- Not human-readable
		- Less convenient for analytics pipelines
		- Requires code generation

| Format   | Primary Goal                      |
| -------- | --------------------------------- |
| JSON     | Human readability                 |
| Avro     | Data pipelines & schema evolution |
| Protobuf | Low-latency service communication |

### Data Engineering Perspective
- As a Data Engineer, you'll likely encounter both formats. For example:
	```
	Microservices
	
	↓
	
	Protobuf
	
	↓
	
	Kafka
	
	↓
	
	Avro
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	```
	- Each format is chosen for the workload it serves.
	- Protobuf is optimized for service-to-service communication.
	- Avro is optimized for Kafka.
	- Parquet is optimized for storage.

### Common Interview Questions
1. Why does Google use Protobuf internally?
> 	Because Google's services communicate constantly over the network. Protobuf minimizes message size and serialization overhead, reducing latency and CPU usage while providing strong typing and excellent support across many programming languages.
2. What is the primary difference between Protobuf and Avro?
> 	Protobuf was designed to optimize synchronous service communication where low latency and efficient serialization are critical. Avro was designed to optimize large-scale data movement and schema evolution in data pipelines.
3. Your team is building a **high-frequency gRPC service** that processes tens of thousands of requests per second between microservices. Would you choose Avro or Protobuf? Why?
> 	I would choose Protobuf because this is a high-frequency, low-latency gRPC service where serialization speed and message size directly impact response times. Protobuf produces compact binary messages, uses generated code for efficient serialization and deserialization, and is the standard format for gRPC. While Avro is also compact, its primary strength is schema evolution for long-lived data pipelines rather than synchronous service communication.
	- This answer specifically connects Protobuf to:
		- low latency
		- high throughput
		- serialization speed
		- gRPC
4. Your company stores **years of Kafka events** in a data lake, and the event schema changes occasionally as the business evolves. Would you recommend Protobuf or Avro?
> 	I would recommend Avro because the data will be stored and consumed over many years, and the schema is expected to evolve as the business changes. Avro's schema evolution capabilities allow producers and consumers to change independently while maintaining compatibility with historical data. Although Protobuf also supports schema evolution, it was primarily designed for efficient service-to-service communication rather than long-lived analytical datasets.
	- This answer demonstrates that you know **why** schema evolution is important in a data lake.
5. Could a company use JSON, Protobuf, and Avro at the same time?
	- Yes. For example:
		```
		Mobile App
		
		↓
		
		REST API
		
		↓
		
		JSON
		
		↓
		
		Microservices
		
		↓
		
		Protobuf
		
		↓
		
		Kafka
		
		↓
		
		Avro
		
		↓
		
		Parquet
		
		↓
		
		Data Warehouse
		```

- This table acts as a helpful guide towards deciding which format to implement based on the primary use case:

| Question                  | Best Answer |
| ------------------------- | ----------- |
| Public REST API           | JSON        |
| Internal gRPC API         | Protobuf    |
| Kafka Event Stream        | Avro        |
| Data Lake Storage         | Avro        |
| Configuration File        | JSON        |
| Human Debugging           | JSON        |
| Low Latency Microservices | Protobuf    |
| Analytics Pipelines       | Avro        |
- You're choosing the format based **workload**, not features.

## 9. Parquet

- Almost every modern analytics platform is built around Parquet, including:
	- Spark
	- Databricks
	- Delta Lake
	- Iceberg
	- Athena
	- BigQuery
	- Snowflake
	- Redshift Spectrum
- Imagine a table with one billion customer records:
	```
	CustomerID | Name | Country | Age | Salary
	```
- Now suppose an analyst runs a query, such:
	```sql
	SELECT Country
	FROM Customers;
	```
	- Ideallym only the `Country` column of the file should be read to reduce I/O.
	- Unfortunately, traditional storage formats don't work this way.
- **Row-Oriented Storage:**
	```
	ID   Name    Country    Age
	
	1    Alice   USA        30
	2    Bob     Canada     45
	3    Carol   USA        28
	```
	- Rows are stored together.
	- When an analyst queries `SELECT Country`, the database still needs to read the other three columns for **every row**.
	- Most of that work is wasted.
- **Column-Oriented Storage:**
	```
	Country
	
	USA
	Canada
	USA
	```
	- Instead of storing data grouped by rows, data is grouped by columns.
	- When analyst queries `SELECT Country`, **only the `Country` column is read**. Nothing else.
	- This is a massive reduction in disk I/O.
- **Better Compression:**
	- Parquet has another huge advantage. Columns usually contain similar values. Repeated values compress very well.
	- With row-oriented storage, each row almost always contains a mix of data types. This doesn't compress as well as column-oriented storage.
- **Predicate Pushdown:**
	- Suppose someone runs the following query:
		```sql
		SELECT *
		FROM Sales
		WHERE Country = 'USA';
		```
		- Parquet stores statistics about groups of rows. It will know whether `USA` exists in certain sections of the data set.
		- Using this knowledge, it is able to skip these sections entirely. This is called **predicate pushdown**.
		- This reduces disk I/O even further.
- **Compression Formats:**
	- JSON: Optimized for humans.
	- Avro: Optimized for streaming.
	- Protobuf: Optimized for RPC.
	- Parquet: Optimized for:
		- Analytics.
		- Different workloads.
		- Different formats.
- **Why not use Parquet for APIs?**
	- Parquet is optimized for **scanning large data sets**, not random record lookup.
	- If someone calls a REST API's `GET` endpoint looking for a single customer, Parquet format is not optimized for this workload.
- **Data Engineering Example:**
	```
	Kafka
	
	↓
	
	Avro Events
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Data Lake
	
	↓
	
	Athena
	
	↓
	
	Dashboard
	```
	- Avro transports events.
	- Parquet stores analytics.
	- These are different jobs.

### Tradeoffs
- Pros:
	- Excellent compression
	- Reads only needed columns
	- Predicate pushdown
	- Great for analytics
	- Lower storage costs
	- Faster analytical queries
- Cons:
	- Not human readable
	- Poor for transactional workloads
	- Poor for random single-record access
	- More expensive to update individual rows
- **Real-World Example:**
	- Imagine Amazon stores every purchase ever made. An analyst queries:
		```sql
		SELECT
		SUM(Sales)
		FROM Purchases
		WHERE Year = 2026;
		```
		- Parquet only reads the `Sales` and `Year` columns. Nothing else.
		- JSON reads everything.
- **Important Distinction:**
	- Avro and Parquet are **not** competitors.
	- Avro moves data, while Parquet stores data. They work together.
		```
		Application
		
		↓
		
		Avro
		
		↓
		
		Kafka
		
		↓
		
		Spark
		
		↓
		
		Parquet
		
		↓
		
		S3
		
		↓
		
		Athena
		```
- **Parquet vs. CSV:**
	- CSV has:
		- no data types
		- poor compression
		- row-oriented storage
		- no predicate pushdown
	- Parquet has:
		- typed columns
		- compression
		- columnar storage
		- predicate pushdown
	- This is why modern data lakes overwhelming use Parquet over CSV.

| Format   | Primary Purpose                       |
| -------- | ------------------------------------- |
| JSON     | Human-readable APIs and configuration |
| Avro     | Streaming events and schema evolution |
| Protobuf | Low-latency service communication     |
| Parquet  | Analytical storage and querying       |

### Common Interview Questions
1. Why is Parquet so popular in Data Engineering?
> 	Because analytical queries often access only a subset of columns across very large datasets. Parquet's columnar storage allows those queries to read only the required columns, while its compression and predicate pushdown significantly reduce storage and disk I/O, making analytical workloads much more efficient.
2. Your Spark job analyzes **50 TB of historical sales data**, but each query only needs five out of 120 columns. Would you store the data in JSON or Parquet?
> 	I would most likely choose Parquet over JSON. While JSON is more human readable and easier to debug, Parquet offers better compression, column-based storage, predicate pushdown. These features make analytical queries much more efficient. Since the query only accesses five of 120 columns, Parquet's columnar storage allows Spark to read only the required columns instead of scanning the entire dataset. Combined with compression and predicate pushdown, this significantly reduces disk I/O and improves query performance.
3. A teammate suggests replacing all Avro messages in your Kafka pipeline with Parquet because "Parquet is more efficient." Would you agree? Why or why not?
> 	No, I would not agree. I would mention that the two formats are not mutually exclusive. Avro is optimized for streaming data, while Parquet is optimized for storing and analyzing data. Instead, I would recommend using Avro for the streaming layer and Parquet for the storage and analytics layer.
4. Compare JSON, Avro, Protobuf, and Parquet?
	```
	Does a human read it?
	
	↓
	
	JSON
	
	----------------
	
	Is it service-to-service communication?
	
	↓
	
	Protobuf
	
	----------------
	
	Is it event streaming?
	
	↓
	
	Avro
	
	----------------
	
	Is it long-term analytical storage?
	
	↓
	
	Parquet
	```
1. Can Parquet replace a database?
	- No. Parquet is optimized for:
		- Large scans
		- Analytics
		- Batch processing
	- Databases are optimized for:
		- Transactions
		- Frequent updates
		- Random lookups
		- ACID guarantees
		- Concurrent users
2. Why doesn't Amazon store production orders directly in Parquet?
	- When a customer clicks the `Buy Now` button, the system needs to:
		- insert one order
		- update inventory
		- charge payment
		- update shipping
	- These are all **transactional** operations.

## 10. Schema Evolution

- Schema evolution isn't just about adding fields to data set or columns to a table. Schema evolution is about enabling dozens (or hundreds) of independent services to evolve without breaking each other.
- **The Problem:**
	- Imagine today, your Order event looks like:
		```
		Order
		
		orderId
		customerId
		amount
		```
	- The Producer sends:
		```json
		{
		  "orderId": 123,
		  "customerId": 456,
		  "amount": 99.99
		}
		```
		- Everything works.
	- Three months later, the Producer changes the event:
		```
		Order
		
		orderId
		customerId
		amount
		email
		```
	- The Producer now sends:
		```json
		{
		  "orderId": 123,
		  "customerId": 456,
		  "amount": 99.99,
		  "email": "alice@email.com"
		}
		```
	- What happens to consumers that haven't been updated?
		- **Bad Answer:**
			- They all crash.
			- That would be terrible. Imagine:
				```
				Checkout
				
				↓
				
				Kafka
				
				↓
				
				Inventory
				
				Fraud
				
				Billing
				
				Analytics
				
				Email
				```
				- Do all six teams make deployments at the same time? Probably not.
				- Teams at large companies make deployments independently.
				- One team may update today. Another team may update next month.
				- Schema evolution allows for this kind of flexibility.
- **The Goal:**
	- The goal isn't simply to change the schema. It's to change the schema **without breaking existing consumers**.

### Backward Compatibility
- Definition: New consumers can read old data.
- **Example:**
	- Old records:
		```
		customerId
		name
		```
	- New schema:
		```
		customerId
		name
		email
		```
	- Old records don't have an `email` field.
	- The solution is to provide a default value, such `email = null`. This allows the new application to still work.

### Forward Compatibility
- Definition: Old consumers can read new data.
- **Example:**
	- New records:
		```
		customerId
		name
		email
		```
	- Old schema:
		```
		customerId
		name
		```
	- The old consumer simply ignores the `email` field. Everything continues working.
- **Full Compatibility:**
	- Occurs when both directions work.
	- Old consumers can read new data and new consumers can read old data.
	- No breaking changes.
	- This is ideal.
- **What Changes Are Safe?**
	- **Adding an Optional Field:** Usually safe because it can be filled with a default value.
	- **Adding a Required Field:**
		- No default value.
		- Old data doesn't have the new required field.
		- New consumers fail.
		- Potentially breaking.
	- **Renaming a Field:**
		- Old consumers won't recognize the renamed field.
		- Potentially breaking.
	- **Removing a Field:**
		- Old consumers still expect the removed field.
		- Potentially breaking.
	- **Changine Data Type:**
		- Very dangerous.
		- Potentially catastrophic.

### Why Schema Evolution Matters
- Imagine Kafka stores 5 years of historical events.
- Today's consumer may still read about events produced 3 years ago.
- If schemas weren't compatible, historical data would become unreliable. This is unacceptable.

### Schema Registry
- Most Kafka-based systems introduce another component:
	```
	Producer
	
	↓
	
	Schema Registry
	
	↓
	
	Kafka
	
	↓
	
	Consumer
	```
- Instead of every service creating schemas independently, everyone registers schemas centrally.
- This schema registry validates:
	- Compatibility
	- Versions
	- Evolution rules
- This prevents the accidental introduction of breaking changes.
- **Example:**
	- A producer wants to create `Version 5` of a schema.
	- The registry checks if the version is compatible with other registered schemas.
		- If it is compatible, it accepts the new version.
		- If not, it rejects the new version.
- **Real-World Example:**
	- Imagine Amazon's purchase events. Originally:
		```
		Order
		
		orderId
		customerId
		amount
		```
		- Now, the Marketing team wants to add a `couponCode` field.
		- Billing doesn't care.
		- Fraud doesn't care.
		- Analytics does care. It starts using the new field when it is deployed.
		- No coordination required.

### Why Avro Excels
- When Avro is used for serialization, the schema travels **separately** from the data.
- This feature makes versioning more manageable.
- Comparison With JSON:
	```json
	{
	  "email": "alice@email.com"
	}
	```
	- No one verifies the name of the `email` field or its type.
	- No one verifies compatibility.
	- Custom code usually tracks versions, instead of a unified system.
	- Avro solves these problems systematically.
- **Data Engineering Example:**
	```
	Checkout
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Dashboard
	```
	- Suppose Checkout ads a `promotionCode` field to the schema.
	- Spark doesn't need it yet.
	- The dashboard doesn't need it.
	- The pipeline keeps working.
	- Six months later, the dashboard starts using the field without any issues.
	- This is what successful schema evolution looks like.

### Tradeoffs
- Schema evolution provides:
	- Independent teams
	- Backward compatibility
	- Forward compatibility
	- Long-lived datasets
	- Safer deployments
- But requires:
	- Schema management
	- Versioning
	- Registry
	- Compatibility rules

### Mental Model
```
JSON

↓

Flexible

(No enforcement)

----------------

Avro

↓

Flexible

(With schema management)

----------------

Schema Registry

↓

Safe evolution
```
- Schema enforcement isn't about making schemas rigid It's about making changes to schemas safe.

### Common Interview Questions
1. Why do Kafka teams often use Schema Registry?
> 	Because producers and consumers evolve independently. A Schema Registry centrally manages schema versions and enforces compatibility rules, preventing breaking schema changes from disrupting downstream consumers while allowing services to evolve safely and independently over time.
2. Your team wants to add a new `promotionCode` field to an existing Kafka event. Some downstream consumers haven't been updated yet. How would you introduce this change without breaking them?
> 	I would create a new schema version with `promotionCode` as an optional field (or provide a default value such as `null`). I'd register the schema with the Schema Registry so compatibility checks pass before deployment. This allows existing consumers to ignore the new field while newer consumers can begin using it.
	- Adding a field to a new schema version isn't enough by itself. You should also ensure the field is **optional** and provide an appropriate default value.
3. Another engineer wants to rename `customerId` to `clientId` because "its a better name." Would you approve this change? Why or why not?
> 	I would ask the engineer to provide better reasoning than just "it's a better a name", given the name change could cause compatibility issues with old consumers that don't recognize the new name. This could have more significant impact than a minor cosmetic improvement.
	- If the new field name truly was required, a phased migration would be the most appropriate solution:
		- First, add a new schema version to the registry that contains **both** field names.
		- Old consumers can still read the original field name and aren't broken.
		- New consumers can read the updated field name.
		- Once all consumers have updated to the new version, deprecate the old field name.
4. If Avro supports schema evolution, can I make any schema change I want?
	- No. Avro supports schema evolution **within compatibility rules**.
	- Adding or removing fields is typically safe.
	- Updating field names or data types must be handled with extreme care and usually requires a phased rollout.
	- Safe Changes:
		- Adding an optional field.
		- Adding a field with a default value.
	- Risky Changes:
		- Renaming fields.
		- Removing **required** fields.
		- Changing data types.
		- Adding required fields without default values.
			- A phased rollout mitigates the risk. Provide a default value until all consumers have updated their schemas, then remove the default field.