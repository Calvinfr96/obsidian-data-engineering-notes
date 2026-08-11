To prepare for data engineering system design rounds, you must practice mapping business requirements to distributed systems components while justifying your architectural trade-offs.
# Prompt 1: The Real-Time Ad Analytics Dashboard
- **The Scenario:** Design an end-to-end data platform that ingests ad impression and click events from global web clients, tracks billing metrics, and populates an internal real-time analytics dashboard for advertisers.
- **Scale Requirements:** 100,000 events per second average, peaking at 500,000 events per second.
- **Core Technical Challenges:**
    - **Idempotency & Exact-Once:** Advertisers cannot be double-billed for duplicate click events caused by network retries.
    - **Late-Arriving Data:** Mobile clients occasionally cache clicks offline and send them hours late. How do you handle windowed aggregations when event time deviates significantly from processing time?

## Overview

- **Key Requirements**:
	- 500K events/sec peak
	- Billing must not happen twice
	- Late events must still be attributed to the correct event-time window
- These requirements necessitate **two related pipelines**:
	```
	                    Ad Events
	                       │
	                       ▼
	              ┌─────────────────┐
	              │ Streaming Layer │
	              └────────┬────────┘
	                       │
	                       ▼
	              ┌─────────────────┐
	              │ Stream Processor│
	              │     (Flink)     │
	              └───────┬─────────┘
	                      │
	             ┌────────┴─────────┐
	             │                  │
	             ▼                  ▼
	       Billing Path        Analytics Path
	             │                  │
	             ▼                  ▼
	      Durable Ledger      Real-Time Store
	             │                  │
	             │                  ▼
	             │             Dashboard
	             │
	             ▼
	          Finance
	```
	- The **billing path and analytics path have different correctness requirements**.
- **Functional Requirements**:
	- Ingest impression events
	- Ingest click events
	- Calculate advertising metrics
	- Calculate billing metrics
	- Provide a real-time dashboard
	- Support global clients
- **Scale**:
	- Average: 100,000 events/sec
	- Peak: 500,000 events/sec (1.8B events/hour)
- **Correctness**:
	- A duplicate click must **not result in duplicate billing**.
	- A click received hours late must still be attributed to its **original event-time window**.
- **Event Schema (Example)**:
	```json
	{
	  "event_id": "abc123",
	  "event_type": "click",
	  "event_time": "2026-08-10T14:00:03Z",
	  "received_time": "2026-08-10T14:00:05Z",
	  "advertiser_id": "adv42",
	  "campaign_id": "campaign7",
	  "user_id": "user123",
	  "impression_id": "imp456",
	  "cost": 0.25
	}
	```
	- The most important field here is `event_id`.
	- That gives us a way to identify duplicate deliveries.
	- Another important fields are `event_time` and `received_time`. These tell us how to manage late-arriving click events.

## Ingestion Layer

- At 500K events/sec peak, we need a **durable distributed streaming layer** between clients and processors.
- An AWS-oriented architecture could be:
	```
	Global Clients
	     │
	     ▼
	API Gateway / Load Balancer
	     │
	     ▼
	Kinesis Data Streams
	     │
	     ├───────────────┐
	     ▼               ▼
	Flink             Raw Storage
	                  S3
	```
	- Kafka would be a perfectly reasonable alternative to Kinesis.
- The most important architectural property is not having clients **synchronously** call the billing or analytics system directly.
- Streaming gives us:
	- Buffering
	- Horizontal scalability
	- Durability
	- Replay
	- Multiple consumers
	- Decoupling between ingestion and processing
- At 500K events/sec, the stream should be partitioned based on an appropriate key such as `advertiser_id`, `campaign_id`, or another key that provides good distribution while preserving any ordering guarantees we need.

## Processing Layer

- **Exactly-Once Processing**:
	- Don't rely on "exactly-once" guarantees from the streaming system alone. **Exactly-once** only guarantees the same event won't be **delivered** twice, not that it won't be **processed** twice by the consumer.
	- Using `event_id` as an idempotency key can ensure there are no duplicate effects from an event being processed twice.
	- A DynamoDB-based implementation could look like:
		```
		event_id       status       charge
		--------------------------------------
		ABC123         PROCESSED    $0.25
		```
	- When a record **first** arrives:
		```
		Conditional Put ABC123
		       ↓
		Success
		       ↓
		Create billing record
		```
		- It is stored in DynamoDB as shown above, if processed successfully.
	- When a **duplicate** record arrives:
		```
		Conditional Put ABC123
		       ↓
		Conditional check fails
		       ↓
		Duplicate
		       ↓
		Do NOT bill
		```
		- The conditional write is important because two copies could arrive concurrently.
		- To avoid concurrent writes from causing double billing, you need an **atomic conditional operation**.
- **Handling Billing Failures**:
	- What happens if DynamoDB marks an event as `PROCESSED`, but the billing operation fails?
	- You don't want:
		- Idempotency Record = `PROCESSED`
		- Billing = `FAILED`
	- The billing record itself should be uniquely constrained by `event_id`.
	- Conceptually:
		```
		Billing Ledger
		
		event_id    advertiser    amount    status
		------------------------------------------------
		ABC123      ADV42         $0.25     BILLED
		```
		- Instead of prohibiting duplicate `PROCESSED` events, prohibit duplicated `BILLED` events in the billing ledger.
	- Summary:
		```
		At-least-once delivery
		        +
		Idempotent consumer
		        +
		Atomic billing record
		        =
		Exactly-once business effect
		```
	- In an interview, avoid promising "exactly-once" in a distributed system environment. Instead, say:
> 	I'd use at-least-once delivery with idempotent processing. Every event has a globally unique event ID, and the billing ledger enforces uniqueness on that ID using an atomic conditional write or transactional operation. This means retries can safely occur without creating a second billing effect.
- **Handling 500K/sec Peak Traffic**:
	- Example Architecture:
		```
		500K events/sec
		        ↓
		Partitioned stream
		        ↓
		Multiple Flink parallelism units
		        ↓
		Parallel processing
		```
	- Important KPIs:
		- Stream throughput
		- Consumer lag
		- Flink backpressure
		- Checkpoint duration
		- Processing latency
		- State size
		- Dashboard-store latency
	- It would also be important to load test the architecture **above** the 500k/sec peak to ensure it performs well.

### Handling Late-Arriving Data
- Suppose we're calculating 'Clicks per advertiser per 5-minute window':
	- A click happens at 10:02.
	- The mobile device is offline. It sends the click event at 13:00.
- If `received_time` is used instead of `event_time`, the click would be incorrectly placed in the `13:00 - 13:05` window.
- The event really belongs in the `10:00 - 10:05` window.
- The `event_time` field needs to be used to establish analytical windows, not `processed_time`.
- **Benefits of Using Flink**:
	- Flink is a **stateful** event processor. It's ability to maintain state over a stream of events allows it to identify a late arriving event and place it in the correct analytical window.
	- The stream processor can operate using:
		- Event time
		- Windows
		- Watermarks
		- Allowed lateness
		- State
- **Watermarks**:
	- A watermark can help the streaming layer decide when it's appropriate to close an analytical window to any late-arriving events.
	- Conceptually:
		```
		Event time
		──────────────────────────────>
		
		10:00    10:05    10:10
		                   ↑
		                watermark
		```
		- The watermark represents the system's estimate of how far event time has progressed.
	- Setting an `Allowed lateness` watermark of 5 minutes could potentially invalidate events arriving hours late, which a design requirement. This conflict can be resolved in a variety of ways:
		- Option 1:
			- One approach to solving this conflict is to keep windows open for hours by setting the watermark appropriately.
			- However, doing so would dramatically increase the amount of state that must be retained.
			- **This becomes particularly expensive at 500K events/sec**.
		- Option 2:
			- Another approach allowing real-time results to be continuously corrected by late-arriving events.
			- Real-Time Result:
				```
				10:00–10:05
					 ↓
				Initial count = 1,000
				```
			- Later on:
				```
				Late event arrives
					 ↓
				event_time = 10:03
					 ↓
				Correct window
					 ↓
				New count = 1,001
				```
			- The dashboard can be updated as events arrive.
			- To implement this, the analytics store would need to support **upserts**, rather than append-only writes.
- **Separate Billing and Analytics**:
	- Don't make billing depend on the eventual completion of an analytics window.
	- A late-arriving click event should be processed immediately and billed once.
	- For analytics, the late-arriving click should be attributed to the correct aggregation window and the aggregation should be updated accordingly.
	- These are two separate events that should not depend on one another.

### Handling Global Clients
- Because clients are global, Events shouldn't necessarily be sent across the world to one region.
- A more scalable architecture would look like:
	```
	North America ──> US ingestion
	Europe ─────────> EU ingestion
	Asia ───────────> APAC ingestion
	```
- Then aggregate globally downstream. That reduces network latency and provides regional fault isolation.
- You'd need to explicitly clarify whether billing must have a **single global source of truth** or whether billing can be partitioned by region.

## Storage Layer

- Raw events should be stored in S3:
	```
	Kinesis
	   ├──> Flink
	   │      ↓
	   │   Dashboard
	   │
	   └──> S3
	```
- S3 provides durable historical storage, which is useful for:
	- Auditing
	- Billing reconciliation
	- Replay
	- Historical analytics
	- Debugging
	- Model building
	- Correcting historical aggregations
- Replay is especially important for advertising events. Corrections need to be made if a bug is found in any aggregation logic.
- For example, suppose the CTR calculation is incorrect. Because raw events have been retained in S3:
	```
	S3
	 ↓
	Reprocess
	 ↓
	Corrected aggregation
	 ↓
	Analytics store
	```
	
## Analytics Layer

- The dashboard should **not** query the raw S3 data. Instead the dashboard should be fed by a real-time analytics store:
	```
	Flink
	 ↓
	Aggregated metrics
	 ↓
	Real-time analytical store
	 ↓
	Dashboard
	```
	- Depending on the constraints, candidates include:
		- Amazon OpenSearch
		- ClickHouse
		- Apache Pinot
		- Druid
		- Specialized time-series/OLAP store

## Proposed Architecture

- Putting it all together:
	```
						 GLOBAL CLIENTS
				 ┌──────────┼──────────┐
				 ▼          ▼          ▼
				US         EU         APAC
				 │          │          │
				 └──────────┼──────────┘
							▼
					Kinesis / Kafka
							│
				 ┌──────────┴──────────┐
				 │                     │
				 ▼                     ▼
			   Flink                  S3
				 │                  Raw Events
		   ┌─────┴──────┐
		   │            │
		   ▼            ▼
	   Billing       Analytics
		Path           Path
		   │            │
		   ▼            ▼
	 Idempotency    Event-Time
	  + Ledger      Aggregations
		   │            │
		   │            ▼
		   │       Real-Time OLAP
		   │            │
		   │            ▼
		   │        Dashboard
		   │
		   ▼
		Finance
	```
- Inside Flink:
	```
	Event
	  │
	  ├── validate
	  ├── deduplicate
	  ├── enrich
	  │
	  ├───────────────> Billing
	  │
	  └── event-time window
	           │
	           ├── watermark
	           ├── allowed lateness
	           └── late-event correction
	```

### Error Handling
1. Ingestion Error:
	- At the streaming layer, I'd rely on the stream's **durability and retry/replay capabilities**.
	- For example, if Flink temporarily goes down, events aren't immediately lost. Once Flink recovers, it can continue consuming them.
	- This is one reason I would **not acknowledge/remove an event from the ingestion system until the downstream consumer has successfully processed it**.
2. Malformed / Invalid Events:
	- Flink would perform event validation early:
		```
		Event
		 ↓
		Validate
		 ├── Valid ──→ normal processing
		 │
		 └── Invalid → Dead-letter / quarantine
		```
	- The invalid event should be retained somewhere such as an S3 **dead-letter/quarantine area**, along with enough metadata to investigate why it failed.
	- One malformed event shouldn't repeatedly fail and block processing of the rest of the stream.
3. Transient Processing Failures:
	- Suppose Flink temporarily can't access DynamoDB or the analytics store.
	- That's a situation where **retrying makes sense**.
	- The following strategies could be used:
		- Retries
		- Exponential backoff
		- Bounded retry attempts
	- If the dependency remains unavailable, the event should remain recoverable rather than simply disappearing.
4. Billing Errors:
	- If the billing service temporarily becomes unavailable, **the click event should not be discarded**.
	- Because billing is a financial operation, event/billing requests should be retained and retried.
	- If a billing request succeeds, but the response is lost due to a network failure, Flink would retry the event.
	- Due to the idempotent nature of billing processing, the advertiser is not billed twice for the same event.
5. Analytics Errors:
	- If dashboard aggregation fails, that shouldn't affect billing. The analytics path can retry/reprocess independently.
	- This is another reason **separate consumers/processing paths** should be maintained rather than making the entire pipeline one giant workflow.
6. Dead-Letter Handling:
	- A dead-letter/quarantine path would be maintained for events that **cannot be successfully processed after retries**.
	- Then operations can investigate those events and potentially replay them after fixing the underlying problem.
	- **DLQ and raw event storage should remain separate in S3**. The raw S3 copy is the **complete historical event stream**. The DLQ is specifically for events that **failed processing** and need operational attention.

### Monitoring and Alerting
- Important errors / metrics to monitor:
	- Ingestion errors
	- Flink processing errors
	- Consumer lag
	- Retry counts
	- DLQ volume
	- Billing failures
	- Duplicate-event rates
	- Checkpoint failures
	- Dashboard-store errors
	- End-to-end latency

### Interview Questions
1. How do you guarantee exactly-once billing?
> 	I wouldn't rely solely on exactly-once stream processing. I'd use at-least-once delivery combined with an idempotency key derived from the event ID. The billing ledger would enforce uniqueness on that event ID through an atomic conditional or transactional write. That way, retries and duplicate deliveries can't create a second billing effect.
2. How do you handle clicks that arrive hours late?
> 	I'd use event-time rather than processing-time windows. Flink would use watermarks to determine when windows are likely complete and an allowed-lateness policy for late events. Because events can arrive hours late, I wouldn't necessarily keep every window open for hours; instead, I'd emit provisional aggregates and support corrections or upserts when late events arrive. The analytics serving layer would therefore need to support updating previously published aggregates.
3. When you say exact-once, do you mean exactly-once processing, or exactly-once billing effects?
	- The second is the actual business requirement.
	- Events being processed twice internally can be tolerated **as long as the billing effect occurs only once**.
4. How accurate does the dashboard need to be at any given moment, and how long are we willing to revise historical windows?
> 	If the dashboard needs continuously correct results, I'd use corrections/upserts. If it only needs approximate real-time metrics with a final reconciliation later, we could optimize the architecture differently.
5. Where does error handling occur?
> 	I'd handle errors at each stage rather than having a single error-handling component. The streaming layer provides durability and replay if consumers fail. Flink validates events and routes malformed events to a dead-letter or quarantine path rather than retrying permanently invalid data. Transient dependency failures use bounded retries with exponential backoff. For billing failures, events remain recoverable and the billing operation is idempotent using the event ID, so retries can't create duplicate charges. Analytics failures are isolated from the billing path and can be retried independently. Finally, I'd monitor consumer lag, processing errors, retry counts, DLQ volume, and billing failures and alert when they exceed defined thresholds.

| Challenge                        | Design response                                                  |
| -------------------------------- | ---------------------------------------------------------------- |
| **500K events/sec**              | Partitioned durable stream + horizontally scalable Flink         |
| **Duplicate clicks**             | Unique event ID + idempotent billing ledger                      |
| **Exactly-once business effect** | Atomic/transactional deduplication + billing                     |
| **Hours-late events**            | Event-time windows + watermarks + allowed lateness + corrections |
- Everything else supports these four requirements:
	- Transport =  Kafka / Kinesis
	- Processing / State Management / Windowing = Flink
	- Idempotency / Billing State = DynamoDB or another transactional store
	- Dashboard = Real-time OLAP store.
	- Durable Raw History / Replay = S3

# Prompt 2: The E-Commerce Personalization Pipeline
- **The Scenario:** Design a system that captures user clickstream activity on a retail site and updates a "frequently viewed items" feature store, while simultaneously archiving all historical logs for long-term data science model training.
- **Scale Requirements:** 50 Terabytes of raw data generated per day.
- **Core Technical Challenges:**
    - **Storage Tiering & Layout:** How do you partition the storage layer (e.g., in AWS S3 using Apache Iceberg) to ensure data scientists can efficiently query 6 months of history without triggering massive data shuffling or full-table scans?
    - **Kappa vs. Lambda Architecture:** Would you deploy a unified stream engine or split the track into separate batch and speed layers? Defend your choice based on engineering maintenance costs.

## Overview

- The two requirements that should drive the design are:
	- 50 TB/day of raw clickstream data
	- Real-time "frequently viewed items" features + long-term historical data science workloads
- These requirements naturally lead to the following general architecture:
	```
					Clickstream Events
						   │
						   ▼
					Streaming Layer
						   │
					┌──────┴──────┐
					│             │
					▼             ▼
			 Feature Pipeline     S3/Iceberg
					│             │
					▼             │
			  Feature Store       │
					│             │
					▼             ▼
			 Personalization   Data Science
	```
	- The important part is that **the real-time feature path and the historical analytics path have different requirements**, even though they can originate from the same event stream.
- **Real-Time Requirement**:
	- The following clickstream events need to be captured:
		```
		User 123
		  ↓
		Viewed Product A
		Viewed Product B
		Viewed Product A
		Viewed Product C
		```
	- These events need to be used to update a feature store:
		```
		user_123:
		    frequently_viewed = [A, B, C]
		```
	- **The exact latency isn't specified**. "Real-time" needs to be clarified as milliseconds vs. seconds vs. minutes
- **Historical Requirement**:
	- **All clickstream data** needs to be retained for data science. 50 TB/day translates to 1.5 PB/month before compression.
	- At this scale, this is fundamentally a **data lake problem**, not something that can be solved by keeping six months of raw events in a transactional feature store.
- The real-time and historical processing paths should be separated since they have different requirements.
	- **Real-Time Path**:
		```
		Click
		 ↓
		Stream
		 ↓
		Stream processor
		 ↓
		Feature aggregation
		 ↓
		Feature store
		```
		- Optimized for:
			- Low latency
			- Incremental updates
			- Key-based reads/writes
	- **Historical Path**:
		```
		Click
		 ↓
		Stream
		 ↓
		S3 + Iceberg
		 ↓
		Data science
		```
		- Optimized for:
			- Enormous storage
			- Large scans
			- Analytical queries
			- Historical processing
			- Model training
	- This separation is extremely important. A feature store isn't a good place to dump 9 PB of historical clickstream data, and S3 isn't necessarily the ideal low-latency serving layer for a personalization request.

## Ingestion Layer

- Given the massive scale, a durable, distributed event stream should be used for ingestion:
	```
	Web / Mobile Clients
	        ↓
	Kinesis / Kafka
	        ↓
	Stream Processing
	```
	- Kafka and Kinesis are both valid choices.
- The important properties are:
	- High throughput
	- Partitioning
	- Durability
	- Replay
	- **Multiple consumers**
- The streaming layer would feed the feature pipeline and data lake independently:
	```
						Kafka/Kinesis
							 │
				   ┌─────────┴─────────┐
				   ▼                   ▼
			Feature Pipeline       Data Lake
				   │                   │
				   ▼                   ▼
			Feature Store         S3/Iceberg
	```

## Processing Layer

- Kafka / Kinesis act as **message brokers**. The ingest events from producers and distribute those events to multiple consumers. The **stream processor** acts as the consumer and is the component that does something with the events.
- **Apache Flink** would be a great choice for real-time event processing because its state management capabilities.
- For each event:
	```json
	{
	  "user_id": "123",
	  "product_id": "ABC",
	  "event_type": "view",
	  "event_time": "..."
	}
	```
- Flink can maintain state, such as:
	```
	User 123
	 ├── Product A → 50 views
	 ├── Product B → 32 views
	 ├── Product C → 17 views
	 └── Product D → 2 views
	```
- Then periodically update the feature store:
	```
	user_123:
	    top_viewed_products =
	        [A, B, C]
	```
- If the feature store becomes temporarily unavailable during stream processing, the historical stream is still available:
	```
	Kafka/Kinesis
	      ↓
	    S3/Iceberg
	```
	- A temporary outage doesn't mean you've lost the underlying clickstream.
	- When the feature pipeline recovers, it can consume from the appropriate stream position and catch up.

### Duplicate Event Handling
- The prompt doesn't explicitly call out idempotency, but it's nevertheless very important.
- Clickstream clients can retry events, so the event should have a unique `event_id` and the stream processing should have a built-in deduplication strategy.
- Whether you need strict exactly-once semantics depends on how the feature is used. For personalization, an occasional duplicate click may be less catastrophic than a duplicate billing event, but you still don't want duplicates systematically skewing the "frequently viewed" feature.

### Kappa vs. Lambda
- Lambda architecture provides two processing paths:
	```
	                 Events
	                   │
	          ┌────────┴────────┐
	          ▼                 ▼
	     Batch Layer        Speed Layer
	          │                 │
	          ▼                 ▼
	     Historical         Real-time
	     computation       computation
	          │                 │
	          └────────┬────────┘
	                   ▼
	                Results
	```
	- The batch layer periodically recomputes the authoritative results.
	- The speed layer provides low-latency results while the batch computation catches up.
- Kappa architecture provides one streaming path and replays the stream when historical re-computation is required:
	```
	                  Events
	                    │
	                    ▼
	               Kafka/Kinesis
	                    │
	                    ▼
	                  Flink
	                    │
	             ┌──────┴──────┐
	             ▼             ▼
	        Feature Store     S3
	                         Iceberg
	```
	- If processing logic changes, events from S3 are reprocessed, eliminating the need for a separate batch layer.
- Kappa architecture would be superior because the prompt specifically requires defending the decision based on **engineering maintenance costs**. Maintaining a separate batch and streaming layer can be expensive and time consuming, especially when processing logic changes. Special care must be taken to ensure both layers are updated correctly and produce compatible results.
- Furthermore, the fundamental data is an **event stream**, and the same events are already stored in Iceberge for historical analysis. The separate historical analysis almost eliminates the need to replay the stream under Kappa architecture.
- Interview-Ready Response:
> 	I'd lean toward Kappa if the batch and streaming computations are fundamentally the same, because it eliminates maintaining two separate processing implementations. However, if historical processing has fundamentally different computational requirements from the real-time path—for example, very complex joins or large-scale iterative ML workloads—I'd consider a Lambda-style architecture or simply use separate batch analytics alongside the streaming feature pipeline.
- The current architectural proposal doesn't necessarily fall under the "Lambda" label. You don't have a separate batch implementation calculating the **same** real-time feature.
- Instead, you have:
	- Flink handles the online feature computation.
	- S3/Iceberg preserves the historical event stream.
	- Data scientists can run batch analytics independently.
	- Historical feature recomputation can replay the event history through the same processing logic if appropriate.

## Storage Layer

### Feature Store
- The feature store needs a very different access pattern from the historical lake.
- The application might ask: `GET features for user_123` and expect a very fast response.
- A key-value database such as DynamoDB would be a reasonable AWS implementation:
	```
	PK: user_123
	
	frequently_viewed:
	    product_A
	    product_B
	    product_C
	```
	- You could also use a dedicated feature-store technology depending on the organization's ML stack.
	- The important design principle is: The feature store contains the features needed for low-latency inference; S3 contains the historical source data used to generate and retrain those features.

### Historical Data
- Historical clickstream data would be stored at massive scale in S3. The **file format** used would likely be Parquet. Since S3 is being used as a long-term storage layer, the **table format** used would likely be Apache Iceberg. Iceberg is a high-level table format that manages collections of (parquet) files and adds database-like features.
- Conceptually:
	```
	S3
	 └── Iceberg Table
	       ├── Metadata
	       ├── Manifest files
	       └── Parquet data files
	```
- Iceberg gives us table-level metadata and partition management on top of object storage.
- Data should be partitioned based on **how it will be queried**.
- For example, if data scientists typically ask: "Analyze user behavior during January 2026", `event_date` would be a reasonable partition key, rather than something excessively granular, such as `user_id`.
- A query such as: `WHERE event_date BETWEEN '2026-01-01' AND '2026-01-31'` allows the engine to eliminate partitions outside January 2026, before scanning any data.
- At 50 TB/day, a single daily partition could still contain enormous amounts of data. The exact granularity depends on:
	- Query patterns
	- File sizes
	- Number of events
	- Data distribution
	- Query engine
- It's important to remember that **proper partitioning doesn't automatically eliminate small files**. If one partition still contains an excessive number files, even with appropriate partitioning, a **compaction strategy** should be used to appropriately size the Parquet files.
- Conceptually:
	```
	Streaming writes
	     ↓
	Many small files
	     ↓
	Iceberg compaction
	     ↓
	Larger optimized files
	```
	- This is particularly important for a streaming architecture because continuous writes naturally tend to produce small files.
- Parquet is an appropriate file format because it allows for **column pruning** on top of Iceberg partition pruning, further reducing the amount of data that needs to be scanned to execute a query. Parquet also offers efficient compression and predicate pushdown, further optimizing analytical query performance.
- For frequently repeated analytical queries that don't match the partitioning strategy, derived tables/aggregations should be built to avoid maintain query performance.
- For example:
	```
	Raw clickstream
	       ↓
	Daily user/product aggregates
	       ↓
	Monthly aggregates
	```
	- These aggregates are only performed once per day/month, so performance isn't as big of a concern.
	- This allows analysts to avoid scanning billions of raw events.

## Proposed Architecture

- Putting it all together:
	```
	                         GLOBAL USERS
	                              │
	                              ▼
	                       Web / Mobile
	                              │
	                              ▼
	                     Kinesis / Kafka
	                              │
	                    ┌─────────┴─────────┐
	                    │                   │
	                    ▼                   ▼
	                  Flink               S3
	                    │               Iceberg
	                    │                   │
	          ┌─────────┴────────┐          │
	          │                  │          │
	          ▼                  ▼          ▼
	    Feature Aggregation   Raw Events   Data Science
	          │                              │
	          ▼                              │
	    Feature Store                        │
	          │                              │
	          ▼                              │
	 Personalization                         │
	                                         │
	                             Historical Replay
	                                         │
	                                         ▼
	                                      Flink
	```
- S3 / Iceberg Configuration:
	```
	S3
	│
	└── Clickstream Iceberg Table
	      │
	      ├── event_date
	      ├── advertiser/campaign dimensions
	      ├── Parquet files
	      └── Iceberg metadata
	```
	- Implements regular compaction and lifecycle management.

### Interview Questions
1. How would you partition 50 TB/day so six months of history is queryable efficiently?
> 	I'd use Iceberg on S3 with Parquet and partition primarily by event date, probably using a day-level Iceberg partition transform initially. I'd avoid partitioning directly by user because that could create an enormous number of partitions. I'd also control file sizes through compaction because streaming ingestion can create many small files. The combination of partition pruning, Parquet column pruning, predicate pushdown, and Iceberg metadata means a query over a particular time range doesn't need to scan the entire six-month dataset. For common queries that don't filter by time, I'd consider maintaining derived aggregate tables rather than repeatedly scanning the raw events.
2. Would you use Kappa or Lambda architecture?
> 	I'd lean toward a Kappa architecture because the primary computation is naturally stream-based and the historical event stream is already being durably stored in Iceberg. Kappa lets us maintain one stream-processing implementation for the real-time feature computation and replay historical events when we need to recompute features. Lambda would give us separate batch and streaming implementations, which can improve flexibility for workloads with fundamentally different processing requirements, but it also creates significant maintenance and consistency costs because the same business logic has to be implemented and kept synchronized in two systems. Given the engineering-maintenance requirement in this problem, I'd favor Kappa unless the historical computation proves fundamentally different from the streaming computation.

| Problem                       | Design decision                                                                           |
| ----------------------------- | ----------------------------------------------------------------------------------------- |
| **50 TB/day**                 | S3 + Iceberg + Parquet                                                                    |
| **Efficient 6-month queries** | Time-based partitioning + pruning + compaction                                            |
| **Real-time personalization** | Stateful stream processing → feature store                                                |
| **Maintenance cost**          | Favor Kappa when streaming and historical computation can share the same processing logic |

# Prompt 3: The Cross-Region Database Migration Pipeline
- **The Scenario:** Your company is migrating its primary transactional database (OLTP Postgres) from an on-premises data center to a cloud data warehouse (OLAP Snowflake). You need to design the Change Data Capture (CDC) pipeline to replicate data continuously with zero downtime to the production app.
- **Scale Requirements:** 5,000 database writes/updates per second across 200+ tables with evolving schemas.
- **Core Technical Challenges:**
    - **Schema Evolution:** If a software engineer runs a migration that drops or alters a column in the transactional database, how does your pipeline prevent downstream data warehouse breakage?
    - **Data Integrity Verification:** How do you programmatically verify that the data in the analytical warehouse perfectly matches the transactional source without locking the operational database tables?

## Overview

- **Primary Requirements**:
	- Continuously replicate ~5,000 writes/sec across 200+ evolving tables without taking the production application offline.
	- Prove that Snowflake matches Postgres without locking the production tables.
- **Mental Model**:
	- Use CDC to keep Snowflake continuously synchronized while Postgres remains the system of record, then validate the replica and cut over only after reconciliation passes.
- **Target Architecture**:
	```
	                 Existing Production System
	
	Application
	     │
	     ▼
	On-Prem PostgreSQL
	     │
	     │ WAL / CDC
	     ▼
	CDC Pipeline
	     │
	     ▼
	Snowflake
	     │
	     ▼
	Analytics / New Applications
	```
	- The application **continues writing to Postgres throughout the migration**.
	- The migration must take place with **zero downtime**.
	- The migration will consist of an initial full load, followed by incremental CDC. This is much more efficient than conducting a full load each time the two databases need to be synchronized.

## Ingestion Layer

- **Change Data Capture**:
	```
	INITIAL LOAD:
	Postgres
	  10 TB
	   ↓
	Copy entire database
	   ↓
	Snowflake
	
	CDC:
	INSERT
	UPDATE
	DELETE
	
	CDC EXAMPLE:
	UPDATE customer
	SET email = 'new@example.com'
	WHERE customer_id = 123;
	
	INCREMENTAL UPDATES:
	Postgres WAL
	     ↓
	CDC capture
	     ↓
	Streaming transport
	     ↓
	Snowflake
	```
	- The entire database, up to a certain point, is initially copied to Snowflake.
	- Instead of doing this for each incremental update, CDC uses the database transaction log (WAL) to track recent changes.
- **Initial Load + CDC**:
	- Initial Snapshot:
		```
		Postgres
		   │
		   ├──────────────> Initial snapshot ──> Snowflake
		   │
		   └── WAL/CDC ────────────────────────> Snowflake
		```
		- The CDC stream starts from an appropriate WAL position while the initial snapshot is being taken.
		- This allows changes occurring **during the initial load** to be captured rather than lost.
	- CDC:
		```
		Snapshot starts at LSN 1000
		
		Postgres changes:
		1001
		1002
		1003
		...
		
		Snapshot eventually finishes
		
		Then apply:
		1001
		1002
		1003
		...
		```
		- Once the initial snapshot is complete, CDC starts reading the transaction logs from an appropriate point to load incremental changes.
		- The exact mechanics depend on the CDC technology, but the principle is that you need a consistent initial state plus every change that occurred after that state.

## Processing Layer

- For an AWS-oriented architecture, AWS DMS is a natural candidate for capturing changes from an on-premises PostgreSQL database.
- DMS can perform the initial full load and ongoing CDC.
- It's important to note that DMS is not the only solution. The requirement is **CDC**, not using DMS for CDC specifically.
- Other CDC technologies could be appropriate depending on the organization's existing infrastructure.
- The important thing is that the CDC mechanism can reliably capture:
	- Inserts
	- Updates
	- Deletes
	- Transaction ordering/position
	- Schema changes, where supported/detectable

### Schema Evolution
- Suppose the application time executes:
	```sql
	ALTER TABLE customer
	DROP COLUMN phone_number;
	```
- Meanwhile, your Snowflake pipeline expects:
	- `customer_id`
	- `name`
	- `email`
	- `phone_number`
- Dropping the `phone_number` column causes downstream transformation to break.
- Even worse, suppose they change:
	```sql
	ALTER TABLE customer
	ALTER COLUMN customer_id TYPE ...
	```
	- Now the data type may no longer match the warehouse.
- You don't want all schema changes to silently flow downstream.
- **Treat Schema as a Contract**:
	- Establish a **schema-management process** around the CDC pipeline.
	- The basic flow would look like:
		```
		Postgres schema change
		        ↓
		Schema change detected
		        ↓
		Compatibility check
		        ↓
		Approved?
		   ┌────┴────┐
		   │         │
		  Yes        No
		   │         │
		   ▼         ▼
		Update      Quarantine/
		warehouse   alert
		schema
		```
	- This prevents an arbitrary production migration from unexpectedly breaking Snowflake.
- **Classify Schema Changes**:
	- Backward-Compatible:
		```sql
		ALTER TABLE customer
		ADD COLUMN loyalty_tier VARCHAR;
		```
		- This can often be handled automatically.
		- The existing consumers can continue functioning because the new column doesn't invalidate existing fields.
	- Potentially Breaking:
		- Dropping a column:
			```sql
			DROP COLUMN email;
			```
		- Changing a data type:
			```sql
			customer_id INTEGER
			       ↓
			customer_id VARCHAR
			```
		- Renaming a column:
			```sql
			email
			 ↓
			email_address
			```
		- These require much more careful handling.
- **Handling Schema Changes**:
	- Do not let schema changes blindly propagate to downstream systems.
	- Instead:
		```
		DDL change
		   ↓
		Schema registry / metadata
		   ↓
		Compatibility validation
		   ↓
		Breaking?
		   ↓
		Alert + block/quarantine downstream change
		```
	- Then the data engineering team can update the following **before approving a change**:
		- Snowflake tables
		- dbt models
		- downstream queries
		- dashboards
		- data contracts
	- This is especially important with **200+ tables**. You can't rely on engineers remembering every downstream dependency.
- **Dropping Columns**:
	- The safest migration approach is often **expand and contract**:
	- Instead of: `DROP column immediately`.
	- You do the following:
		1. Add new column
		2. Populate new column
		3. Update application
		4. Update downstream consumers
		5. Verify
		6. Stop using old column
		7. Eventually remove old column
	- During the transition, **both columns exist**.
	- Once all consumers have migrated, the column is dropped.
	- This reduces the chance of breaking downstream systems.

### Schema Validation
- During the process of updating a schema, you can't rely entirely on manual processes. A **schema validation mechanism** must be built into the pipeline.
- The CDC pipeline should detect something like:
	```
	Schema version 41
			↓
	Schema version 42
			↓
	Breaking change detected
	```
	- Once a breaking change is detected, the pipeline triggers:
		- Alerts
		- Pipeline quarantine for affected tables
		- Schema review
		- Downstream impact analysis
- **Proving Data Integrity**:
	- Suppose you've replicated the entire database. How do you verify the Snowflake table matches the corresponding Postgres table?
	- You need **non-blocking reconciliation**. To achieve this, you shouldn't compare the tables row-by-row.
	- **Level 1: Row Counts**:
		- For each table, verify the Postgress row count matches.
		- This is a useful first check, but **row counts alone aren't enough**.
		- The actual data within each row may not match.
	- **Level 2: Checksums / Hashes**:
		- A stronger approach is to calculate deterministic hashes over chunks of data.
		- For example, calculate the checksum/hash value for the first million rows in each table. If the match, continue with the next batch.
		- If a batch fails inspection, you know where to look for the inconsistency without scanning an entire table.
		- One important consideration in using checksums to prove data integrity is producing **consistent snapshots**. If a row within a batch is updated while calculating the batch's checksum, the calculation might represent a mixture of different database states.
		- Postgres's MVCC architecture gives you mechanisms for obtaining a consistent snapshot without blocking normal readers/writers in the way a table lock would.
		- The key idea is to read a consistent snapshot rather than locking the table for the duration of the reconciliation.
	- **Using a Common CDC Position / Watermark**:
		- Suppose the Postgres database has been validated up to a certain point in the transaction log: `LSN = 1,000,000`.
		- However, the Snowflake database is falling behind. It has currently only caught up to `LSN = 999,500`.
		- The databases won't match yet. That doesn't necessarily mean there's a data-integrity problem. Snowflake is simply 500 rows behind.
		- The reconciliation needs a **common consistency point**.
		- Once this point is set, only compare the databases when Snowflake has finished processing changes through that consistency point.
		- Think of it like giving Snowflake a goal. Once it reaches the goal, the comparison is performed.
- **Reconciliation Flow**:
	```
	Postgres
	   │
	   │ Create consistent snapshot
	   │
	   ▼
	Snapshot at LSN 1,000,000
	   │
	   ├── Calculate counts/checksums
	   │
	   │
	   │ CDC continues normally
	   │
	   ▼
	CDC stream
	   │
	   ▼
	Snowflake
	   │
	   │ Wait until LSN >= 1,000,000
	   ▼
	Calculate same counts/checksums
	   │
	   ▼
	Compare
	```
	- This ensures **equivalent** states are compared.
	- When there is a mismatch, drill down progressively to find it:
		```
		Table
		 ↓
		Partition
		 ↓
		Row range
		 ↓
		Individual records
		```
	- For example:
		```
		customer table
		  ↓
		partition 5
		  ↓
		customer_id 50M–51M
		  ↓
		find mismatched IDs
		```
	- Then determine whether the problem is:
		- Missing CDC event
		- Duplicate CDC event
		- Transformation error
		- Schema mismatch
		- Deleted record
		- Type conversion issue
		- Pipeline lag

### Cutover Process
- During the migration, Postgres remains the **source of truth**. Snowflake is initially a replica.
- You run the two systems in parallel until:
	```
	CDC lag ≈ 0
	+
	schema validated
	+
	reconciliation passes
	+
	performance validated
	```
- Then you can switch the appropriate downstream application/workload to Snowflake.
- **Phase 1: Initial Migration**:
	```
	Postgres
	   ↓
	Initial snapshot
	   ↓
	Snowflake
	```
- **Phase 2: Continuous CDC**:
	```
	Postgres
	   ↓
	  CDC
	   ↓
	Snowflake
	```
- **Phase 3: Validation**:
	```
	Postgres ↔ Snowflake
	       ↓
	Counts
	Checksums
	CDC watermark
	Schema validation
	```
- **Phase 4: Shadow Testing**:
	- If possible, run analytical workloads against Snowflake without making it the production source.
	- Compare and verify the results, **while maintaining the ability to roll back if necessary**.
- **Important Metrics**:
	- Monitor the following metrics during the migration:
		- CDC lag
		- WAL position
		- Replication throughput
		- Snowflake ingestion latency
		- Failed CDC events
		- Retry counts
	- For example:
		```
		Postgres LSN:     1,500,000
		Snowflake LSN:    1,490,000
		
		Lag = 10,000 changes
		```
		- A growing lag means the target isn't keeping up.
		- You should investigate:
			- Postgres CDC capture
			- CDC transport
			- Transformation
			- Snowflake ingestion
		- Then, **scale the bottleneck**.

## Proposed Architecture
- Putting it all together:
	```
	                    Production App
	                         │
	                         ▼
	                On-Prem PostgreSQL
	                    /          \
	                   /            \
	          Initial Snapshot       WAL
	                │                 │
	                ▼                 ▼
	             Staging          CDC Capture
	                │                 │
	                └────────┬────────┘
	                         ▼
	                    CDC Pipeline
	                         │
	                         ▼
	                     Snowflake
	                         │
	                ┌────────┴────────┐
	                ▼                 ▼
	             Raw/Stage        Curated Models
	                                  │
	                                  ▼
	                               Analytics
	```
- Alongside it:
	```
	Postgres                    Snowflake
	   │                            │
	   │ consistent snapshot        │
	   │                            │
	   ├── counts/checksums ───────>│
	   │                            │
	   │       CDC watermark        │
	   │<───────────────────────────│
	   │                            │
	   └──────── Reconcile ─────────┘
	```

### Interview Questions
1. How do you handle schema evolution?
> 	I'd treat the source schema as a contract and detect DDL changes as part of the CDC pipeline. Backward-compatible changes, such as adding nullable columns, can potentially be propagated automatically. Breaking changes such as dropping or changing the type of a column should trigger compatibility validation and an alert rather than blindly propagating downstream. I'd prefer an expand-and-contract migration strategy for breaking changes so downstream consumers have time to migrate. With 200-plus tables, I'd also maintain metadata and lineage so we can identify which Snowflake models and consumers are affected.
2. How do you verify Snowflake matches Postgres without locking production tables?
> 	I'd use a consistent non-blocking snapshot of Postgres and associate it with a CDC position such as a WAL LSN. I'd allow the CDC pipeline to continue normally, then wait until Snowflake has processed changes through that same position. At that point I'd compare row counts and deterministic checksums across partitioned ranges rather than comparing every row individually. If a partition doesn't match, I'd recursively drill down to identify the mismatched records. This lets us verify the two systems represent the same database state without holding production table locks.

| Challenge                   | Concept                                                |
| --------------------------- | ------------------------------------------------------ |
| Continuous replication      | **CDC / WAL**                                          |
| Zero downtime               | **Initial snapshot + CDC + parallel run**              |
| Schema changes              | **Schema contracts / compatibility / expand-contract** |
| Data correctness            | **Reconciliation / checksums**                         |
| No production locks         | **MVCC / consistent snapshots**                        |
| Comparing equivalent states | **CDC position / LSN watermark**                       |

# Using These Prompts for Interview Preparation
Pick one of the prompts above. To practice effectively, paste the prompt into a new note in your Obsidian vault using your `Company Template.md` framework, and outline your architecture design by addressing:

1. **Ingestion Layer:** (e.g., Push vs. Pull, Event Broker selection)
2. **Processing Layer:** (e.g., Micro-batching vs. True Streaming, Handling state)
3. **Storage Layer:** (e.g., Choosing the right file formats, Partitioning strategies)
4. **Trade-offs:** (e.g., Latency vs. Accuracy, Cost vs. Scale)
