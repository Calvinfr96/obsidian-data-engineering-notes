## 1. Why Kafka Exists

- **Don't think of Kafka as a messaging system first.** Think of it as a **distributed event streaming platform**.
- **The Problem:**
	- Imagine an e-commerce website. When a customer places an order, several systems need to react immediately, including:
		- Inventory Service
		- Payment Service
		- Shipping Service
		- Recommendation Engine
		- Analytics Pipeline
		- Fraud Detection
		- Email Notifications
	- That's seven different consumers.
- **Without Kafka:**
	- The Order Service might call downstream service directly. This can cause several problems.
	- **Problem 1: Tight Coupling**
		- The Order Service must know:
			- Every downstream service
			- Where it's deployed
			- How to communicate with it
			- What happens if it's unavailable
		- Every new consumer requires modifying the Order Service.
		- This is known as **tight coupling**.
	- **Problem 2: Failures Spread**
		- If the Email Service crashes, should customers be prevented from placing orders?
			- Of cource not.
		- However, if the Order Service waits for every downstream consumer, one delay or failure could block the entire request.
	- **Problem 3: Scalability**
		- If the Order Service goes from having 3 consumers to over 30 consumers, now every order triggers 30 network calls.
		- This doesn't scale well.
- **Kafka's Idea:**
	- Instead of sending data directly to every service, publish the event source and allow interested services to consume it independently.
	- The Order Service doesn't need to know or care who consumes the order event.
	- This is called **decoupling**.
	- Publishing events that downstream services subsequently consume independently is called **Event-Driven Architecture**.
- **Newspaper Analogy:**
	- When a publisher prints copies of a newspaper, they don't call each subscriber individually to make sure they get a copy.
	- Readers choose whether to subscribe.
	- Kafka works the same way. A **producer** publishes an event once and **consumers** subscribe if they're interested.
- **Advantages:**
	- Decoupling:
		- Producers don't know or care who consumers are.
		- Consumers don't know who the producer is beyond the event format.
		- **Each side can evolve independently**.
	- Reliability:
		- Consumers can operate independently.
		- If one consumer goes down, they can catch up on missed events by themselves. The other consumers aren't affected.
	- Scalability:
		- Consumers can simply subscribe to a producer to receive events. Nothing needs to be done on the producer side since the event is only published once.
	- Replay:
		- Events remain in Kafka for a configurable retention period.
		- This allows consumers to read events they missed during downtime.
		- Traditional message queues often remove messages immediately after they're consumed.
		- Kafka's retention model allows multiple consumers to read the same event at different times.

### Tradeoffs
- Kafka isn't free. It introduces:
	- Additional infrastructure
	- Operational complexity
	- Monitoring requirements
	- Partition management
	- Ordering considerations
- For a tiny application with two services, direct API calls may be simpler.
- As systems grow, Kafka's benefits usually outweigh its complexity.

### Mental Model
- Connection to Previous Topics:
	```
	Producer
		  │
		  ▼
	Kafka
		  │
		  ▼
	Consumers
		  │
		  ▼
	Backpressure
		  │
		  ▼
	Exactly-once
		  │
		  ▼
	Watermarks
		  │
		  ▼
	Streaming Pipelines
	```

### Common Interview Questions
1. Why use Kafka instead of direct service-to-service communication?
> 	Kafka decouples producers from consumers. Producers publish events without needing to know which services consume them. This improves scalability, fault tolerance, and flexibility because new consumers can be added without changing producers, and consumers can recover from downtime by replaying retained events.
2. Your company has an **Order Service** that currently makes synchronous API calls to:
	- Inventory
	- Billing
	- Shipping
	- Analytics
	- Engineers propose replacing those direct calls with Kafka. What advantages would Kafka provide? Are there any tradeoffs?
> 	I would only recommend switching from synchronous API calls to Kafka if the service could significantly benefit from Kafka's advantages, such as decoupled, event-driven architecture, fault-tolerance, and scalability. If the service would only see negligible benefits, then the additional infrastructure, operational complexity, and monitoring requirements wouldn't be worth it. With synchronous API calls, if the Shipping service is slow or unavailable, the Order Service may also become slow or fail. With Kafka, the Order Service can publish the event and continue, allowing downstream consumers to process it independently.
3. A teammate says: "Kafka is just another message queue." How would you respond? What similarities and differences would you point out?
> 	I would mention that Kafka is a distributed event streaming service. Traditional message queues are often designed to distribute work, where a message is consumed and then removed. Kafka is designed as an append-only event log with configurable retention, allowing multiple consumers to independently read the same events and even replay historical data.
	- Kafka doesn't send messages consumers. It simply publishes messages for a fixed abount of time (configurable retention period), allowing consumers to independtly pull and read messages.
4. If Kafka stores events after they've been consumed, won't it eventually run out of space?
> 	Kafka uses configurable retention policies. Events are typically retained for a fixed period of time or until a storage limit is reached. This allows consumers to replay recent events without requiring Kafka to store data indefinitely.
	- Kafka is meant to act as **durable** storage for recent events, not permanent storage.

## 2. Topics, Partitions, and Offsets

- A **topic** is a logical category of events. For example, an `Orders` topic could contain `OrderCreated`, `OrderUpdated`, and `OrderCancelled` events.
	- Topics help to organize related events.
	- Different consumers apply to topics they care about.
- A topic is divided into **partitions**.
	- Instead of storing events in one huge log file, Kafka stores several smaller log files.
	- Each partition lives in one broker.
	- Partitions are important because they enable **parallelism**. Instead of one consumer processing all events in a topic, partitions allow multiple consumers to process events simultaneously, which is much faster.
	- Similar to databases, Kafka decides how to split events amongst partitions based on a **partition key**.
	- **Kafka guarantees ordering within a partition**. Not across partitions.
- An **offset** is simply the numerical position of an event within a partition.
	- Offsets are important because they allow Kafka to "pick up where it left off" if a producer or consumer crashed while processing data.
	- This is a lot more preferable than reprocessing all of the events in the stream.
	- Offsets are **per partition**, they are not **not globally unique**. For example:
		```
		Orders Topic
		
		Partion 0: Offset 0:
		Order A
		
		Partition 0: Offset 1:
		Order B
		
		Partition 1: Offset 0:
		Order C
		
		Partition 1: Offset 1:
		Oder D
		```
		- Offsets allow **each partition** to individually track what events it has processed.

### Tradeoffs
- More Partitions:
	- Pros:
		- More parallelism
		- Higher throughput
		- More consumers can work independently
	- Cons:
		- More coordination
		- More metadata
		- Ordering only exists within a partition
		- Increased operational complexity
- Fewer Partitions:
	- Pros:
		- Simpler
		- Easy ordering
	- Cons:
		- Lower throughput
		- Limited scalability

### Mental Model
- Connection to Previous Topics:
	```
	Database Partitioning
	        │
	        ▼
	Spark Partitions
	        │
	        ▼
	Kafka Partitions
	        │
	        ▼
	Consumer Parallelism
	        │
	        ▼
	Throughput
	```

### Common Interview Questions
1. Why does Kafka use partitions?
> 	Partitions allow Kafka to scale horizontally by distributing events across multiple logs that can be processed in parallel. They increase throughput, but ordering is only guaranteed within each partition, so choosing an appropriate partition key is important when event order matters.
2. Your team has a Kafka topic with **one partition**, but message volume has increased significantly and consumers can't keep up. What change would you recommend? What tradeoffs come with that change?
> 	Since the topic only has one partition, I would recommend increasing the number of partitions. This would enable greader parallelism and higher throughput because more partitions can be distributed more evenly among consumers. Increasing the number of partitions improves parallelism and throughput, but ordering is only guaranteed within each partition, so choosing an appropriate partition key becomes even more important.
3. A teammate says: "Kafka guarantees that all messages are processed in the exact order they were produced." Would you agree? If not, what is Kafka's actual ordering guarantee, and how does the partition key influence it?
> 	I would clarify that Kafka only guarantees message ordering within a partition. Choosing the correct number of partitions comes down to striking the right balance between parallelism and the need for ordering guarantee. The partition key determines which partition an event is written to. Events with the same partition key are consistently routed to the same partition, preserving their relative order. Events with different keys may be placed in different partitions, where no ordering is guaranteed between them.
4. If ordering is important, why not just always use one partition?
> 	Using a single partition preserves ordering for all events, but it limits throughput because only one consumer can process that partition at a time. Multiple partitions allow Kafka to scale horizontally, so the partitioning strategy depends on whether throughput or ordering is the higher priority.
5. How do you choose a partition key?
> 	I choose a key that groups events requiring ordering while still distributing load evenly across partitions.

## 3. Producers and Consumers

- Imagine a ride-sharing app. When a customer requests a ride, the application creates a `RideRequested` event. The application doesn't write this event directly to a partition. Instead, it uses a producer:
	```
	Application
	
	↓
	
	Producer
	
	↓
	
	Kafka
	```
- On the other side:
	```
	Kafka
	
	↓
	
	Consumer
	
	↓
	
	Dispatch Service
	```
- The producer writes while the consumer reads.
- A **producer** simply an application that publishes records to a Kafka topic. Examples include:
	- Order Service
	- Payment Service
	- Inventory Service
	- Mobile App
	- IoT Device
- When a producer publishes a message, it decides:
	- Which topic?
	- Which partition? (using the partition key)
	- When to retry?
	- How many acknowledgments to require?
- Whe a producer writes a message to a Kafka topic, it uses **acknowledgements** to decide when a write is considered successful.
	- A producer can choose different acknowledgement levels (`acks`):
		- When `acks = 0`, the producer sends a message and returns immediately.
			- Pros: Lowest latency
			- Cons: Messages may be lost
		- When `acks = 1`, the producer sends a message, then waits for the **leader partition** to confirm the right.
			- Pros: Better durability
			- Cons: Leader could fail before followers replicate the data
		- When `acks = all`, the producer sends a message, then waits for **all required replicas** to acknowledge.
			- Pros: Highest durability
			- Cons: Highest latency
- **Producer Retries:**
	- When a producer sends a message to a topic and there is a network interruption before it receives an acknowledgement, it will retry.
	- Retrying introduces the possibility of duplicates, which is why it's important for downstream systems to be idempotent.
- **Idempotent Producers:**
	- Kafka supports idempotent producers. These producers attach an identifier to each message they send.
	- If Kafka receives the same message more than once because of retries, it recognizes the duplicate and only one message is written to the topic.
- A **consumer** simply an application that reads records from a Kafka topic. Examples include:
	- Analytics Pipeline
	- Fraud Detection
	- Cache Warmer
	- Billing
	- Recommendation Engine
- **Consumer Workflow:**
	- A consumer repeatedly does the following as it consumes messages from a Kafka topic:
		```
		Read
		
		↓
		
		Process
		
		↓
		
		Commit Offset
		
		↓
		
		Repeat
		```
	- Notice that the offset is committed **after processing has completed**.
	- If a consumer commits too early and crashes, a message could be lost.
	- If a consumer commits too late and crashes, a message could be processed more than once.
- **Offset Commits:**
	- An offset commit means, "I've successfully processed everything before this offset."
	- Kafka doesn't check the underlying business logic. It trusts the consumer.
	- This is why committing offsets **at the right time** is so important.

### Tradeoffs
- `acks = 0`:
	- Pros: Fastest
	- Cons: Least durable
- `acks = 1`:
	- Pros: Good balance
	- Cons: Possible data loss if the leader fails before replication
- `acks = all`:
	- Pros: Highest durability
	- Cons: Increased latency
- Commit Offsets Early:
	- Pros: Fewer duplicates
	- Cons: Risk of losing unprocessed messafes
- Commit Offsets After Processing:
	- Pros: Avoid data loss
	- Cons: Duplicates are possible if a crash occurs before the commit.

### Mental Model
- Connection to Previous Topics:
	```
	Producer
	      │
	      ▼
	Acknowledgments
	      │
	      ▼
	Retries
	      │
	      ▼
	Duplicates
	      │
	      ▼
	Idempotency
	      │
	      ▼
	Consumer Processing
	      │
	      ▼
	Offset Commits
	      │
	      ▼
	Exactly-once Semantics
	```
- Consumer Message Processing:
	```
	Consumer reads message
	          │
	          ▼
	Business logic runs
	          │
	          ▼
	Database updated
	          │
	          ▼
	Offset committed
	          │
	          ▼
	Crash?
	          │
	     ┌────┴────┐
	     ▼         ▼
	Commit      No Commit
	     │         │
	     ▼         ▼
	Continue   Redelivery
	                  │
	                  ▼
	           Idempotent Processing
	```
### Common Interview Questions
1. When should a Kafka consumer commit its offset?
> 	Typically after successfully processing the message. Committing too early risks data loss if processing fails, while committing afterward may result in duplicate processing after a crash. Many systems accept at-least-once delivery and make downstream processing idempotent to safely handle retries.
2. You're building a pipeline that processes customer orders. After reading a message from Kafka, your consumer:
	1. Updates a database.
	2. Commits the Kafka offset.
	- Suppose the database update succeeds, but the application crashes **before** committing the offset. What happens when the consumer restarts? Would you consider this a bug? Why or why not?
> 	If application crashes before committing the offset, the consumer will attempt to update the database again, potentially allowing duplicate records to be written. This isn't a. bug, it's a useful feature. If the database update had failed, the application should retry. The system can prevent processing errors by making the update idempotent. One way to do this would be to use upsert logic to perform the update.
3. A teammate says: "To avoid duplicates, let's commit the Kafka offset immediately after reading the message, before we process it." What problem does this create, and why do many Kafka consumers intentionally commit offsets only after successful processing?
> 	I wouldn't recommend committing the offset before processing. If the consumer crashes after committing but before finishing the work, Kafka assumes the message was already processed and won't redeliver it, resulting in data loss. That's why many Kafka consumers commit offsets only after successful processing, even though it means duplicates are possible after a crash. Those duplicates are typically handled through idempotent processing.
4. Can Kafka itself tell whether your database update succeeded?
	- No. Kafka only knows whether:
		- The consumer read a message.
		- The consumer committed an offset.
		- Kafka has **no knowledge** of what your application did.

## 4. Consumer Groups

- Consumer groups are one of the most important concepts in Kafka because they explain **how Kafka scales horizontally while avoiding duplicate work**.
- A consumer group is a set of consumers that cooperate to process a topic.
- Within a consumer group:
	- Each partition is assigned to **exactly one consumer**.
	- Consumers split the workload.
	- No duplicate processing occurs **within the group**.
	- This is how Kafka supports both:
		- Fan-out between applications
		- Parallelism within an application.
- Partition count limits parallelism because Kafka can only assign one partition to one consumer. If there are more consumers in a group than there are partitions in a topic, some consumers will sit idle.
	- If there are more partitions than consumers in a group, then one consumer will be assigned multiple partitions.
- **Rebalancing:**
	- Kafka handles consumer failure by redistributing partitions among remaining consumers. This is known as **rebalancing**.
	- Kafka also rebalances when consumers are added to a group. This ensures work is evenly distributed.
	- Rebalancing is useful, but it isn't free. During rebalancing:
		- Partition ownership changes.
		- Consumers may pause briefly.
		- Processing can temporarily stop while assignments are updated.
	- Modern Kafka has improved this (for example, with cooperative rebalancing), but it's still something to be aware of in production.

### Tradeoffs
- More Consumers:
	- Pros:
		- Higher throughput (up to the number of partitions)
		- Better fault tolerance
		- Faster processing
	- Cons:
		- Rebalancing overhead
		- No benefit when the number of consumers exceeds the number of partitions
- More Partitions:
	- Pros:
		- Higher maximum parallelism
		- Better scalability
	- Cons:
		- More operational complexity
		- More metadata
		- Ordering is only guaranteed within a single partition

### Mental Model
- Connection to Previous Topics:
	```
	Topic
	    │
	    ▼
	Partitions
	    │
	    ▼
	Offsets
	    │
	    ▼
	Consumers
	    │
	    ▼
	Consumer Groups
	    │
	    ▼
	Parallel Processing
	```
- Imagine a restaurant:
	- Waiters bring orders to the kitchen.
	- The kitchen has four chefs.
	- The head check assigns each ticket to one chef. This prevents duplicate work.
	- Now imagine another restaurant in the same building:
		- This restaurant receives copies of the same catering requests, but they prepare deserts instead of entrées.
		- This is like a different **consumer group**. Each group performs a different task with the same message.
	- Each group divides work internally, while multiple groups can each process the same events for different purposes.

### Common Interview Questions
1. Why can't two consumers in the same consumer group read the same partition?
> 	Because a consumer group is intended to divide work among its members. Assigning a partition to multiple consumers in the same group would result in duplicate processing. Kafka ensures each partition is owned by only one consumer within a group while allowing different consumer groups to independently consume the same topic. This allows each group to perform separate tasks with the same message, while internally ensuring tasks aren't performed more than once.
2. Your Kafka topic has **6 partitions** and your consumer group has **3 consumers**. How will Kafka distribute the work? If you later increase the consumer group to **8 consumers**, what happens?
> 	Kafka will assign 2 partitions to each of three consumers. If the consumer group increases to 8 consumers, Kafka will perform rebalancing and assign each consumer 1 partition. However, 2 of the 8 consumers would remain idle.
3. A teammate says: "If we want Billing and Analytics to both process every order event, we should put them in the same consumer group." Would you agree? If not, how should consumer groups be organized, and why?
> 	No, we should not put them in the same consumer group. Billing and Analytics each perform different work with the messages in a topic. Assigning them to the same consumer group would mean each message is only partially processed. Some messages would only have the billing work processed, while others would only have the analytics work processed. If you wanted to place them all in the same consumer group, order events would need to be split into billing and anlytics events and those events would need to be assigned to different topics. The billing consumers would subscribe to the billing topic and the analytics consumers would subscribe to the analytics topic. This changes the event model and increases complexity for creating and assigning messages.
	- The first approach (one broad message) is highly preferred in distributed systems because it **keeps producers and consumers decoupled**. If a new consumer joins, the producer doesn't need to create a specialized event topic for them to subscribe to.
4. If I have 20 partitions and only one consumer, is that bad?
> 	Not necessarily. One consumer can own multiple partitions. The system will still function correctly, although throughput may be lower than if multiple consumers processed partitions in parallel. Whether that's a problem depends on the workload and latency requirements.

## 5. Ordering Guarantees

- Kafka can only guarantee message ordering **within a single partition**, not across multiple partitions in a topic.
- Kafka can't guarantee global ordering because each partition is its **own independent log**. Furthermore, different partitions can be assigned to different consumers at different speeds.
	- If Kafka guaranteed global ordering across partitions, it would greatly reduce parallelism because each consumer would potentially need to wait for another consumer to finish processing a message before it could move on to its next message. **Consumers would no longer be decoupled**.
- **Partition Keys and Ordering:**
	- Kafka uses a partition key to assign messages to a partition. If a partition key randomly assigned messages to a partition, ordering would not be guaranteed for related messages.
	- A good partition key should:
		1. Preserve ordering where required.
		2. Distribute load evenly.
		3. Avoid hot partitions.
- Kafka intentionally prioritizes scalability over ordering. In some specialized systems, you can come very close to both guaranteeing global ordering and maintaining high throughput. However, in most cases, you need strike a careful balance between the two priorities.

### Mental Model
- Connection to Previous Topics:
	```
	Partition Keys
	        │
	        ▼
	Ordering
	        │
	        ▼
	Parallelism
	        │
	        ▼
	Throughput
	        │
	        ▼
	Hot Partitions
	        │
	        ▼
	Consumer Groups
	```

### Common Interview Questions
1. How do you choose a Kafka partition key?
> 	I choose a key that groups together events requiring ordered processing while still distributing load evenly across partitions. The ideal partition key preserves the ordering needed by the business without creating hot partitions that reduce throughput.
2. Your application processes events for individual bank accounts, and **the order of transactions for each account is critical**. How would you choose a partition key? What tradeoffs would you consider?
> 	I would partition by account ID to guarantee all transactions for a given account land in the same partition in a specific order. While this would guarantee ordering, it would introduce the potential of creating a hot partition if one account became much busier than other accounts. In practice, I'd also monitor partition distribution to verify that account activity is reasonably balanced. If a small number of accounts generated disproportionate traffic, I might need to revisit the partitioning strategy.
3. A teammate says: "Let's use a single partition so we get perfect ordering for all events." Would you agree? What are the benefits, and what are the downsides of that approach?
> 	While a signle partition guarantees ordering for all events, it decreases throughput and limits the system's ability to scale horizontally. Choosing a partition key requires carefully striking a balance between the need for strict message ordering and the need for high throughput and scalability.
4. Suppose one bank account generates 90% of all transactions. Even though you're partitioning by AccountID, one partition becomes overwhelmed. What would you do?
> 	First, I'd confirm that the hot partition is actually the bottleneck. If strict per-account ordering is a hard business requirement, I may have to accept that one partition becomes the limiting factor for that account. If the business can tolerate relaxing ordering for some operations, I could explore repartitioning strategies or redesigning how those events are processed. The right solution depends on whether ordering or throughput is the higher priority.
	- The answer isn't to immediately explore technical solutions, **it's to identify whether the issue actually needs to be addressed**.
	- Next, it's about **examining business requirements** and proposing a solution that best addresses the issue.

## 6. Replication and Fault Tolerance

- When a producer writes a message to Kafka, it's saved in a **broker**. This broker decides how to partition messages based on partition key.
	- Suppose there's only one broker. If that broker crashes, the message is lost.
	- Kafka solves this by storing multiple copies of a partition in different brokers:
		```
		Broker 1
		Partition 0 (Leader)
		
		Broker 2
		Partition 0 (Follower)
		
		Broker 3
		Partition 0 (Follower)
		```
		- Every partition has one **leader** and zero or more **followers**.
		- Only the leader:
			- Accepts writes
			- Serves consumer reads (in most configurations)
			- **Coordinates replication**
		- Followers:
			- Continuously copy data from the leader
			- Don't accept producer writes
			- Stay synchronized
- When a leader crashes, Kafka elects one of the followers as the new leader. **It does not lose the partition**.
- Suppose a leader crashes before followers can replicate a message. Whether the message is lost depends primarily on **acknowledgements**:
	- `acks = 1`: The producer only waits for the leader before declaring success. This creates the potential for messages to be lost if replication doesn't occur.
	- `acks = all`: The producer waits for the leader and all **required** followers before declaring success. This is much safer, but slightly slower.
- **In-Sync Replicas**:
	- Not every follower is always caught up. Some followers can fall behind while replicating messages.
	- Kafka keeps track of which replicas are sufficiently caught up. This is called the **ISR set**. Only replicas in the ISR set can become leaders. This is because promoting an outdated replica creates potential for losing committed data.
- The number of brokers that messages are replicated to should be carefully chosen because replication has hidden costs:
	- More network traffic
	- More disk usage
	- Higher write latency
	- More coordination
- The number of brokers a message is replicated to is sometimes referred to as the **replication factor**:
	- Development: Replication factor = 1. No redundancy.
	- Production: Replication factor = 3. Good balance between cost and durability.
	- Critical Systems: Replication factor = 5. Higher durability. Higher cost.

### Mental Model
- Connection to Previous Topics:
	```
	Replication
	      │
	      ▼
	Leader
	      │
	      ▼
	Followers
	      │
	      ▼
	Acknowledgments
	      │
	      ▼
	Fault Tolerance
	      │
	      ▼
	Exactly-once
	```

### Common Interview Questions
1. Why does Kafka use leader/follower replication?
> 	Replication allows Kafka to tolerate broker failures without losing data. Each partition has one leader that handles reads and writes and one or more follower replicas that continuously copy the leader's data. If the leader fails, Kafka can elect an in-sync follower as the new leader and continue processing with minimal interruption.
2. Your Kafka cluster has a **replication factor of 3**. A producer uses: `acks = 1`. The leader acknowledges the write, but crashes before either follower has replicated the message. What happens? Would that message necessarily survive?
> 	If the leader crashes before replication occurs, the message is lost because the producer only waited for the leader to acknowledge the write. It didn't wait for required followers to also acknowledge the write. If that level of durability is unacceptable, I'd use `acks=all` so the producer waits for the required in-sync replicas before considering the write successful.
	- In some systems, acks can be set at 1. Messages are retained on the producer side and recent if they are not processed in a certain time window. However, setting `acks = all` is the simpler approach.
3. A teammate says: "Let's increase the replication factor from 3 to 10 so we never lose data." Would you agree? What are the benefits, and what tradeoffs would you explain?
> 	Increasing replication factor from 3 to 10 would significantly increase network traffic, disk usage, and latency. While this could make the system more durable, it should only be done if there are strict business requirements that can't tolerate lost messages. Beyond the storage and network overhead, a replication factor of 10 also requires a much larger cluster and increases operational complexity. For most production systems, a replication factor of 3 provides a good balance between durability, availability, and cost.
4. If replication factor is 3, does that mean Kafka can always survive two broker failures?
> Not necessarily. It depends on **which brokers fail** and whether the remaining replica is part of the **In-Sync Replica (ISR)** set.

## 7. Message Retention

- With the traditional queue model, a producer publishes a message to a queue, then immediately deletes it once it's consumed (not processed) by a consumer.
	- This can lead to data loss if a message is lost after it's consumed, but before it's processed.
	- It also mitigates the ability of other consumers to read the message at a later point.
- Kafka differentiates itself by retaining messages after they're processed. Kafka doesn't delete messages as soon as a consumer commits an offset.
	- This is useful for cases where messages are processed incorrectly and need to be reprocessed once a bug is fixed.
	- In this case, the consumer's offset would be reset and the messages would be replayed. Problem solved. This is one of Kafka's greatest strengths.
- **Retention Policies**:
	- Kafka doesn't keep messages forever by default. It uses configurable retention policies.
	- The two most common types of policies are:
		- Time-based retention
		- Size-based retention
	- Kafka can use both policies together. Whichever limit is hit first triggers message deletion.
- **Replaying Data**:
	- Data can be replayed in Kafka by resetting a consumer's offset. This tells Kafka to resend old messages to that consumer so it can reprocess them.
	- No special backups required. As long as the message is still retained, it can be replayed.

### Log Compaction
- Imagine you maintain a Kafka topic that stores consumer profiles. If a user regularly updates their profile, the event log constantly changes.
- Instead of storing every version of an event, **log compaction** allows Kafka to only store the most recent version.
- Log compaction is ideal for topics that primarily represent the **current state**, such as:
	- Customer profiles
	- Product catalog
	- Account settings
	- Inventory counts
	- User preferences
- Log compaction should be avoided in scenarios where you're storing event history, such as order progress from creation to delivery.

### Tradeoffs
- Longer Retention:
	- Pros:
		- Easier replay
		- Better recovery
		- Historical analysis
	- Cons:
		- More storage
		- Longer recovery times (a larger amount of data needs to be replayed)
		- Higher costs
- Shorter Retention:
	- Pros:
		- Lower storage costs
		- Simpler operations
	- Cons:
		- Limited replay
		- Less flexibility
		- Harder debugging
- Log Compaction:
	- Pros:
		- Current state stays available
		- Lower storage usage
		- Faster recovery of stateful applications
	- Cons:
		- Historical changes are removed
		- Not suitable for event history

### Mental Model
- Connection to Previous Topics:
	```
	Producer
	     │
	     ▼
	Kafka Log
	     │
	     ▼
	Retention
	     │
	     ▼
	Offsets
	     │
	     ▼
	Replay
	     │
	     ▼
	Fault Recovery
	```

### Common Interview Questions
1. Why doesn't Kafka delete messages immediately after consumers process them?
> 	Kafka separates message consumption from message retention. Consumers track their own offsets, allowing messages to remain available for replay, recovery, and multiple independent consumer groups. This makes Kafka well suited for event streaming and analytics workloads.
2. Your team discovers that a bug caused your analytics application to incorrectly process the last three days of Kafka events. How could Kafka's retention model help you recover? What conditions would need to be true for this to work?
> 	Kafka's retention model can aid recovery by allowing consumer offsets to be reset. This would allow Kafka to resend messages to those consumers so they can be reprocessed with the updated logic. In order for this to work, the retention policy must be set so that the last three days of events are retained. This would mean setting a time-based retention policy greater than 3 days or a size-based retention policy that accommodates more than 3 days worth of messages. The replayed processing should also be idempotent or directed to a new output location if rerunning the events would otherwise create duplicate effects."
	- Make sure to explicitly mention an idempotent replay process. this is very important.
3. A teammate says: "Let's enable log compaction on our financial transactions topic so we save storage." Would you agree? Why or why not? What kinds of data are good candidates for log compaction, and what kinds are not?
> 	I would only agree if the business requirements did not require any type of historical analysis. If this was not a requirement, the benefit of storage savings would outweigh the lack of historical state data.
	- Log compaction is about removing historical **state** data, not historical **event** data. Each event is its own separate entity, not related to previous events. State, such as the state of a customer's profile, is related to previous state data, such as when a profile is updated.
4. Can a Kafka topic have both retention and log compaction?
	- Yes, Kafka can support delete retention and log compaction at the same time.
	- You might compact a topic to preserve the latest value for each key while also deleting very old records after a retention period.

## 8. Exactly-once in Kafka

- Deciding when a consumer should commit an offset can be tricky:
	- Commit fist: Risk losing the message before work is complete.
	- Commit last: Risk performing duplicate work.
- The fundamental issue is that Kafka, the consumer, and downstream systems, such as a database, are **separate entities**.
	- Each entity has its own state.
	- Kafka tracks offsets.
	- The database tracks rows.
	- If one system succeeds and the other fails, the systems become **inconsistent**.
	- Exactly-once processing is about keeping multiple systems consistent.
- **Kafka's Solution**:
	- Kafka introduces **transactions**.
	- Kafka allows certain operations to be treated as **one atomic unit**. This means that either everything succeeds or everything fails.
	- For example, a **producer transaction** would be something like a write transaction. Either a message is successfully written to all desired topics or the transaction is considered failed.
	- An example of a **consumer transaction** wold be consumers coordinating to produce new messages or commit offsets.
	- Suppose:
		```
		Read Topic A
		
		↓
		
		Transform
		
		↓
		
		Write Topic B
		
		↓
		
		Commit Offset
		```
		- Transactions allow Kafka to force the write and commit to succeed together. If the transaction aborts, nothing happens.
		- Transactions are "all or nothing events."
- **Idempotent Producers**:
	- With idempotent producers, Kafka assigns each producer a producer ID and sequence numbers.
	- When a producer retries sending a message because it never received the acknowledgement, Kafka can recognize and ignore any duplicate records.
- Exactly-once processing ensures that the **final observable effect** happens once, not that message is only processed once.
	- If a message is processed twice, any duplicate effect is ignored and the original result is resent.
- **End-to-End Exactly Once**:
	```
	Producer
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Kafka
	
	↓
	
	Consumer
	```
	- Exactly-once requires cooperation among all entities.
	- Producer: Idempotent
	- Kafka: Transactions
	- Consumer: Correct offset management
- **Limitations**:
	- Kafka cannot guarantee exactly-once processing everywhere.
	- Kafka can only provide exactly-once semantics between Kafka clients and Kafka topics. Kafka **cannot** automatically make PostgreSQL, for example, participate in Kafka transactions.
		- This would usually be achieved through:
			- Database transactions
			- Idempotent writes
			- Transactional outbox patterns.

### Tradeoffs
- Advantages:
	- No duplicate **observable** effects.
	- Simpler downstream reasoning.
	- Strong correctness guarantees.
- Disadvantages:
	- More coordination.
	- Higher latency.
	- Additional complexity.
	- Transaction management overhead.
- For many data pipelines, at-least-once plus idempotency **is still the preferred architecture**.

### Mental Model
- Connection to Previous Topics:
	```
	Retries
	     │
	     ▼
	Duplicates
	     │
	     ▼
	Idempotent Producers
	     │
	     ▼
	Transactions
	     │
	     ▼
	Offset Commits
	     │
	     ▼
	Exactly-Once Semantics
	```

### Common Interview Questions
1. How does Kafka achieve exactly-once semantics?
> 	Kafka combines idempotent producers with transactions to ensure retries don't create duplicate records and that related operations, such as writing output messages and committing consumer offsets, succeed atomically. For end-to-end exactly-once processing, downstream systems must also participate through transactional or idempotent writes because Kafka alone cannot guarantee correctness across external systems.
2. Your Kafka application reads events from **Topic A**, transforms them, writes the results to **Topic B**, and then commits the consumer offsets. Why can this pipeline still produce duplicate output if transactions are **not** used? How do Kafka transactions solve this problem?
> 	This pipeline can still produce duplicate transactions if a consumer crashes before committing the offset. Kafka transactions allow the write to Topic B and the consumer offset commit to be committed as a single atomic operation. If the transaction aborts, neither the output message nor the offset commit becomes visible. If it succeeds, both do. This prevents duplicate output caused by retries after failures.
	- Don't just mention that Kafka transactions are atomic. Explain what that means.
3. A teammate says: "Kafka supports exactly-once processing, so we no longer need idempotent database writes."
> 	I would clarify that Kafka can only guarantee exactly-once processing between Kafka clients and Kafka topics. external systems must still be made idempotent to guarantee correctness. Even with exactly-once processing, the database still needs idempotent or transactional writes to prevent duplicate effects.
4. If I have Kafka transactions, do I still need idempotent consumers?
> 	Usually, yes. Kafka transactions protect operations that occur within Kafka, such as producing records and committing offsets atomically. If a consumer updates an external system like PostgreSQL, Redis, or a REST API, Kafka can't guarantee exactly-once behavior for that external operation. The consumer should still use idempotent or transactional writes when interacting with systems outside Kafka.

## 9. Kafka System Design & Troubleshooting Scenarios

- Imagine you get paged at 2:00 AM and the alert says: "Kafka consumer lag is increasing".
	- A junior engineer might immediately jump to "add more consumers."
	- An experienced engineer thinks: "Why is lag increasing?"
	- Adding consumers **only treats the symptom**, not necessarily the **root cause**.

### Scenario 1: Consumer Lag
- Imagine this pipeline:
	```
	Application
		  │
		  ▼
	Kafka
		  │
		  ▼
	Spark Structured Streaming
		  │
		  ▼
	Snowflake
	```
	- One day, the pipeline is heathy. The producer is operating at 50k events/sec and the consumer is operating at 50k events/sec.
	- The next day, the consumer decreases to 30k events/sec. Consumer lag has started to increase and continues increasing.
	- Consumer lag tells you that messages are arriving more quickly than a consumer can process them. **It doesn't tell you why**. It's just a **symptom**, not the **cause**.
	- Investigating the root cause is a multi-step process.
- **Step 1: Is The Procedure Normal?**
	- Has the producer changed? Is it producing more events than usual?
		- If so, that's not a consumer problem. That's a traffic spike.
	- **Metrics**:
			- Producer throughput
			- Messages/sec
			- Topic write rate
- **Step 2: Is Kafka Healthy?**
	- **Metrics**:
		- Broker CPU
		- Broker memory
		- Network utilization
		- Disk I/O
		- Under-replicated partitions
		- Offline partitions
	- If brokers are overloaded, consumers may not be receiving data efficiently.
- **Step 3: Are Consumers Busy?**
	- **Metrics**:
		- CPU Utilization
		- Memory Utilization
	- **Questions**:
		- Is there frequent garbage collection?
		- Are consumers idle?
		- Are all partitions assigned?
	- If CPU Utilization is only 15%, adding more consumers won't solve the issue.
	- If CPU Utilization is 98%, now compute might actually be a bottleneck. Scaling out **might** help.
- **Step 4: Spark**
	- If Spark is the consumer, look for the following:
		- Data skew
		- Excessive shuffling
		- Expensive joins
		- Missing broadcast joins
		- Repeated recomputation
		- Slow UDFs
- **Step 5: Downstream Systems**
	- Suppose Spark writes to Snowflake. If Snowflake becomes slow:
		- Spark needs to wait.
		- Kafka consumer lag grows.
	- Kafka isn't the bottleneck. Snowflake is.
	- **This is why end-to-end thinking matters**.
- **Root Cause Summary**:
	- Growing consumer lag is generally caused by:
		- Traffic spike
		- Insufficient compute
		- Data skew
		- Slow transformations
		- Slow downstream database
		- Kafka broker issues
		- Network bottlenecks

### Scenario 2: Hot Partition
- A topic has 24 partitions. Reviewing metrics, you see that 90% of traffic is being directed to Partition 17. Every other partition is practically idle.
	- This is typically caused by a poor partition key.
- **Confirming The Diagnosis**:
	- Look at metrics such as:
		- Messages per partition
		- Bytes per partition
		- Consumer utilization
		- Partition throughput
	- These will usually point out which partition is the hot partition.
- **Fixes**:
	- Choose a better partition key to distribute load more evenly.

### Scenario 3: Rebalance Storms
- Symptoms:
	```
	Consumers stop
	
	Resume
	
	Stop
	
	Resume
	
	Stop
	
	Resume
	```
	- Consumer lag spikes repeatedly.
- **Common Causes**:
	- Consumers crashing
	- Consumers restarting
	- New consumers joining
	- Consumers leaving
	- Long GC pauses
	- Heartbeats timing out
- **Why Are Frequent Rebalances Bad?**
	- During a rebalance:
		- Partition ownership changes
		- Consumers pause briefly
		- Throughput drops
		- Consumer lag increases
	- Occasional rebalances are normal and expected.
	- Frequent or continuous rebalancing indicates a problem.
- **Investigation**:
	- Look for:
		- Consumer logs
		- Restarts
		- OOM errors
		- Timeout exceptions
		- Deployment history
	- Maybe someone accidentally configured Kubernetes to restart pods every few minutes. In this case, Kafka isn't broken, the deployment is.

### Scenario 4: Choosing a Good Partition Key
- When a manager asks: "How many partitions should this topic have?" Think about factors such as:
	- Current throughput
	- **Future growth**
	- **Maximum desired parallelism**
	- **Ordering requirements**
	- Broker count
	- Operational complexity
- Suppose that a topic only has 4 consumers today, but grows to 20 consumers in the future. If the topic is only split up into 4 partitions, most consumers will sit idle and you will need to repartition.
- You could have originally split the topic into 20 partitions. However, you need to strike a careful balance between partition size and expected future growth. Too many partitions can be just as bad as too few partitions.
- **Planning ahead matters**.

### Scenario 5: Scaling Kafka
- Suppose throughput is low. Knowing what to add depends on the bottleneck.
- **Addidng Consumers**:
	- Helps when consumers are saturated.
	- Doesn't help when you have more consumers than partitions.
- **Adding Partitions**:
	- Helps when parallelism is limited.
	- Won't help when a single hot partition is dominating.
- **Adding Brokers**:
	- Helps when cluster resources are exhausted.
	- Won't help if consumers are slow.
- **Optimizing Processing**:
	- This is often the best answer.
	- Optimizations typically include:
		- Broadcast joins
		- Filtering early
		- Better partition key
		- Caching
		- Avoid shuffles

### End-to-End Thinking
- One of the biggest mistakes you can make is assuming Kafka is always the bottleneck.
- Imagine the following pipeline:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Snowflake
	```
	- If the dashboard is stale, where is the issue? What's causing it?
- It could be:
	- Producer
	- Kafka
	- Spark
	- Snowflake
	- Network
- **Never assume. Always measure.**

### Mental Model
- Think of Kafka like a highway system.
	- **Adding lanes (partitions)** helps if traffic can spread out.
	- **Adding more drivers (consumers)** helps if there are enough lanes to use.
	- **Building more highways (brokers)** helps if the infrastructure itself is overloaded.
	- But if **one lane is blocked by an accident (hot partition or slow consumer)**, adding more drivers won't solve the traffic jam.
- The key is identifying **where** the bottleneck is before deciding **how** to fix it.

### Common Interview Questions
1. Consumer lag is increasing. What do you do?
> 	First I'd determine whether the issue is increased production or reduced consumption. I'd compare producer and consumer throughput, examine Kafka broker health, and then investigate the consumer application for bottlenecks such as data skew, expensive transformations, or slow downstream systems. Only after identifying the bottleneck would I decide whether to optimize processing, scale consumers, add partitions, or address infrastructure issues.
	- Notice the structure:
		- Observe
		- Hypothesize
		- Measure
		- Diagnose
		- Fix
2. Your monitoring shows:
	- Producer throughput is unchanged.
	- Kafka brokers have low CPU and normal disk usage.
	- Spark executors are mostly idle **except one**, which is running at nearly 100% CPU and taking much longer than the others.
	- What is your leading hypothesis? How would you confirm it? What are some possible solutions?
> 	My leading hypothesis is uneven work distribution. That could originate from a Kafka hot partition or from Spark data skew during processing. I'd use Kafka metrics to check message distribution across partitions and the Spark UI to examine task durations and shuffle metrics to determine where the imbalance is introduced. Possible solutions would include choosing a better partition key or salting the existing partition key.
	- Other solutions include:
		- Repartitioning after reading from Kafka (if the skew is introduced later)
		- Filtering earlier
		- Pre-aggregation
3. Your team wants to double the number of Kafka consumers because consumer lag has increased. Before approving the change, what information would you want to gather? In what situations would doubling consumers help, and when would it have little or no effect?
> 	Before doubling the number of consumers, I would want to know whether consumer throughput was the bottleneck or if there is a problematic downstream system. Adding consumers would help in situations where there are a sufficient number of partitions to rebalance the workload, but wouldn't help in scenarios where there is a hot partition.
4. Suppose you confirm the problem is a hot partition caused by one extremely active customer. You can't change the partition key because events for that customer must remain ordered. What do you do?
	- If ordering for that customer is a hard business requirement, then that customer's events **must** stay on one partition. That partition becomes the throughput limit for that customer's workload.
	- Possible solutions include:
		- Verify that the hot customer is actually impacting business SLAs.
		- Optimize processing for that partition (reduce expensive transformations, filter early, etc.).
		- Scale vertically if appropriate.
		- Revisit business requirements—can some event types be processed independently, or is strict ordering truly required for every operation?
	- Notice that the solution may involve **negotiating requirements**, not just changing Kafka.