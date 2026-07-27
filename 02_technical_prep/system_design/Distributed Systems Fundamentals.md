# 📐 Distributed Systems & Data Architecture Foundations

## 1. Core Distributed Systems Concepts

### The CAP Theorem
In a distributed data store, you can only guarantee two out of the following three properties at the same time:
* **Consistency (C):** Every read receives the most recent write or an error.
* **Availability (A):** Every non-failing node returns a non-error response (without guaranteeing it contains the most recent write).
* **Partition Tolerance (P):** The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.

👉 **Interview Reality:** Network partitions (P) are inevitable in distributed systems. Therefore, data engineers must always choose between **Consistency (CP)** or **Availability (AP)** during a network partition.

### Data Shuffling, Partitioning, and Replication
* **Partitioning (Sharding):** Splitting a single massive dataset across multiple physical machines to parallelize processing.
* **Replication:** Copying the exact same data across multiple machines to ensure fault tolerance and high availability.
* **Data Shuffling:** The process of redistributing data across different cluster nodes over the network. This occurs during `JOIN` or `GROUP BY` operations when the system needs to group identical keys onto the same physical machine. Shuffling is highly expensive and is the number one bottleneck in distributed computing (e.g., Apache Spark).

---

## 2. Big Data Processing Architectures

When designing data pipelines, interviewers often ask how you balance real-time, low-latency insights with historically accurate data processing.

```text
Lambda Architecture:  [Data Source] ➔ 👥 Split ➔ 🚂 Batch Layer (Accurate/Slow) ➔ [Serving Layer]
                                            ➔ 🚀 Speed Layer (Real-time/Fast) ┘

Kappa Architecture:   [Data Source] ➔ 🌀 Streaming Pipeline (Single Stream Engine) ➔ [Serving Layer]
```

### Ⅰ. Lambda Architecture
* **How it works:** Data is split into two parallel tracks. A **Batch Layer** manages immutable master data and computes accurate, historical views. A **Speed Layer** processes delta data in real time using a streaming engine to fill the latency gap.
* **Pros:** Highly fault-tolerant. If the speed layer fails, the batch layer recomputes everything accurately on the next run.
* **Cons:** High code duplication. Engineers must maintain two separate codebases (e.g., Spark for batch and Flink for streaming) to compute identical business logic.

### Ⅱ. Kappa Architecture
* **How it works:** Removes the batch layer entirely. **All data** is treated as a continuous stream of events and processed through a single streaming engine (e.g., Apache Flink or Spark Streaming).
* **Pros:** Single codebase to maintain. To reprocess historical data, you simply reset the log offset of your streaming cluster (e.g., Kafka) and replay the historical events from the beginning.
* **Cons:** Requires highly complex stream-joining and state-management capabilities.

---

## 3. Storage Layer Architecture (Modern Data Lakehouse)

The trend in modern data infrastructure is decoupling storage from compute, culminating in the **Data Lakehouse** pattern.

```text
Compute Layer:         [ Snowflake ]      [ Databricks ]      [ Trino / Presto ]
                             ▼                  ▼                    ▼
Metadata Layer:   ────────────────── [ Iceberg / Delta / Hudi ] ──────────────────
Storage Layer:    ───────────────────────── [ AWS S3 ] ───────────────────────────
```

### Table Formats: Iceberg vs. Delta vs. Hudi
Traditional data lakes (raw files on S3) lack ACID transactions, schema enforcement, and file management. Modern open-source table formats solve this by introducing an abstraction metadata layer over raw files (like Parquet).

* **Apache Iceberg:** Outstanding for cross-engine compatibility (works seamlessly with Snowflake, Spark, and Trino simultaneously). It uses hidden partitioning and object-storage layout tracking rather than folder-based structures.
* **Delta Lake:** Deeply integrated with Databricks and the Apache Spark ecosystem. Features excellent performance optimization tools (like Z-Ordering).
* **Apache Hudi:** Optimized for fast upserts and incremental data streams, making it a strong choice for near-real-time ingestion pipelines.

---

## 4. System Design Interview Trade-off Matrix

Use this mental framework to justify your architecture choices when an interviewer asks you to build a system from scratch:

| Target Goal | Architecture Choice | The Alternative | The Trade-off Justification |
| :--- | :--- | :--- | :--- |
| **Real-time Analytics** | Event Broker (Kafka) + Olap Store (ClickHouse) | Batch Ingestion (S3 + Snowflake) | Low ingestion latency at the cost of high system complexity and storage duplication. |
| **Exact-Once Delivery** | Idempotent Consumer (Upserts based on Unique ID) | At-Least-Once Delivery | Processing deduplication costs slight processing overhead but guarantees strict data accuracy. |
| **Schema Evolution** | Protocol Buffers or Avro (Schema Registry) | Raw JSON Storage | Enforcing schemas early reduces processing errors downstream and optimizes data compression. |

## 5. Caching and Caching Strategies

### Caching Overview
- A cache is simply a fast storage layer placed in front of something slower, like a database. They exist to to reduce latency and offload the database, not to permanently store data. The database remains the source of truth because it guarantees durability and consistency. If the cache is lost or contains stale data, it can always be rebuilt from the database.
	```
	Application
	      ↓
	Database (20ms)
	```
	You have:
	```
	Application
	      ↓
	 Cache (1ms)
	      ↓
	 Database (20ms)
	```
	If the data already exists in the cache:
	```
	Application
	      ↓
	Cache
	```
	Done. No database access required.
- Why do we use caches:
	- Reduce latency
	- Reduce database load
	- Increase scalability
- Imagine Amazon Ads serves 500,000 requests/sec. Without a cache, you would need 500,000 DB reads per second. Using a cache with a 95% cache hit rate would result in only 25,000 DB reads per second (for the 5% of requests that resulted in a cache miss).
- Cache Hit vs. Cache Miss:
	- A Cache Hit occurs when the requested data already exists in the cache. This speeds up data access:
		```
		Application
		      ↓
		Cache
		      ↓
		Return data
		```
	- If the data doesn't already exist in the cache. This slows down data access:
		```
		Application
		      ↓
		Cache
		
		Not Found
		
		↓
		
		Database
		
		↓
		
		Store in Cache
		
		↓
		
		Return
		```

### Common Caching Strategies
1. Cache Aside (Lazy Loading):
	```
	Request
	
	↓
	
	Check Cache
	
	↓
	
	Miss?
	
	↓
	
	Database
	
	↓
	
	Store in Cache
	```
	- This is the most common caching strategy. Advantages include:
		- Easy implementation.
		- Only updates frequently-accessed data.
	- Disadvantages:
		- The first request for a given record is slower because it must be read from the database.
	- Redis commonly uses this caching strategy.
2. Write Through:
	```
	Application
	
	↓
	
	Cache
	
	↓
	
	Database
	```
	- Every database write also writes to the cache.
	- Advantages:
		- The cache remains updated.
	- Disadvantages:
		- Writes become slower.
3. Write Back (Write Behind):
	- The application only writes to the cache.
	- The cache then updates the DB asynchronously.
	- Advantages:
		- Very fast writes.
	- Disadvantages:
		- Higher risk of data loss (if a cache entry is evicted before being written to the DB).
4. Refresh Ahead:
	- Predict the entries users will request.
	- Refreshes the cache before expiration.
	- Best used for popular items.

### Time To Live (TTL) and Eviction Policies
- Every item in the cache is eventually evicted based on its TTL. For example, if creative A had a TTL of 1 hour, it would be evicted from the cache 1 hour after it was written into the cache.
	- Setting an appropriate TTL ensures items in a cache don't become stale when their corresponding database record changes. Without a TTL, cache items would become stale when updates occur, then remain stale indefinitely.
- Eviction policies decide what items are removed from a cache and can serve a similar purpose to a TTL. Common policies include:
	- Least Recently Used (LRU): Remove items that haven't been used recently.
	- Least Frequently Used (LFU): Remove rarely accessed items.
	- First In First Out (FIFO): Remove items based on when they're written to the cache, regardless of recency or frequency.

### Cache Stampede
```
1000 users

↓

Need same key

↓

Cache Miss
```
- Imagine 1000 users request the same item at the same time. Without proper protection, this would initiate 1000 database reads to retrieve and cache the missing item. This would overwhelm the database.
- The solution to this problem is to use a **Single Flight Pattern:**
	```
	Request 1
	
	↓
	
	Database
	
	↓
	
	Cache
	
	↓
	
	999 waiting requests
	
	↓
	
	Read cache
	```
	- The first cache miss triggers a database read to fetch and cache the item, while the other 999 requests wait. This results in only one database read.

### Cache Troubleshooting
- When a cache's hit rate suddenly experiences a significant decline, a good diagnostic procedure would be to:
	1. Determine if the issue was caused by configuration or workload changes.
	2. Examine the following configuration for recent changes:
		1. TTL Values
		2. Eviction Policies
	3. Examine the following metrics for abnormalities:
		1. Cache Hit Rate
		2. Cache Miss Rate
		3. Eviction Rate
		4. Memory Utilization
		5. Cache Restarts
	4. Further Examination:
		1. Look for changes in application traffic. Is a new feature requesting data that hasn't been cached?
		2. Are cache keys being generated consistently?
### Common Interview Questions
1. Why not just increase the cache size?
> 	Larger caches delay the problem but don't eliminate inefficient access patterns. Optimizing what we cache and how we invalidate it usually provides a more scalable long-term solution.
2. What happens when cached data becomes stale?
> 	That's a cache consistency problem. We typically address it with TTLs, explicit invalidation when the underlying data changes, or event-driven updates so the cache stays reasonably synchronized with the source of truth.
3. Why not cache everything?
> 	Memory is limited, and not all data is accessed frequently. Caching everything wastes memory and can evict hot data that benefits performance more.

### Common Discussion Questions
You should be able to discuss questions like:

- Which data should be cached?
- How long should it remain cached?
- What happens when the source changes?
- How do you prevent stale data?
- How do you avoid cache stampedes?
- How do you monitor cache hit rate?
- How does caching reduce infrastructure cost?

These often matter more than knowing specific cache APIs.

### Cheat Sheet
| Concept        | Key Idea                          |
| -------------- | --------------------------------- |
| Cache          | Fast copy of frequently used data |
| Cache Hit      | Data found in cache               |
| Cache Miss     | Fetch from database               |
| TTL            | Automatically expire cached data  |
| Cache Aside    | Load on first request             |
| Write Through  | Update cache and DB together      |
| Write Back     | Update cache first, DB later      |
| LRU            | Evict least recently used         |
| LFU            | Evict least frequently used       |
| Cache Stampede | Many requests miss simultaneously |
| Single Flight  | One fetch, many wait              |

## 6. Consistency

### Eventual Consistency:
```
Virginia Database
		|
Oregon Replica
		|
Ireland Replica
```
- imagine you have copies of the same data in three different locations. When a user updates there profile in Virginia, should the Oregon and Ireland databases instantly have the new value?
	- In a distributed system, no. Replication takes time. For a brief period, the other databases will contain stale data. Eventually, every replica converges to the same value. This is known as **eventual** consistency.
- Distributed systems accept the possibility of stale data because strong consistency is expensive. If every write had to wait for every replica in the world:
	- Latency increases
	- Availability decreases
	- Throughput decreases
	- Most systems would become much slower
- For many applications, briefly stale data is not a serious concern. Examples include:
	- Social media likes or posts.
		- When a user posts conent in the US, EU users won't see it for a brief period of time (~10 seconds).
		- Nothing is broken. The data simply hasn't propagated to every region yet.
	- Product reviews
	- Analytics dashboards
	- Inventory counts that tolerate brief lag
	- **Keep in mid that different parts of the same system can have different consistency needs.**
		- For example, when a customer updates their shipping address, there should be strong consistency between the user database and the system responsible for placing orders. If a customer updates there address, then *immediately* places an order, using the old address is unacceptable.
		- Eventual consistency is acceptable between the user database and downstream consumers, such as analytics or recommendation systems. A few seconds of delay **won't impact the user.**
		- In large, distributed systems, it's common to have:
			- **Strong consistency** for the transactional workflow (placing the order).
			- **Eventual consistency** for downstream consumers (analytics, search indexes, reporting, recommendation engines).
	- **Consistency requirements are driven by business requirements.** Experienced Data Engineers always ask "What freshness does this business actually need?"
- When stale data in a cache causes issues in production, this can potentially be fixed by synchronizing cache initialization with database commits, not be simply switching to a strong consistency model.
- Pros:
	- High availability
	- Low latency
	- Better scalability
	- Faster writes
- Cons:
	- Stale reads
	- More complex application logic
	- Harder debugging
	- Temporary inconsistencies

### Strong Consistency
```
User updates balance

Database:
$150

Immediately after...

Every read returns:

$150
```
- A system is **strongly consistent** when every read immediately reflects the most-recent successful write in a distributed system. Once a write completes, every user sees the same value.
- Strong consistency is difficult to implement in a distributed system because a write cannot be considered complete until the system can guarantee *all* future reads return the most current data. This often requires:
	- waiting for replicas
	- coordinating nodes
	- agreeing on ordering
- This level of coordination introduces latency.
- Strong consistency is often required in situations where simply cannot be stale or incorrect. For example if you withdraw $80 from an account that only has $100, you might be able to quickly move to another ATM and withdraw another $80 if the account balance data is eventually consistent.
- Other examples that typically require strong consistency include:
	- Payment processing
	- Inventory reservations (selling the last item)
	- Authentication/password changes
	- Airline seat reservations
	- Order placement
	- Note that these all involve **transactions**, where the cost of stale data is high.
- Strong consistency typically isn't required for things like:
	- Product recommendations
	- Analytics dashboards
	- Social media likes
	- Trending topics
	- Search indexes
	- A few seconds of delay rarely has any business impact.

### Common Real-World Examples
|Requirement|Appropriate solution|
|---|---|
|Financial fraud detection|Real-time streaming|
|Live order tracking|Real-time streaming|
|Executive dashboard|Eventual consistency (minutes)|
|Daily sales report|Batch processing|
|Monthly finance report|Batch processing|

### Tradeoffs
| Strong Consistency                 | Eventual Consistency        |
| ---------------------------------- | --------------------------- |
| Always latest data                 | May be briefly stale        |
| Higher latency                     | Lower latency               |
| More coordination                  | Less coordination           |
| Lower availability during failures | Higher availability         |
| Simpler application logic          | More application complexity |
- There isn't universally a "better choice" between strong and eventual consistency. There is only: **"What does the business require?"**
	- Systems that are eventually consistent can be moved closer to strong consistency without a full redesign. This can be accomplished, for example, by synchronizing database commits with cache initialization to eliminate time gaps that would otherwise cause stale / inconsistent data.
- Isolating operations that require strong consistency is a better system design approach than the entire system strongly consistent. This minimizes coordination overhead while protecting the operations that truly need correctness. For example:
	- Payments → Strong
	- Analytics → Eventual
	- Search → Eventual
	- Reporting → Eventual
	- User account balance → Strong
- Strong consistency guarantees correctness. Eventual consistency optimizes scalability. Whenever you're asked "Which one would you choose?", the answer almost never "Strong" or "Eventual". The real answer is: "Whichever satisfies the business requirements with the least complexity."
### Common Interview Questions
1. Imagine a user updates their shipping address. Would you design that workflow using **eventual consistency** or **strong consistency**? Explain **why**, not just which one you'd choose.
> 	For a shipping address, I'd first consider when that address is actually used. If the customer updates their address and immediately places an order, then the ordering service should read the updated address before creating the shipment. However, if the address is being replicated to analytics systems, recommendation systems, or customer profile replicas, eventual consistency is perfectly acceptable because a few seconds of delay doesn't affect the user.
2. Suppose you have an analytics dashboard showing yesterday's sales. The data pipeline takes **3 minutes** to process new transactions. A business user complains that the dashboard doesn't updated **immediately** after a sale. Would you redesign the pipeline to provide **strong consistency**, or would you keep **eventual consistency**? Why?
	> Before redesigning the system, I'd **clarify the business requirement**. If the dashboard is intended for operational monitoring or executive reporting, a 3-minute delay is probably acceptable and I'd explain that the dashboard is eventually consistent by design. If the delay truly impacts business decisions, I'd first look for incremental improvements such as reducing pipeline latency or increasing processing frequency. I would only redesign the architecture for near real-time streaming if the business justified the added complexity and cost.
3. You're building an **Amazon product page**. Which of these should use **strong consistency**, and which can use **eventual consistency**? Explain your reasoning.
	1. Product description
	2. Number of reviews
	3. Current inventors
	4. Average star ratings
> 	Product descriptions, review counts, and average ratings can all tolerate brief delays because seeing slightly stale values doesn't affect the correctness of a purchase. Inventory, however, directly affects whether a customer can successfully buy an item. If inventory isn't strongly consistent, multiple customers could purchase the last unit simultaneously, leading to overselling."
4. Suppose an interviewer says: "Strong consistency sounds strictly better. Why doesn't every distributed system use it?" How would you answer?
> 	Strong consistency requires coordinating writes across multiple nodes before they are considered complete. That coordination increases latency, reduces availability during failures, and adds operational complexity. Since many applications can tolerate briefly stale data, eventual consistency provides better performance and scalability without sacrificing the business requirements.

Interviewers love hearing words like:

- coordination
- latency
- scalability
- availability
- tradeoffs

Those are the keywords of distributed systems.

## 7. Race Conditions

### Overview
- A **race condition** occurs when the correctness of a program depends on the timing or ordering of two or more operations. In other words:
> 	Two operations are "racing" each other, and whichever finishes first changes the outcome.
- The scary part is that race conditions are often:
	- intermittent
	- difficult to reproduce
	- impossible to catch with simple unit tests
- Simple Example:
	- Imagine a bank account with a balance of $100.
	- Two users try to withdraw $80 at the exact same time.
	- Thread A reads a balance of $100.
	- Thread B reads a balance of $100.
	- Both think there's a balance of $100.
	- Both withdraw $80.
	- Now, there's ($60) in the account.
- Neither thread was wrong individually. The issue was that they both read the same old value before either write completed. **That's a race condition.**
	- Another way to define a race condition is "an outcome that depends entirely on timing".
- Race conditions occur whenever two things can happen **concurrently**, such as:
	- multiple users
	- multiple threads
	- multiple services
	- multiple servers
	- asynchronous jobs
	- distributed systems
- If they all access the same resource **without coordination,** race conditions become possible.
- Distributed systems exacerbate race conditions because each server in the system can independently receive and process a request. Even if each server is **individually correct**, they must coordinate to produce a correct result.
- Common causes of race conditions include:
	- Concurrent requests
	- Multiple threads
	- Asynchronous processing
	- Distributed services
	- Shared mutable state
	- Out-of-order execution

### Common Solutions
| Problem                               | Typical Solution     |
| ------------------------------------- | -------------------- |
| Two threads modify same object        | Locking              |
| Services execute in wrong order       | Synchronization      |
| Duplicate requests                    | Idempotency          |
| Multiple updates overwrite each other | Optimistic locking   |
| Too much contention                   | Pessimistic locking  |
| Temporary failures                    | Retries with backoff |
- Almost every distributed systems technique exists because of race conditions.
- One solution not listed above is a time-based solution, such as adding a one second delay between two events. **Time-based solutions are brittle** because system timing is unpredictable.
	- "Never use time to guarantee ordering."
	- "Sleep is not synchronization."

### Common Interview Quesstions
1. Imagine two Spark jobs. Job A writes `sales.csv`, while Job B realds `sales.csv`. Job B reads becfore Job A finishes writing. When this occurs, data may be incomplete, corrupted, or inconsistent because the two jobs raced each other. An interviewer asks "Why did this bug occur?"
> 	The cache wasn't ready because two independent operations were executing concurrently without coordination, creating a race condition.
2. Suppose two users edit the same customer profile at the same time. User A changes the email. User B changes the phone number. Both click **Save** within milliseconds. **What race condition could occur?**
3. In your [[Meeting Tight Deadline]] story, suppose someone suggests: "Instead of synchronizing the cache initialization with the database commit, let's just sleep for one second before reading the cache." Why is this a poor solution? What problems could it introduce?
> 	Sleeping doesn't establish any dependency between the operations. If the cache initialization takes longer than expected because of network latency, heavy load, or another delay, the request can still arrive before the cache is ready. On the other hand, if the cache is ready almost immediately, we've added a one-second delay that only hurts performance. It's both unreliable and inefficient.

## 8. Locking

- The biggest mistake people make when it comes to understanding how locking works is trying to memorize the definitions. Instead, think about the **problem each strategy is trying to solve.** For example:
	- Imagine you're editing a Google Doc.
	- You open the document.
	- I open the document.
	- We both start editing at the same time.
	- How should the system prevent us from overwriting each other's work?
	- There are two broad philosophies: Optimistic and Pessimistic Locking.

### Pessimistic Locking
- Philosophy: Assume conflicts are common. Prevent them from happening.
- When someone starts modifying data, the system locks it. Everyone else must wait. Imagine one bathroom key at a gas station:
	```
	Bathroom Key
	
	↓
	
	One person has it
	
	↓
	
	Everyone else waits
	```
	- No two people can use the bathroom key at the same time. That's pessimistic locking.
- Database Example: Suppose we have an inventory table:
	```
	Inventory
	
	Laptop
	
	Quantity = 1
	```
	- Customer A begins making purchases.
	- The database locks the row for this item.
	- While Customer A is checking out:
	```
	Customer B
	
	↓
	
	Attempts update
	
	↓
	
	Blocked
	```
	- Customer B must wait until the lock is released to proceed with their purchase.
	- This eliminates the possibility of overselling.
- Advantages:
	- Easy to reason about
	- Prevents race conditions
	- Guarantees only one writer at a time
	- Good when conflicts are common
- Disadvantages:
	- Imagine thousands of customers buying the same product.
	- Everyone waits.
	- The system becomes slower.
	- Locks can also create:
		- bottlenecks
		- deadlocks
		- reduced throughput

### Optimistic Locking
- Philosophy: Assume conflicts are rare. Let everyone work concurrently, then detect conflicts before saving.
	- Instead of locking, all users are allowed to make edits independently.
	- The system only checks if the data has changed when it is saving changes.
- Multiple people can make edits to a record at the same time, but only one person can actually commit those changes. Imagine two users trying to update the same balance:
	```
	Version = 5
	
	Balance = $100
	```
	- User A reads Version 5.
	- User B also reads Version 5.
	- Both modify the data.
	- User A saves.
	- The database saves the changes and updates the Version:
	```
	Version = 6
	```
	- Now User B tries to save changes on Version 5.
	- Their update says: "Update Version 5."
	- The database responds with:
	```
	Current Version = 6
	
	Conflict
	```
	- The update is rejected.
	- User B must reload the latest data and try again.
	- No lock was ever held.
- This is actually how Git handles concurrent edits.
	- When two people both try to push changes on the same version of the repository, who ever pushes their changes first "wins". The "loser"  must pull the "winner's" changes and resolve any potential merge conflicts before pushing their changes.
	- Imagine how frustrating it would be if Git used pessimistic locking instead. Only one person could do work on a repo at a given time.
- It's called "optimistic locking", given the current state of the system, it's optimistic enough to believe: "Most of the time, nobody else changes this."
	- Most updates succeed immediately.
	- Only ocassional conflicts need extra work.
- Amazon Ads Example:
	- Two advertisers try to make edits to the same campaign at the same time.
	- The first edit saves successfully.
	- The second edit fails with the message: "This campaign has been modified. Please refresh and try again."
	- This is an inexpensive way of resolving the issue because conflicts like this are rare.
- Black Friday Example:
	- There's one Nintendo Switch left in stock and thousands of users are shopping at the same time.
	- In a high-contention scenario such as this, pessimistic locking—or another coordination mechanism—may be more appropriate because conflicts are expected.
- Data Engineering Example:
	- Two ETL jobs are updating the same warehouse table.
	- Pessimistic Locking:
		- Job B waits until Job A finishes.
		- No conflicts.
		- Lower throughput.
	- Optimistic Locking:
		- Both jobs run at the same time.
		- At commit time, one job succeeds and the other must be retried.
		- This is common in modern data lake formats like Apache Iceberg and Delta Lake, which use optimistic concurrency control.

### Comparing The Two
| Pessimistic                        | Optimistic                     |
| ---------------------------------- | ------------------------------ |
| Lock first                         | Check later                    |
| Wait before updating               | Retry on conflict              |
| Higher latency                     | Higher throughput              |
| Better when conflicts are frequent | Better when conflicts are rare |
| Users may block each other         | Users rarely block each other  |
- Neither one is better than the other.
- They're solving different problems.
- A common misconception is that "Optimistic locking prevents conflicts."
	- It doesn't. It allows conflicts to occur but **detects them before committing**, preventing inconsistent data from being written.
	- Pessimistic locking, on the other hand, **prevents the conflict from happening in the first place** by blocking concurrent modifications.
- Pessimistic locking is a **concurrency strategy**, not necessarily "use SQL row locks."

### Common Interview Questions
1. When would you choose optimistic locking?
> 	When updates to the same data are relatively rare. Optimistic locking avoids the overhead of holding locks and improves throughput, while still detecting conflicting updates before they are committed. I'd choose it when conflicting updates are expected or when the cost of a conflict is very high, such as inventory reservations or financial transactions."
2. You're building a **user profile service**. Millions of users exist. Most users edit their profile only a few times per year. Would you choose **optimistic** or **pessimistic** locking? Why?
> 	I would choose optimistic locking because simultaneous edits to the same user profile are relatively rare. Holding locks for every profile update would add unnecessary overhead. If a conflict does occur, the application can detect it during the update, reject the stale write, and ask the user to refresh before retrying.
3. Now imagine you're building the checkout system for a concert venue with only **one VIP seat remaining**. Thousands of people click **Buy** at almost the same time. Which locking strategy would you choose? Explain your reasoning and any tradeoffs.
> 	I would choose pessimistic locking in this scenario because the likelihood of a conflict is incredibly high. Furthermore, any conflict would have a direct impact on customer experience.
4. Would Amazon actually use a database lock for concert tickets?
> 	Not necessarily. At very large scale, systems often avoid traditional database locks because they don't scale well under extreme contention. Instead, they might use the following:
> 		- Atomic database operations (e.g., UPDATE ... WHERE quantity > 0)
> 		- Distributed locks
> 		- Reservation services
> 		- Message queues
> 		- Compare and Swap (CAS) operations
> 	The important point is that **the underlying strategy is still pessimistic**: only one successful purchaser gets the last seat.

## 9. Idempotency

- Idempotency is very important in distributed systems because failures occur all the time:
	- networks time out
	- services crash
	- consumers restart
	- messages are delivered twice
	- retries happen automatically
- The questions becomes, **"If the same operation runs twice, what happens?"** This is exactly what idempotency solves.
- An operation is **idempotent** if performing it multiple times has the same effect as performing it once. In other words, the operation can be run once or multiple times, but still produce the same correct result. The final state of the system is identical.
- Imagine you're updating your shipping address, but the same request is sent three times:
	```
	Set Address =
	
	123 Main Street
	
	↓
	
	123 Main Street
	
	↓
	
	123 Main Street
	
	```
	- The final result (updated address) is the same. This operation is idempotent.
- On the contrary, imagine a request is being sent to increase your account balance, but the same request is sent three times:
	```
	Increase Account Balance
	
	+$100
	
	↓
	
	$100
	
	↓
	
	$200
	
	↓
	
	$300
	```
	- This operation is not idempotent. The repeated execution changed the final result.
- Duplicate requests can and do occur in the real world for multiple reasons:
	- A network response could be lost, resulting in a timeout. The client making the request assumes it failed and sends another request. If the request being sent was related to making an order, two separate orders could be made without idempotency.
	- A client's browser could freeze while clicking the "Place Order" button. The client clicks again, or reloads the page, then clicks again. Without idempotency, this could also result in duplicate orders.

### Implementing Idempotency
- One common approach is using an **idempotency key**. When a client makes a request, it generates a unique identifier:
	```
	Request
	
	Idempotency-Key
	
	ABC123
	
	↓
	
	Order #987
	```
	- The server receives it, processes it correctly, and stores the idempotency key alongside the Order ID.
	- If the same request arrives again later, the system will be able to identify that it's already been processed, based on the idempotency key. It will simply return the processed order in the response. No duplicates occur.
- Another common approach is to use operations that are naturally idempotent:
	```
	UPDATE users
	
	SET email = "new@email.com"
	```
	- Running this SQL statement multiple times produces the same final result.
	```
	UPDATE accounts
	
	SET balance = balance + $100
	```
	- This SQL statement is not idempotent because the balance is updated during each execution.
- **Data Engineering Example:** Imagine a Kafka consumer that processes:
	```
	Order Event
	
	↓
	
	Write to Warehouse
	```
	- If the consumer suddenly crashes after writing, Kafka doesn't know if the write succeeded. When the consumer restarts, it receives the same message again.
	- Without idempotency, duplicate records are created.
	- With idempotency, the consumer first checks if it has already processed the record. If it has, it skips it.

#### Common Examples
Interviewers don't expect you to memorize every implementation, but it's good to recognize the common patterns:

|Technique|Idea|
|---|---|
|Idempotency key|Unique request ID prevents duplicate processing|
|Upsert (`INSERT ... ON CONFLICT`)|Insert if new, otherwise update|
|Unique constraints|Database rejects duplicates|
|Version numbers|Ignore stale or duplicate updates|
|Event IDs|Track which events have already been processed|
- Many candidates think that retries automatically cause duplicates. This is not necessarily correct. Retries **expose** duplicates. The real problem is that the underlying operation isn't idempotent. That's why retries and idempotency are almost always discussed together.
- Idempotency doesn't aim to prevent retries, it aims to make retries safe.

### Common Interview Questions
1. Why are idempotent APIs important?
> 	Because distributed systems experience timeouts, network failures, and retries. If a client can't determine whether a previous request succeeded, it needs to safely retry the operation. Idempotency ensures those retries don't produce duplicate side effects.
2. A user clicks **"Pay Invoice"** and the payment request times out. The payment may have succeeded, but the client doesn't know. How would you design the API so the client can safely retry without charging the customer twice?
> 	I'd have the client generate a unique idempotency key for each payment request. The server would store that key along with the payment result. If the client retries because of a timeout, the server checks whether the key has already been processed. If it has, the server returns the original response instead of charging the customer again.
	- Note that server doesn't simply **ignore** the duplicate request, it returns **the same successful response** it returned the first time.
	- This makes it so that the client doesn't have to distinguish between:
		- "This is the first request."
		- "This is a retry."
3. Suppose an ETL job processes yesterday's sales and crashes halfway through. When it restarts, it begins reading from the same input again. How could you make the pipeline idempotent so rerunning it doesn't create duplicate records?
> 	I would design the pipeline to merge incoming data so that it inserted new data and updated existing data based on the primary key.
	```sql
	MERGE INTO customers AS target
	USING staging AS source
	ON target.customer_id = source.customer_id
	WHEN MATCHED THEN
	    UPDATE ...
	WHEN NOT MATCHED THEN
	    INSERT ...
	```
4. Is every `MERGE` operation automatically idempotent?
> 	No, every MERGE operation is not automatically idempotent.
	```sql
	UPDATE sales
	SET total = total + source.amount
	```
	- In this case, if the same data is processed twice, the total is incremented twice. Thats **not** idempotent.
	- However:
	```sql
	UPDATE sales
	SET total = source.total
	```
	- Is idempotent because every rerun sets the value to the same result. The key idea is that **the operation itself** must be safe to repeat.

## 9.  Retries With Exponential Backoff

- An important thing to understand about distributed systems is that they **fail regularly**.  Networks time out. Services become overloaded. Containers restart. The question isn't **if** failures happen—it's **how your system responds**.
- A mistake many engineers make is thinking, "If a request fails, just try again immediately." Ironically, that's often the worst thing you can do.
- Retries exist because network requests, while usually successful, can sometimes fail for several reasons:
	- network latency spikes
	- the service is restarting
	- CPU usage reaches 100%
	- the database is temporarily unavailable
- The naive approach to executing retries to retry immediately:
	```
	Attempt 1
	
	↓
	
	Fail
	
	↓
	
	Attempt 2
	
	↓
	
	Fail
	
	↓
	
	Attempt 3
	
	↓
	
	Fail
	```
	- The issue with this approach is that a failed request is a sign that the service is already struggling. By trying again immediately, you're making it work even harder.
	- Now imagine **10,000 clients** all doing the exact same thing. The service gets flooded with retry requests. This is called a **retry storm**.
- Exponential Backoff is a more sustainable approach: Wait a little (1 second) before retrying. Then wait longer (2 seconds), Then wait even longer (4 seconds). The delay between each retry grows exponentially. This gives the downstream service time to recover.
	- This can still cause retry storms if every client uses the same strategy at the same time. Even though the delay between requests is increasing, the server is still being flooded with requests during each attempt.
	- Adding Jitter can help solve this issue: Instead of each client waiting the same amount of time between retries, each client waits a random (but reasonable) amount of time between retries. This dramatically reduces traffic spikes.
- Exponential Backoff with Jitter is a suitable retry strategy, but it's also important to recognize that not every failure should be retries. Failures that are good candidates for retry include:
	- Network timeout
	- Temporary database outage
	- HTTP 503 (Service Unavailable)
	- HTTP 429 (Too Many Requests)
	- Temporary connection failures
	- These often resolve on their own.
- Bad candidates include:
	- HTTP 400 (Bad Request): Retrying the request won't solve the issue because the request is malformed.
	- HTTP 401 (Unauthorized): Retrying the request won't magically create valid credentials.
- A good rule of thumb is to retry **transient** failures, but not **permanent** failures.
- Retries and Idempotency are two highly related topics because idempotency makes retries safe to perform, while retrying with the correct strategy ensures a successful response when transient failures occur.

### Common Examples
- **Data Engineering Example:** Suppose your spark job writes data to Snowflake. If the connection drops, should the job fail immediately?
	- Usually not. It should retry because:
		- The warehouse may be temporarily busy
		- The network issue may resolve
		- The database may be restarting
	- If the write is idempotent (for example, using a `MERGE` based on primary keys), retrying is safe.
- **Amazon Example:** Imagine your service calls another internal service to retrieve metadata. One request fails because the internal service is currently being deployed and is unavailable for 5 seconds. Should the engineer be paged immediately?
	- No they shouldn't. If the client service retries the request with exponential backoff, the request will succeed after 5 seconds and the customer will never notice.

### Tradeoffs
| Immediate Retries               | Exponential Backoff           |
| ------------------------------- | ----------------------------- |
| Fast recovery if issue is brief | Better for sustained failures |
| Can overwhelm services          | Protects overloaded services  |
| Higher risk of retry storms     | Spreads requests over time    |
| Simple                          | Slightly higher latency       |

### Common Interview Questions
1. Why don't we retry forever?
> 	Because some failures are permanent, and unlimited retries waste resources, increase latency, and can overwhelm downstream services. Most systems limit the number of retries and surface an error if the operation continues to fail.
	- This shows you understand that retries are a recovery mechanism—not a substitute for handling real failures.
2. Suppose your ETL pipeline tries to write to a data warehouse. The warehouse returns **HTTP 503 (Service Unavailable).** Would you retry? How would you retry? When would you stop retrying?
> 	I would retry using exponential backoff with jitter because a 503 indicates the service is temporarily unavailable. I'd configure a maximum number of retries or an overall timeout to prevent retrying indefinitely. If the retries are exhausted, I'd surface an error to the caller or move the request to a dead-letter queue or retry queue, depending on the application's requirements.
	- Don't mention stopping retries after a specific period of time unless the interviewer gives you a service-level requirement. Instead, I'd talk about **retry limits** or a **retry policy**.
	- This is stronger because you're not guessing an arbitrary timeout. You're recognizing that **retry policies depend on business requirements**. You're also introducing concepts such as **maximum retries**, **overall timeouts**, and **dead-letter / retry queues**, which interviewers associate with resilient distributed systems.
3. An API returns **HTTP 400 (Bad Request).** Should your client retry with exponential backoff? Why or why not?
> 	An HTTP 400 indicates the request itself is invalid. Retrying the exact same request will produce the same result. Instead, the client or developer should inspect the error response, correct the request, and submit a new valid request.
4. What if 100 services all retry at exactly the same time?
> 	That's why I'd add jitter to the exponential backoff. Randomizing the retry interval prevents all clients from retrying simultaneously, reducing the chance of a retry storm and giving the downstream service a better opportunity to recover.
	- If you mention **jitter** in an interview without being prompted, it demonstrates that you've moved beyond the basics.

### Connecting Retries With Other Concepts
```
Transient failure
        ↓
Retry
        ↓
Safe because of idempotency
        ↓
Backoff + jitter prevents retry storms
        ↓
Concurrency can still create race conditions
        ↓
Locking and synchronization resolve them
```
- This is a coherent mental model of distributed systems, and it's the kind of reasoning interviewers look for.

## 10. Circuit Breakers

- Retries help recover from temporary failures, but what if a downstream service stays unhealthy for minutes instead of seconds? At some point, continuing to send requests only makes things worse.
- **Circuit breakers** answer the question: "When should we stop retrying, and fail instead?"
	- This is another foundational resilience pattern that's frequently discussed in Data Engineering and distributed systems interviews.
- Imagine an Order Service that depended on a Payment Service:
	```
	Order Service
	      │
	      ▼
	Payment Service
	```
	- Normally, a request results in a successful response.
	- What happens if the Payment Service crashes and every request continues trying to reach it?
		- Imagine 10,000 users are all trying to place orders at the same time while the Payment Service is down. Now, the Order Service spends all of its time and resources waiting for a service that is unavailable. **You've turned one failing service into two failing services**.
- A circuit breaker in an applicatio works just like a circuit breaker in a house:
```
Circuit Breaker

↓

Trips

↓

Power Stops
```
- When too much current flows through a circuit, the breaker trips, and the power is cut off. This protects the electrical system from being overloaded.
- Software uses the same idea. If too many requests fail, a circuit breaker trips, and requests stop being sent, instead of continuing to hammer a failing service.

### Circuit Breaker Functionality
A circuit breaker has three states: Closed (Normal), Open (Fail Fast), and Half-Open (Recovery)

1. Closed (Normal):
	```
	Order Service
	
	↓
	
	Payment Service
	
	↓
	
	Success
	```
	- Everything is healthy and requests flow normally.
1. Open (Fail Fast):
	```
	Order Service
	
	↓
	
	Circuit Breaker
	
	↓
	
	Reject Request
	```
	- When failures exceed a threshold, the circuit breaker opens (pops). Once the circuit breaker pops, subsequent requests **never reach** the Payment Service.
	- The circuit breaker immediately returns an error or fallback response.
	- This is called **failing fast**.
	- Without a circuit breaker:
		```
		Request
		
		↓
		
		30-second timeout
		
		↓
		
		Failure
		```
	- With a circuit breaker:
		```
		Request
		
		↓
		
		Immediate failure
		
		↓
		
		Return response in milliseconds
		```
		- The user gets an answer much sooner, and your application isn't wasting resources waiting on a service that is known to be unhealthy.
1. Half-Open (Recovery):
	- Eventually, we want to know whether the Payment Service has recovered. Instead of sending all traffic, the circuit breaker starts to send a small number of test requests:
		```
		One request
		
		↓
		
		Payment Service
		```
		- If the requests succeed, the breaker closes, and traffic resumes normally.
		- If the requests fail, the breaker closes again, and the service gets more time to recover.
- State Diagram:
	```
	Closed
	   │
	Too many failures
	   ▼
	Open
	   │
	Wait
	   ▼
	Half Open
	   │
	      ├── Success → Closed
	      │
	      └── Failure → Open
	```
	- This is worth remembering because interviewers often ask about these three states.

### Real-World Examples
- **Amazon Example:** Imagine Amazon's recommendation service goes down. Should customers be prevented from buying products?
	- No. Instead, when the recommendation service fails, the website continues functioning:
		```
		Product Page
		
		↓
		
		Recommendations unavailable
		
		↓
		
		Show page anyway
		```
		- This is called graceful degredation.
- **Data Engineering Example:** Suppose an ETL pipeline loads data from an external API. When the API starts returning 503 errors, how should the pipeline behave?
	- Without a circuit breaker:
		```
		100 workers
		
		↓
		
		API
		
		↓
		
		100 failures
		
		↓
		
		100 retries
		```
		- The API gets overwhelmed.
	- With a circuit breaker:
		```
		Workers
		
		↓
		
		Circuit Open
		
		↓
		
		Stop Requests
		
		↓
		
		Retry Later
		```
		- Both systems are protected.

### Relationship to Retries
- A common interview question is "Why do we need circuit breakers if we already have retries?"
	- Circuit breakers and retries solve different problems.
	- Retries assume the service will recover soon. They try to recover from transient failure.
	- Circuit breakers assume the service is still unhealthy. They prevent additional damage.
	- They complement each other. A common workflow looks like:
		```
		Request
		
		↓
		
		Retry 1
		
		↓
		
		Retry 2
		
		↓
		
		Retry 3
		
		↓
		
		Still failing
		
		↓
		
		Circuit Opens
		```
- **Amazon Example:** Suppose your service depends on another internal service. A deployment goes wrong and the service is unavailable for five minutes.
	- Without a breaker:
		- Thousands of requests wait for timeouts.
		- CPU usage increases.
		- Thread pools fill up.
		- Memory usage grows.
		- Eventually, your service starts failing too.
		- This is called **cascading failure**.
	- With a breaker:
		- The dependency fails.
		- Your service quickly returns an error or fallback.
		- Only one service is unhealthy.

### Common Fallbacks
- When a circuit breaker is open, what should your application do?
	- The answer depends on the feature. Examples include:
		- Return cached data.
		- Return default values.
		- Skip a non-essential feature.
		- Queue the request for later.
		- Return a friendly error.
	- For example, if recommendations are unavailable, simply show the product page without recommendations. If the Recommendation Service specifically shows user-personalized recommendations, default / generic recommendations can be shown instead.
	- If inventory is unavailable, return an error message saying "We're unable to verify inventory right now. Please try again shortly."

### Tradeoffs
| Without Circuit Breaker           | With Circuit Breaker        |
| --------------------------------- | --------------------------- |
| More retries                      | Fail fast                   |
| Longer response times             | Faster failures             |
| Can overload dependencies         | Protects dependencies       |
| Higher risk of cascading failures | Better resilience           |
| Simpler implementation            | More operational complexity |
- Circuit breakers are primarily used to prevent cascading failures among related services

### Common Interview Questions
1. Your data pipeline calls an external weather API. The API has been returning **503 Service Unavailable** for the past **10 minutes**. Would you continue retrying every request, or would you use a circuit breaker?
> 	I would use a circuit breaker. Continuing to retry puts unnecessary strain on both the weather API and the data pipeline. Once the circuit breaker transitions to the half-open state, I'd allow a small number of requests through. If they succeed, the circuit would close and normal traffic would resume.. While the weather API is down, I would either continue showing the most recent data or return error messages, depending on the business requirements.
	-  Remember a circuit breaker **opens** in response to failure. It doesn't close.
2. What's the difference between retries with exponential backoff and a circuit breaker?
> 	Retries with exponential backoff and jitter assume the failure is temporary and give the downstream service time to recover without creating a retry storm. Circuit breakers are used when failures persist. They stop sending requests to the unhealthy service, fail fast, and prevent cascading failures while periodically checking whether the dependency has recovered.
3. If you have retries, exponential backoff, idempotency, and a circuit breaker, do you still need monitoring and alerting?
> 	Yes. All of these patterns make a system more resilient, but none of them fix the underlying problem. If your service is nown for 30 minutes, retries will eventually stop, the circuit breaker will remain open, and idempotency will handle any duplicate requests gracefully. None of this changes the fact that the service is still down. Someone needs to manually investigate and resolve the outage. That's where monitoring, alerting, dashboards, and on-call engineers come in.

### Overall Pattern
```
Request

↓

Transient failure

↓

Retry with exponential backoff + jitter

↓

Still failing?

↓

Circuit breaker opens

↓

Fail fast

↓

Half-open probe

↓

Recovered?

↓

Resume traffic
```

## 11. Synchronization

- If a race condition answers "what went wrong?", synchronization answers "How do we guarantee operations happen in the correct order?"
- Synchronization is the process of coordinating multiple operations so they occur in the correct order or at the correct time. Think of it as establishing **dependencies** between operations.
	- Instead of:
		```
		Operation A
		
		Operation B
		
		...whoever finishes first wins
		```
	- You explicitly say:
		```
		Operation A
		
		↓
		
		Operation B
		```
		- Operation B is not allowed to begin until Operation A has completed.
- Synchronization is needed in scenarios where order of operations is critical to the final outcome. Think about baking a cake:
	```
	Mix ingredients
	
	↓
	
	Pour batter
	
	↓
	
	Bake
	
	↓
	
	Decorate
	```
	- You can't bake the cake before mixing the ingredients or pouring the batter. The correctness of the process depends on the sequence.
- In the [[Meeting Tight Deadline]] story, the sequence of events was:
	```
	Database Commit
	        │
	        ├──────────────┐
	        │              │
	        ▼              ▼
	Cache Warmer      User Request
	```
	- The Cache Warmer and the User Request were completely independent.
	- Sometimes:
		```
		Database Commit
		
		↓
		
		Cache Warmer
		
		↓
		
		Request
		
		↓
		
		Success
		```
	- Other times:
		```
		Database Commit
		
		↓
		
		Request
		
		↓
		
		Cache Miss
		
		↓
		
		Failure
		```
	- The final result depended on timing, which is what created the race condition.
- This was corrected by synchronizing the two operations:
	```
	Database Commit
	
	↓
	
	Initialize Cache
	
	↓
	
	Complete Request
	```
	- This introduced a dependency. The request could not complete until the cache was initialized.
	- The problem was fixed by synchronizing the operations, not by making the cache faster.
	- Synchronization made the ordering of events **deterministic**.

### Synchronization vs. Waiting
- A common misconception people make is thinking synchronization means waiting a **specific** amount of time. Synchronization actually means waiting **until a specific event happens**.
- Waiting:
	```
	Sleep(1000 ms)
	
	↓
	
	Read Cache
	```
	- Doesn't guarantee ordering.
- Synchronizing:
	```
	Wait until cache initialization completes
	
	↓
	
	Read Cache
	```
	- Guarantees ordering.

### Common Examples
- ETL Pipeline:
	```
	Extract
	
	↓
	
	Transform
	
	↓
	
	Load
	```
	- Transformation depends on the Extract phase completing successfully.
	- Loading depends on the Transform phase completing.
	- This dependency guarantees data is processed correctly.
- Distributed Systems:
	- Imagine an Inventory Service and an Order Service. When a customer purchases a laptop, should the order service confirm the purchase before inventory is reserved?
	- No. The correct order is:
		```
		Reserve Inventory
		
		↓
		
		Confirm Order
		```
		- Otherwise, the merchant has promised the customer something they can't deliver.
- Data Engineering:
	```
	Job A completes
	
	↓
	
	Signal completion
	
	↓
	
	Job B begins
	```
	- Job A generates a Daily Sales file.
	- Job B loads the file into the data warehouse.
		- If Job B starts too early, a half-written file is loaded into the warehouse, corrupting the data inside.
	- Instead, the two jobs are synchronized.
	- Many workflow orchestration tools like Apache Airflow and AWS Step Functions are built around the concept of defining task dependencies.

### Common Synchronization Mechanisms
- Synchronization isn't a single technology. It's a goal that can be achieved in different ways. Example include:
	- Locks
	- Semaphores
	- Transactions
	- Barriers
	- Message queues
	- Events
	- Futures/Promises
	- Awaiting asynchronous operations
- The mechanism depends on the problem you're solving.

### Synchronization vs. Locking
- Interviewers sometimes use these terms interchangeably, but they're not the same.
	- Locking controls **who** can access a resource.
	- Synchronization controls **when** operations occur, preventing race conditions.
- **Locking** is about **mutual exclusion**.
- **Synchronization** is about **coordination**.
- **Synchronization establishes ordering guarantees.**

### Overall Pattern
```
Concurrent operations

↓

Race condition

↓

Missing cache

↓

Synchronize database commit
with cache initialization

↓

Deterministic ordering

↓

No race condition
```
- The [[Meeting Tight Deadline]] story is a beautiful systems-design story because you didn't just "fix a bug." Instead, you:
	- Identified a race condition.
	- Understood that retries wouldn't solve it.
	- Eliminated the race by synchronizing two dependent operations.
	- Improved both correctness and latency.
- That's a much stronger narrative than "I fixed a cache issue."

### Common Interview Questions
1. An ETL pipeline has two jobs:
	- Job A generates a daily customer file.
	- Job B loads that file into a data warehouse.
	- One day, Job B starts before Job A has finished writing the file.
	- **What problem could occur, and how would you synchronize the two jobs?**
> 	If Job B starts before Job A has finished writing the file, Job B could load incomplete or corrupted data to the data warehouse. Synchronizing the two jobs so that Job B must wait until Job A has successfully completed would resolve the issue. Rather than relying on timing, I'd have Job B start only after receiving a completion signal from Job A or use an orchestration tool like Airflow to enforce the dependency.
2. Suppose an engineer says, "I'll just add a 30-second delay before Job B starts. That should fix the problem." Why is that not true synchronization?
> 	This is not synchronization because it doesn't actually establish a relationship between two operations, it just waits an arbitrary, potentially unnecessary amount of time. If Job B's dependency is completely unreliable for some reason, waiting any amount of time won't solve the issue.
	1. Another failure mode to mention would be Job A succeeding, but taking more than 30 seconds to complete. In this scenario, the 30-second delay is ineffective.
	2. Similarly, if Job A finishes in just 5 seconds because of a smaller batch size, you've wasted 25 seconds.
3. The [[Meeting Tight Deadline]] story can now be explained as follows:
> 	The root cause was a race condition between the database commit and cache initialization. Requests could reach the cache before it had been populated, leading to intermittent failures. Rather than masking the issue with retries or fixed delays, I synchronized cache initialization with the successful database commit, establishing an ordering guarantee that eliminated the race condition entirely.
	- This explanation naturally incorporates the following topics:
		- Race conditions
		- Synchronization
		- Event-driven coordination
		- Why retries aren't sufficient
		- Why sleeping isn't synchronization
		- Stronger consistency for a critical workflow

## 12. Deployment Strategies

- Interviewers don't expect you to have operated a massive deployment platform. What they want to know is: "How would you reduce the risk of deploying new code to production?" **That's what deployment strategies are all about**.
- The safest deployment strategy is one that **limits the blast radius** if something goes wrong.
	- Suppose you've just finished implementing a new feature.
	- Everything passed:
		- Unit tests
		- Integration tests
		- Staging and Load tests
	- Does this guarantee the feature will work in production?
		- No. Tests can never account for all possible scenarios that occur in a live production environment:
			- real users
			- real traffic
			- unexpected edge cases
			- different scale

### Traditional ("Big Bang") Deployment
```
Old Version

↓

Deploy

↓

New Version
```
- This is the simplest deployment strategy. Every server is update at once and 100% of users are affected.
- Advantages:
	- Simple
	- Fast
	- Easy to understand
- Disadvantages
	- Highest deployment risk
	- Largest blast radius
	- Rollbacks can be disruptive

### Rolling Deployment
```
Server 1

↓

Server 2

↓

Server 3

↓

Server 4
```
- Instead of updating everything simultaneously, only a small portion of servers are updated at a time.
- During deployment, the old and new versions of the software temporarily coexist.
- This is much better than the Traditional deployment strategy because it reduces blast radius. If one server starts throwing errors, the rollout can be stopped while only a small percentage of users experience an issue.
- This strategy significantly reduces deployment risk.
- Advantages:
	- Low downtime
	- Lower risk
	- Efficient resource utilization
	- Easy to pause during rollout.
- Disadvantages:
	- For a short period of time, both versions of a service must work together. **This makes backward compatibility very important.**

### Blue-Green Deployment
```
Blue

(Current Production)

Green

(New Version)
```
- Instead of replacing servers gradually, two complete environments are maintained:
	- Blue contains the current production environment.
	- Green contains the new production environment.
- Users are initially directed the the Blue environment. After testing the Green environment, traffic switches.
	- If a rollback needs to be performed, the traffic is simply switched back.
- Advantages:
	- Very fast rollback
	- Minimal downtime
	- Easy to validate before switching
- Disadvantages
	- Requires two production environments
	- More infrastructure cost

### Canary Deployment
```
1%

↓

5%

↓

10%

↓

25%

↓

100%
```
- This is the strategy interviewers ask about the most.
- Instead of sending the new version to everyone, a canary deployment starts with a small percentage.
- The following metrics are monitored:
	- Error rates
	- Latency
	- CPU
	- Memory
	- Business metrics
- If everything looks healthy, the traffic is gradually increased. If not, the rollout is stoped and rolled back.
- Canary deployments are popular because they answer an important question before exposing a feature to every customer: "Does this version actually work in production?"
	- Many large companies—including Amazon—heavily use canary-style deployments.

### Comparing Strategies
| Strategy    | Risk   | Infrastructure Cost | Rollback Speed |
| ----------- | ------ | ------------------- | -------------- |
| Traditional | High   | Low                 | Slow           |
| Rolling     | Medium | Low                 | Moderate       |
| Blue-Green  | Low    | High                | Very Fast      |
| Canary      | Lowest | Medium              | Fast           |
- There isn't a universal best strategy. The right choice depends on:
	- System criticality
	- Infrastructure budget
	- Rollback requirements
	- Deployment frequency

### Common Examples
```
Old Pipeline

↓

Warehouse A

New Pipeline

↓

Warehouse B
```
- Suppose you've changed a Spark job that calculates customer lifetime value. Would you immediately replace the existing job?
	- Probably not. A safer approach would be to run both versions in different environments (warehouses).
	- Next, you would compare outputs.
	- If they match, you would promote the new pipeline to all production environments.
	- This is conceptually similar to a Blue-Green Deployment.
- Another example would be deployment an optimized Kafka consumer. Instead of simultaneously replacing all consumers, you replace one, then monitor lag, throughput, and failures. If everything is healthy, you'd roll out the change to the rest of the consumers.
- Deploying the cache initialization fix from the [[Meeting Tight Deadline]] story might look like:
	- Deploy to one region.
	- Monitor:
		- cache hit rate
		- latency
		- error rate
		- customer impact
	- If everything looks good, continue the rollout.

### Common Metrics During Deployment
- A successful deployment isn't just "the application started."
- Engineers monitor metrics such as:
	- Error rate
	- Latency
	- Throughput
	- CPU utilization
	- Memory usage
	- Request success rate
	- Customer-facing metrics (e.g., checkout success, ad delivery rate)
	- **Business KPIs**
- The last point is often overlooked. A deployment can be technically healthy but still hurt the business (for example, by reducing conversion rates).

### Common Interview Questions
1. Why do companies use canary deployments instead of just testing more thoroughly before release?
> 	Because no testing environment perfectly replicates production. Canary deployments allow engineers to validate behavior under real production traffic while limiting the impact if unexpected issues occur.
2. You've deployed a new version of your ETL pipeline using a canary deployment. Only **5%** of traffic is using the new version. You notice:
	- CPU usage is normal.
	- Memory usage is normal.
	- Error rate is normal.
	- **The number of processed records is 20% lower than expected.**
	- Would you continue the rollout? Why or why not?
> 	I wouldn't continue the rollout until I understood the discrepancy. I'd compare the canary's output with the previous version, review metrics and logs for processing failures, and verify whether the lower record count was an expected result of the change. If it wasn't expected, I'd pause the rollout while the team investigated before exposing more traffic to the new version.
3. If blue-green deployments have such fast rollbacks, why doesn't every company use them?
> 	Blue-Green deployments require maintaining two production environments, which increases infrastructure cost and operational complexity. While they provide very fast rollbacks, they typically switch all traffic at once. Canary deployments, on the other hand, gradually expose the new version to increasing percentages of production traffic, making it easier to detect issues before they affect the entire user base.
	- A Blue-Green deployment does not isolate a new version of a service to a subset of users in the same way a Canary deployment does. Blue-Green uses two complete prodution environments.
	- Often, 100% of traffic is switched from Blue to Green. The isolation comes from the fact that the old environment is still available for an immediate rollback—not because only a subset of users are on Green.
	- Changes can be rolled much more quickly in a Blue-Green deployment than a Canary deployment.
	- Blue-Green asks "Can I switch traffic instantly and roll back instantly?"
	- Canary asks "Should I trust this release with all users yet?"

## 13. Shadow Traffic

- Instead of only sending requests to a production service:
	```
	User
	
	↓
	
	Current Service
	```
- You secretly duplicate the request:
	```
	                → Current Service (response returned)
	User Request →
	                → New Service (response ignored)
	```
	- The customer only sees the response from the current production service.
	- The new service receives the exact same request, but **the response is discarded**.
	- The user never knows a shadow request was sent to the new service.
- Shadow traffic is useful because it tells you how a brand new or modified service will behave in production. You don't want customers using it yet, but you want to see how it behaves.
	- The user doesn't know what's happening behind the scenes, but engineers can see the following metrics under real production load:
		- CPU usage
		- Memory usage
		- Latency
		- Error rates
		- Database queries
		- External API call

### Shadow Traffic vs. Canary
- Canary deployments are different than shadow traffic in that real customers are receiving responses from the new service and can be impacted by potential failures.

|Canary|Shadow Traffic|
|---|---|
|Users receive new responses|Users receive old responses|
|Small customer risk|No customer risk|
|Validates customer experience|Validates operational behavior|
|Can detect business logic bugs|Great for performance and stability testing|

### Limitations
- Interviewers love asking "Can shadow traffic fully validate a deployment?"
	- The answer is no, because the responses aren't actually used in the workflow.
- Shadow traffic can only tell you about:
	- performance
	- resource usage
	- crashes
	- latency
- **It doesn't tell you if the business logic is correct**. That's why engineers will often compare responses offline after the requests are complete.

### Common Examples
- **Amazon Example:** Imagine your cache initialization fix. Suppose you rewrote the cache population logic. Would you immediately let production use it?
	- Probably not. Instead:
		```
		Production Request
		
		↓
		
		Current Cache Logic
		
		↓
		
		Return Response
		
		↓
		
		Mirror Request
		
		↓
		
		New Cache Logic
		
		↓
		
		Measure latency
		Measure cache hits
		Measure errors
		```
		- Customers remain unaffected while you gather confidence.
- **Data Engineering Example:** Suppose you've rewritten an ETL pipeline in Spark.mInstead of replacing the old pipeline, you run both:
	```
	Production Events
	
	↓
	
	Old Pipeline
	
	↓
	
	Warehouse A
	
	Same Events
	
	↓
	
	New Pipeline
	
	↓
	
	Warehouse B
	```
	- Next, you compare the following before promoting the new pipeline:
		- Row counts
		- Aggregates
		- Processing time
		- Error rates
		- Resource usage
	- This is essentially shadow traffic for data pipelines.

### Tradeoffs
- Advantages:
	- Zero customer impact.
	- Real production traffic.
	- Excellent for performance validation.
	- Great for observing resource usage.
	- Safe way to validate major architectural changes.
- Disadvantages:
	- Requires extra infrastructure.
	- Doesn't validate the customer experience directly.
	- Doubles some compute costs.
	- Downstream systems may need protection so mirrored requests don't cause duplicate side effects.
		- For example, if the mirrored request actually charges a credit card, you've got a problem.
		- Shadow environments often stub out or disable side effects like payments, emails, and notifications.
- **Shadow traffic validates operational behavior, while canary deployments validate customer behavior**.

### Hybrid Deployment Strategies
- Many people think of deployment techniques as alternatives, but in mature engineering organizations they're often **combined**.
- For example, a rollout might look like:
	```
	Development
	
	↓
	
	Testing
	
	↓
	
	Shadow Traffic
	(Does it behave correctly?)
	
	↓
	
	Canary
	(Do customers experience it correctly?)
	
	↓
	
	Rolling Deployment
	(Gradually replace servers)
	
	↓
	
	100% Production
	```
- Each stage anwers a different question:

|Stage|Question|
|---|---|
|Shadow|Can the new system handle real traffic?|
|Canary|Do real users have a good experience?|
|Rolling|Can we safely scale the deployment?|
- This layered approach reduces risk at each step.
### Common Interview Questions
1. You're rewriting a Kafka consumer that processes millions of events per day. Would you choose a **canary deployment** or **shadow traffic** for the first production test? Why?
> 	It depends on what I'm trying to validate. If I've significantly changed the internals of the consumer and want to evaluate throughput, latency, resource usage, or stability under real production load without affecting customers, I'd start with shadow traffic. If I need to validate the actual customer-facing behavior of the new version, I'd move to a canary deployment that gradually exposes real users while limiting risk.
2. Suppose your shadow deployment sends mirrored requests to a service that processes credit card payments. What problem could occur, and how would you prevent it?
> 	I'd ensure the mirrored requests can't produce real side effects. That could mean routing them to sandbox versions of downstream services, mocking payment processors, or configuring the shadow environment to skip external writes while still collecting metrics.
	- Generally speaking, read-only operations are generally safe to mirror. Write operations require additional safeguards.

## 14. Feature Flags

- Many people think a deployment automatically means users get a new feature. **Feature flags break that assumption**.
- Imagine you've spent three months building a new recommendation engine. It's ready, but the Marketing department tells you that they're launching next Tuesday.
	- Instead of waiting until Tuesday to deploy, the feature can be deployed right away, but enabled later using a feature flag:
		```
		Deploy Code
		
		↓
		
		Feature Disabled
		
		↓
		
		Later...
		
		↓
		
		Enable Feature
		```
		- The code is in production, it's just not active until the it is explicitly enabled using the feature flag.
- Example Implementation:
	```python
	if feature_enabled:
	    use_new_recommendation_engine()
	else:
	    use_old_recommendation_engine()
	```
	- Changing the feature flag determines which code path is executed. No new deployments are required.
- Feature flags are commonly used for:
	- Gradual rollouts
	- A/B testing
	- Beta features
	- Internal testing
	- Emergency feature disablement
	- Regional launches
	- Customer-specific features
- One subtle, but important point is that feature flags **cannot be used as a form of access control**. Feature flags only determine **whether code executes**. Authorization determines **who is allowed** to execute it.

### Common Examples
- Suppose it's Black Friday. Do you really want to deploy brand-new code?
	- Probably not. Instead:
		- Deploy it weeks earlier.
		- Verify everything is stable.
		- When the business is ready, flip the feature flag.
	- The deployment risk and the business release become separate events.
- Suppose you're launching a redesigned checkout phase
	- Without a feature flag:
		```
		Deploy
		
		↓
		
		Everyone Uses New Checkout
		```
		- If there's a bug, everyone is affected.
	- With a feature flag:
		```
		Deploy
		
		↓
		
		Feature Off
		
		↓
		
		Internal Employees
		
		↓
		
		10% Users
		
		↓
		
		50%
		
		↓
		
		100%
		```
		- You control exposure independently of deployment.

### Feature Flag vs. Canary
- Canary deployments are gradual. Some servers will run Version X of a software while other servers run Version Y.
- A feature flag is usually binary. While it's off, every server runs the old logic. While it's on, every server runs the new logic. However, only one version of the software is deployed (the version with the feature flag).

| Canary                         | Feature Flag                |
| ------------------------------ | --------------------------- |
| Controls deployment            | Controls functionality      |
| Different software versions    | Same software version       |
| Gradual infrastructure rollout | Gradual feature rollout     |
| Often infrastructure-managed   | Usually application-managed |
- The two approaches can be combined by using a percentage feature flag, instead of a binary feature flag. For example:
	1. Deploy new code.
	2. Keep feature disabled.
	3. Enable for internal employees.
	4. Enable for 1%.
	5. Enable for 10%.
	6. Enable for everyone.
- The deployment happended once, but the rollout happened gradually because of the feature flag.
- Data engineers frequently use feature flags for:
	- New transformation logic
	- New validation rules
	- New data quality checks
	- New enrichment sources
	- Alternative algorithms

### Feature Flag vs. Rollback
- Suppose a new version of software introduces:
	- Feature A
	- Feature B (Performance improvements)
- If Feature A is broken and there are no feature flags, everying (including Feature B) must be rolled back.
- With feature flags, Feature A can be disabled while Feature B (and its performance improvements) remain in effect.

### Tradeoffs
- Advantages:
	- Separate deployment from release.
	- Instant enable/disable.
	- Easy experimentation.
	- A/B testing.
	- Safer releases.
	- Faster recovery from feature-specific issues.
- Disadvantages:
	- Over time, as more flags are added, your code becomes difficult to understand and maintain. For example:
		```
		if flag_A:
		
		else:
		
		if flag_B:
		
		else:
		
		if flag_C:
		
		else:
		```
	- This is called flag debt. To avoid this, companies usually remove feature flags (along with the older versions of code) when they're no longer needed.

### Common Interview Questions
1. Your team has deployed a new search algorithm behind a feature flag. After enabling it for **10%** of users, search latency increases significantly. What would you do next? Why is this preferable to performing a full rollback?
> 	I would dial back the feature flag to 0% and investigate the issue. This is preferable to performing a full rollback because that takes longer and may not actually be necessary, depending on the cause of the latency spike. I'd also look at the metrics segmented by users with the feature enabled versus disabled to confirm the latency increase is actually correlated with the new feature.
2. If feature flags let you turn features on and off instantly, why do companies still need deployment strategies like canary or blue-green?
> 	Feature flags and deployment strategies solve different problems. Feature flags control whether a feature is enabled after the application is running, while deployment strategies control how new software versions are introduced into production. If the new version can't start, crashes on launch, or has infrastructure-level issues, feature flags can't help because the application never reaches the point where it can evaluate them. That's why companies still use strategies like canary, rolling, and blue-green deployments alongside feature flags.

| Concern                                       | Tool                |
| --------------------------------------------- | ------------------- |
| Is the application healthy?                   | Deployment strategy |
| Should users see the feature?                 | Feature flag        |
| Can the new system handle production traffic? | Shadow traffic      |
| What if something goes wrong?                 | Rollback strategy   |

## 15. Rollback Strategies

- No matter how much testing you perform:
	- Unit tests
	- Integration tests
	- Staging
	- Canary
	- Shadow traffic
- Something can still go wrong in production.
- A rollback strategy answers: "How do we return the system to a known-good state?"

### Overview
- A rollback is the process of restoring the previous working version after a failed deployment:
	```
	Version 1
	
	↓
	
	Deploy Version 2
	
	↓
	
	Problems Found
	
	↓
	
	Restore Version 1
	```
	- The primary goal is to minimize recovery time and customer impact.
- A rollback is a **business decision**, it's not an automatic response to issues faced in production. For example:
	- If error rates increased by 0.01% after a deployment, you probably wouldn't roll back.
	- If checkout failures increased by 30% after a deployment, you almost certainly would roll back.
- The decision to roll back ultimately depends on:
	- Customer impact
	- Business impact
	- Severity
	- Availability of a workaround

### Common Rollback Strategies
1. Traditional Rollback
	```
	Version 2
	
	↓
	
	Deploy Version 1
	```
	- Deploy the previous version.
	- Simple. But it takes time.
1. Rolling Rollback
	```
	New
	
	↓
	
	Old
	
	↓
	
	Old
	
	↓
	
	Old
	```
	- Replaces servers gradually, just like a rolling deployment.
	- Useful for large fleets.
	- Recovery is gradual.
1. Blue-Green Rollback
	```
	Blue
	
	(Current)
	
	Green
	
	(New)
	```
	- No deployment required.
	- Simply change the traffic routing from Green to Blue.
	- This is one reason Blue-Green deployments are popular. Rollback often takes seconds, instead of minutes or hours.
1. Feature Flag Rollback
	```
	Feature Enabled
	
	↓
	
	Feature Disabled
	```
	- If the deployment itself is unhealthy, but only one feature is problematic, this strategy allows you to disable the feature without a new deployment or rollback.
	- Often leads to the fastest recovery.
1. Canary Rollback
	```
	5%
	
	↓
	
	New Version
	```
	- If metrics worsen when the feature is introduced to 5% of production traffic, you can stop and rollback. The remaining 95% of traffic was never affected.
	- This is how Canary Deployments reduce blast radius.

### Rollback vs. Hot Fix
- If a bug is discovered after a deployment, do you roll back or fix it immediately?
	- It depends. A rollback is usually preferable when:
		- Recovery is fast.
		- Previous version is stable.
		- Customer impact is significant.
	- A hot fix may be better when:
		- Rolling back would break another critical feature.
		- The bug is isolated and easily fixed.
		- The previous version is incompatible with recent data or schema changes.
- The hardest rollback problem involves changes to a database schema:
	```
	Version 1
	
	↓
	
	Schema A
	
	↓
	
	Deploy
	
	↓
	
	Schema B
	```
	- When you try to rollback, Version 1 doesn't understand Schema B.
	- This is why database changes require special care.

### Backward-Compatible Migrations
- Experienced teams often use **expand-and-contract** migrations.
	- Instead of:
		```
		Remove Old Column
		
		↓
		
		Add New Column
		```
	- They do:
		```
		Add New Column
		
		↓
		
		Application Writes Both
		
		↓
		
		Migrate Data
		
		↓
		
		Switch Reads
		
		↓
		
		Remove Old Column
		```
	- The old column is gradually deprecated and removed after the new column is added.
	- This allows a rollback to remain possible.

### Common Examples
- **Data Engineering Example:** Suppose you've deployed an ETL pipeline.
	- After the deployment:
		- Record counts drop.
		- No processing errors.
		- Dashboard metrics are incorrect.
	- In this scenario, you usually wouldn't keep debugging in production. If business reporting is affected, you would:
		- Restore the previous pipeline.
		- Investigate offline.
	- **Correctness is more important than experimenting in production.**
- **Amazon Example:** Imagine the cache initialization fix.
	- After the deployment:
		- Error rate decreases.
		- Latency doubles.
	- Before rolling back you would ask:
		- Is latency within our SLA?
		- Is customer experience worse?
		- Did we accidentally introduce blocking?
		- Is the synchronization mechanism causing lock contention?
	- The answer isn't automatically "rollback." It's **evaluate the business impact first**.
- It's important to keep in mind that rollback **doesn't always equate to failure**. Senior engineers view rolling back as a natural part of a safe deployment process.
- The easier it is to rollback, the more confident a team becomes at shipping frequently.
- **The best rollback is the one you've already planned before deployment.**

### Putting It All Together
- Here's how all of these concepts fit together:
	```
	Develop Feature
	
	↓
	
	Deploy New Version
	
	↓
	
	Shadow Traffic
	(Operational validation)
	
	↓
	
	Canary
	(Limited customer exposure)
	
	↓
	
	Feature Flag
	(Control who sees the feature)
	
	↓
	
	Monitor Metrics
	
	↓
	
	Healthy?
	
	├── Yes → Increase rollout
	│
	└── No
	      │
	      ├── Disable feature flag (feature issue)
	      │
	      ├── Stop canary rollout
	      │
	      ├── Switch Blue → Green
	      │
	      └── Roll back deployment
	```
	- Each technique reduces risk at a different stage.
	- They compliment each other. They don't replace one another.

| Goal                          | Technique                        |
| ----------------------------- | -------------------------------- |
| Validate operational behavior | Shadow traffic                   |
| Limit customer exposure       | Canary deployment                |
| Deploy without releasing      | Feature flags                    |
| Replace infrastructure safely | Rolling / Blue-Green deployments |
| Recover quickly               | Rollback strategies              |

### Common Interview Questions
1. Your team deployed a new version of an ETL pipeline.
	- Ten minutes later:
		- CPU is normal.
		- Memory is normal.
		- No exceptions are logged.
		- **Executive dashboards show revenue has dropped by 40%**.
		- Would you immediately roll back? Why or Why not?
> 	A 40% revenue drop is severe enough that I'd immediately pause the rollout and begin investigating. I'd compare the new pipeline's outputs against the previous version, examine data quality metrics such as row counts and aggregates, and determine whether the discrepancy is caused by the deployment. If the evidence suggests the pipeline is producing incorrect business data, I'd roll back to restore accurate reporting while continuing the investigation offline.
			- You don't rollback immediately. Instead you:
				- Pause the rollout.
				- Quickly investigate.
				- Rollback if the deployment is responsible.
2. If you can simply disable a feature flag, why would you ever need a rollback?
> 	Feature flags only control whether specific functionality is enabled after the application is running. If the deployment itself has problems—such as startup failures, configuration issues, dependency incompatibilities, or memory leaks—a feature flag can't help because the application may never reach the point where it can evaluate the flag. In those cases, a deployment rollback is the appropriate recovery strategy.