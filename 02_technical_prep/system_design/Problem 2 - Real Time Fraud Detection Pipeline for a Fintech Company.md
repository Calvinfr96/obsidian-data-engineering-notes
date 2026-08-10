# Problem

## Business Context

ABC FinTech is a global digital payments company processing millions of card, wallet, and peer-to-peer transactions daily across multiple geographies. The platform supports diverse payment channels — mobile apps, e-commerce merchants, and partner APIs — and operates under strict regulatory oversight requiring transparent, auditable, and timely fraud prevention controls.

Currently, the fraud detection process relies on hourly batch jobs that analyze transactions post-authorization. This delay allows fraudulent activities to complete before alerts are generated, leading to financial loss, customer dissatisfaction, and regulatory exposure.

## Business Problem

- Current, the fraud detection process relies on hourly batch jobs, which leads to:
	- Latency in detection: Fraudulent transactions can go undetected for up to an hour before batch checks flag them.
	- Operational inefficiency: Manual reconciliation and after-the-fact blocking increase fraud operations workload.
	- Regulatory risk: Lack of real-time lineage and traceability complicates audits and compliance reporting.
	- Limited scalability: The existing ETL-based batch system struggles with spikes during promotions and peak seasons.

## Current Ecosystem

- Core Banking System: Traditional RDBMS-based (PostgreSQL / Oracle) transaction store.
- ETL / Fraud Scoring: Batch-based Spark jobs triggered hourly.
- Alerts: Manual notifications through internal dashboards and emails.
- Reporting: Tableau dashboards refreshed daily.
- Infrastructure: On-prem Hadoop cluster; limited autoscaling and real-time support.    

## Objectives

- ABC FinTech aims to modernize its fraud monitoring system with a low-latency, scalable, and compliant streaming architecture that delivers:
	- Sub-second fraud detection for all payment events. 
	- Event-driven alerts and automatic blocking of suspicious transactions before settlement. 
	- Complete lineage and traceability of each event for regulatory audits (e.g., PCI DSS, AML, GDPR). 
	- Elastic scalability to handle unpredictable transaction surges without performance degradation. 
	- Unified data lake integration for long-term storage, analytics, and model retraining.
    
## Additional Technical & Business Requirements

- Data Sources: 
	- Point-of-Sale System (POS) Data: Capture in-store transactions and customer interactions.
	- Payment APIs: Real-time credit/debit card, wallet, and P2P transactions (JSON).
	- Mobile App: Transactions and user interactions generated through ABC FinTech’s mobile applications (Android, iOS).
	- E-Commerce Merchant: Transactions processed via merchant websites, apps, and integrated payment gateways (similar to Stripe or Shopify).
	- Partner APIs: External integrations with banking partners, payment networks, AML databases, and identity verification services.  
- Daily Volume: 1–2 million events/day (~1 TB/day in peak hours)
- Data Velocity: Sub-second arrival, millisecond scoring required.  
- Data Format: JSON (streams)
- Real Time Feeds: Transactional and device fingerprint data
- Compliance: Encrypt sensitive fields (PAN, CVV, email, phone), masking, RBAC, audit trails
- Growth (1‑3 yrs): 3–5× in next 3 years from new regions, IoT payment devices, and digital wallet expansion
- Existing use‑cases: Real-time fraud scoring with sub-second latency, Auto-blocking or flagging of suspicious transactions pre-settlement, Anomaly detection: on user session patterns (behavioral biometrics).
- Primary users: Fraud Analysts, Data Scientists, Risk & Compliance Teams, Operations Teams
- Success metrics: Fraud detection latency ↓ ~60 minutes → < 5 second

# Problem Breakdown

## Business Problems

- Fraudulent transactions can go undetected for up to an hour before batch checks flag them.
- Manual reconciliation and after-the-fact blocking increase fraud operations workload.
- Lack of real-time lineage and traceability complicates audits and compliance reporting.
- The existing ETL-based batch system struggles with spikes during promotions and peak seasons.

## Functional Requirements

- Process transactions in real-time.
- Score transactions with very low latency
- Event-driven alerts and automatic blocking of suspicious transactions before settlement. 
- Complete lineage and traceability of each event for regulatory audits (e.g., PCI DSS, AML, GDPR). 
- Elastic scalability to handle unpredictable transaction surges without performance degradation. 
- Unified data lake integration for long-term storage, analytics, and model retraining.

## Non-Functional Requirements

- Process millions of card, wallet, and peer-to-peer transactions daily across multiple geographies.
- Sub-second fraud detection for all payment events. 
- Support diverse payment channels — mobile apps, e-commerce merchants, and partner APIs.
- Operate under strict regulatory oversight requiring transparent, auditable, and timely fraud prevention controls.
- **Every payment event needs to move through the system quickly enough that a fraud decision can be made before settlement**.

## Data Sources

- POS
- Payment APIs
- Mobile App
- E-Commerce Merchants
- Partner APIs
- These are fundamentally **event-producing systems**.
- For example:
	```
	Customer purchases something
	        ↓
	Payment event generated
	        ↓
	Fraud system
	```
-  Data from these sources needs to processed in real time, so we need: `Event → Streaming Pipeline → Fraud Decision`

# Solution

![](/photos/_solution_2_architecture_diagram.png)

## Proposed Architecture

```
             DATA SOURCES
                  │
                  ▼
       ┌─────────────────────┐
       │ Streaming/Ingestion │
       └─────────────────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │ Processing &        │
       │ Fraud Scoring       │
       └─────────────────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │ Decision / Actions   │
       └─────────────────────┘
          │             │
       Allow          Block
          │             │
          └──────┬──────┘
                 ▼
             Data Lake
                 │
                 ▼
             Analytics
```
- Cross-cutting conerns:
	- Security
	- Monitoring
	- Orchestration
	- CI/CD

## Data Ingestion Layer

- **Option 1**:
	```
	API Gateway
	      ↓
	   Kinesis
	      ↓
	  Firehose
	```
- **Option 2**:
	```
	API Gateway
	      ↓
	AWS Managed Kafka
	      ↓
	    Flink
	```
- **Why Streaming Is Needed**:
	- Suppose 10,000 transactions arrive simultaneously.
	- You don't want every producer directly invoking the fraud scoring service.
	- Instead, you want:
		```
		Producers
		   ↓
		Streaming platform
		   ↓
		Consumers
		```
- The streaming platform **provides a buffer** between producers and consumers.
- This gives you:
	- Decoupling
	- Buffering
	- High throughput
	- Reliability
	- Durability
	- Scalability
	- Replay
	- Multiple consumers

### Kinesis vs. Kafka
- **Kinesis** is attractive because it's an AWS-managed streaming service and integrates naturally with the rest of the AWS architecture.
- The solution emphasizes:
	- AWS-native
	- Real-time ingestion
	- Automatic scaling
	- Reliability
	- Durability
	- Batch and stream analytics
- **Kafka** provides a richer event-streaming ecosystem and gives you greater control over things such as:
	- Topics
	- Partitions
	- Consumer groups
	- Retention
	- Replay
- The solution emphasizes:
	- Rich ecosystem
	- Fine-grained retention control
	- High throughput
	- Replay / reprocessing
	- Multiple independent consumers

## Data Processing Layer

### Apache Flink
- The solution uses:
	```
	Kinesis / Kafka
	       ↓
	     Flink
	```
- Apache Flink is responsible for processing the stream of events.
- The solution specifically identifies:
	- Real-time event processing
	- Feature enrichment
	- Low latency/high throughput
	- Stateful stream processing
	- Integration with ML scoring
- **What does Flink do**:
	- Imagine this transaction:
		```json
		{
		  "user_id": 123,
		  "amount": 5000,
		  "merchant": "ABC",
		  "device_id": "XYZ",
		  "location": "New York"
		}
		```
	- The raw transaction alone may not tell us whether it's fraudulent.
	- Flink can enrich it with additional information.
	- For example:
		```
		Transaction
		     +
		User history
		     +
		Device history
		     +
		Recent transaction count
		     +
		Location
		     +
		Merchant behavior
		     ↓
		Fraud features
		```
	- For example:
		```
		amount = $5,000
		
		transactions_last_5_minutes = 8
		
		distance_from_last_transaction = 2,000 miles
		
		device_seen_before = false
		```
	- Now the ML model has useful features to evaluate.

### DynamoDB
-  DynamoDB is used as feature store in this problem.
- This is very different from using DynamoDB as the primary analytical database.
- DynamoDB is well suited for looking up an item in a database by its key.
- DynamoDB is not well suited for heavy analytical workloads, such as:
	- Scanning massive amounts data
	- Performing complex joins.
	- Grouping data by a column.
- DynamoDB isn't appropriate as the analytical data store.
- DynamoDB is appropriate for low-latency feature lookups during real-time scoring.

### SageMaker
- SageMaker is primarily responsible for performing the fraud detection.
- After fink enriches an event:
	```
	Transaction
	      ↓
	Flink
	      ↓
	Features
	      ↓
	SageMaker
	      ↓
	Fraud Score
	```
	- SageMaker generates a `Fraud probability` score.
	- SageMaker is responsible for deploying and serving the ML model.
- The solution highlights:
	- Managed model deployment
	- Model management/versioning
	- Scalable inference
	- Serverless inference
	- Integration with real-time processing
- The score produced by the ML model is used to make a decision regarding the transaction:
```
Model Score
     ↓
Decision
     ↓
Action (Allow, Notify, or Block)
```

## Action Layer

- The solution has three major actions:
	- **Allow**: Let the payment continue.
	- **Block**: Prevent the suspicious transaction from proceeding.
	- **Notification**: Alert fraud operations.

### SNS
- SNS is useful because we don't necessarily want the fraud scoring service to directly integrate with every notification destination.
- Instead:
	```
	Fraud Decision
	      ↓
	     SNS
	    /   \
	   ↓     ↓
	Slack   Other systems
	```
	- This gives us a decoupled notification mechanism.

### Critical Path vs. Analytics Path
- Critical Path:
	```
	Transaction
	    ↓
	Kinesis/Kafka
	    ↓
	Flink
	    ↓
	DynamoDB
	    ↓
	SageMaker
	    ↓
	Decision
	    ↓
	Allow/Block
	```
	- Needs to be extremely fast to prevent fraudulent transactions.
- Analytics Path:
	```
	Transaction
	     ↓
	Raw Bucket
	     ↓
	Data Lake
	     ↓
	Bronze
	     ↓
	Silver
	     ↓
	Gold
	     ↓
	Analytics
	```
	- Doesn't need millisecond latency.
	- You don't want your fraud decision waiting for a data warehouse or analytics pipeline.

### S3
- The fraud system needs long-term storage for:
	- Historical analysis
	- Regulatory audits
	- Model retraining
	- Investigation
	- Event replay
- The prompt explicitly requires unified data-lake integration for long-term storage, analytics, and model retraining.
- Therefore, the streaming path writes data to the staging/raw buckets.

### Medallion Architecture
- Bronze:
	- Raw event.
	- Useful for:
		- Auditing
		- Replay
		- Debugging
- Silver:
	- Cleaned/processed event.
	- Validated transactions
	- Enriched features
	- Standardized schema
- Gold:
	- Analytics-ready data
	- Fraud statistics
	- Fraud trends
	- Customer risk profile
	- Model performance
- Now analysts can use the data without interfering with the real-time fraud detection path.

### Redshift / Snowflake & QuickSight
- Redshift / Snowflake is not responsible for making the real-time fraud decision. They simply act as data warehouses.
- Fraud detection is performed by downstream analytics.

### Compliance
- The prompt explicitly mentions:
	- PCI DSS
	- AML
	- GDPR
	- PAN
	- CVV
	- Email
	- Phone
	- mMasking
	- RBAC
	- Audit trails
- The solution explicitly includes IAM for role-based access control and CloudWatch for centralized monitoring/logging.
- Interview Answer:
> 	Because we're processing payment and personally identifiable information, I'd encrypt sensitive data at rest and in transit, restrict access using IAM roles, mask sensitive fields where possible, and maintain audit logs so we can trace an event through ingestion, scoring, and the final decision.

### Handling Traffic Spikes
- The requirement says the system must handle **3–5× growth** and unpredictable spikes.
- This is why **streaming** infrastructure is preferable to the existing batch architecture.
- Instead of:
	```
	Fixed-size servers
	       ↓
	Process everything
	```
- We want:
	```
	Traffic spike
	     ↓
	Streaming infrastructure absorbs traffic
	     ↓
	Processing scales
	     ↓
	Events continue flowing
	```
	- Kinesis/Kafka provides buffering, while Flink provides scalable stream processing.

### Failure Scenarios
- Suppose Flink crashes. You don't want to lose transactions.
- That's another reason we have the streaming layer.
	```
	Producer
	   ↓
	Kafka/Kinesis
	   ↓
	Flink crashes
	```
	- The events **remain available** in the streaming system and can be processed again.
	- The solution specifically emphasizes reliability, durability, and replay/event reprocessing.

### End-to-End Flow
- If the interviewer asks: "Walk me through the end-to-end flow":
> 	A payment event originates from a POS system, mobile app, e-commerce merchant, or partner API and enters the streaming layer through API Gateway. The event is written to Kinesis or managed Kafka, which decouples producers from consumers and provides buffering and durability. Flink consumes the event and performs real-time processing and feature enrichment, retrieving additional features from DynamoDB. The enriched event is sent to a SageMaker endpoint for fraud scoring. The resulting score is evaluated by the decision layer, which determines whether the transaction should be allowed or blocked and can publish an alert through SNS. At the same time, the raw and processed events are persisted to the data lake so they can be used later for analytics, auditing, and model retraining.

## Architecture Overview

```
                    ┌──────────────┐
                    │ POS          │
                    │ Payment API  │
                    │ Mobile App   │
                    │ E-Commerce   │
                    │ Partner API  │
                    └──────┬───────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ API Gateway      │
                  └────────┬─────────┘
                           │
                  ┌────────▼─────────┐
                  │ Kinesis / Kafka  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Flink            │
                  │ Stream Processing │
                  └───────┬──────────┘
                          │
                 ┌────────┴─────────┐
                 │                  │
                 ▼                  ▼
        ┌────────────────┐   ┌───────────────┐
        │ DynamoDB       │   │ SageMaker     │
        │ Feature Store  │──▶│ Fraud Model   │
        └────────────────┘   └───────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │ Decision     │
                              └──────┬───────┘
                                ┌────┴────┐
                                ▼         ▼
                              ALLOW     BLOCK
                                          │
                                          ▼
                                      SNS / Slack


             Meanwhile:

Kinesis/Kafka ────────▶ S3 Raw
                          │
                          ▼
                       Bronze
                          │
                          ▼
                       Silver
                          │
                          ▼
                        Gold
                          │
                          ▼
                  Redshift/Snowflake
                          │
                          ▼
                     BI / Analytics
```

### Decision Making

|Decision|Reason|
|---|---|
|**Streaming instead of batch**|Need fraud decisions within seconds|
|**Kinesis/Kafka**|Buffering, decoupling, durability, replay|
|**Flink**|Stateful real-time processing and feature enrichment|
|**DynamoDB**|Extremely fast feature lookups during scoring|
|**SageMaker**|Real-time ML inference|
|**SNS/action layer**|Turn fraud scores into operational actions|
|**S3 + Redshift/Snowflake**|Durable historical storage and downstream analytics|
- The real-time path exists to make the fraud decision. The data-lake path exists to preserve and analyze the data.

## Questions

### Why is Flink used instead of another AWS service?
- The key reason is that **Flink is doing a different job from most of the AWS services in the architecture**.
- Lambda would be a reasonable AWS-native alternative for **simple event processing**, such as:
	```
	Event
	  ↓
	Lambda
	  ↓
	Validate / transform
	```
	- But fraud detection often requires **stateful stream processing**.
	- For example, suppose the current transaction is $5,000 purchase. This doesn't necessarily indicate fraud.
	- We might need to ask:
		- How many transactions has this user made in the last 5 minutes?
		- How many different countries?
		- Has this device been used before?
		- What was the user's previous transaction?
		- How much has the user spent recently?
	- Flink is designed to maintain and process this kind of **state over a stream of events**.
- Lambda's execution time and resource limits play a factor in the decision to use Flink, but it's not the primary reason. The bigger distinction is:
	- Lambda is primarily a stateless event-processing function.
	- Flink is a distributed stream-processing engine designed to continuously process events while maintaining state across those events.
	- Suppose we receive:
		- Transaction 1: $50
		- Transaction 2: $75
		- Transaction 3: $5,000
	- To determine whether Transaction 3 is fraudulent, we may need context such as:
		- Transactions in the last 5 minutes
		- Total spending in last hour
		- Number of distinct locations
		- Number of devices used
		- Time since previous transaction
	- That's **state over a stream**, rather than simply processing one event independently.
	- Flink is specifically suited to this. The provided solution calls out **"stateful stream processing"**, along with real-time event processing and feature enrichment, as reasons for using Flink.
- Conceptually:
	```
	Transaction stream
	       ↓
	     Flink
	       ↓
	Maintain state
	       ↓
	Calculate features
	       ↓
	Fraud model
	```
	- That's why the solution specifically calls out **"stateful stream processing"** as one of Flink's benefits.
- AWS-Native Alternatives:
	- **Lambda** — lightweight event processing
	- **Kinesis Data Analytics / Managed Service for Apache Flink** — stream processing
	- **Glue Streaming** — streaming ETL
	- **EventBridge** — event routing
	- **Amazon Managed Service for Apache Flink** — Allows you to use Flink in AWS while the underlying service is managed for you.
- Conclusion: Why use Flink instead of AWS?
> 	For this design, Flink is a good fit because the fraud pipeline requires stateful, low-latency stream processing and feature enrichment. We could use an AWS-managed Flink offering to avoid managing the underlying infrastructure.
	- This design is an example of one of the places where the **requirements should drive your technology choice**.
	- If the problem simply said, "process each event and validate the JSON," Flink would be overkill.
	- Once you introduce **real-time fraud detection, stateful features, and millisecond-level scoring**, a stream-processing engine becomes much more justified.
#### Where Lambda becomes less attractive
- You could build the fraud detection pipeline with Lambda:
	```
	Kinesis
	   ↓
	Lambda
	   ↓
	DynamoDB
	   ↓
	SageMaker
	```
	- For relatively simple processing, Lambda is a perfect fit.
- As the the stream-processing logic becomes more sophisticated, you'd end up having Lambda functions repeatedly:
	1. Receive an event.
	2. Retrieve state from DynamoDB.
	3. Update state.
	4. Calculate features.
	5. Handle ordering/windowing concerns.
	6. Invoke the ML model.
- At that point you're effectively **building your own stream-processing framework around Lambda + DynamoDB**.
- Flink gives you **stream-processing abstractions** directly:
	```
	Event stream
		 ↓
	Flink
	 ┌───────────────┐
	 │ Windows       │
	 │ State         │
	 │ Aggregations  │
	 │ Enrichment    │
	 │ Event time    │
	 └───────────────┘
		 ↓
	Fraud features
	```
	- This is the more important reason for choosing Flink over Lambda.
- Lambda functions have constraints around:
	- Maximum execution duration
	- Memory/CPU
	- Concurrency
	- Per-invocation overhead
	- Managing state externally
- For a simple event transformation, these aren't problems.
- For a **continuous, stateful, high-throughput stream-processing workload**, a dedicated stream processor is a better abstraction.
- Finally, **Flink itself can run as an AWS-managed service**, so choosing Flink doesn't necessarily mean you're choosing to manage servers yourself.
- **Important latency consideration**:
	- You might think: "If Lambda is serverless, shouldn't Lambda actually be faster?"
		- Not necessarily. For a single isolated event, Lambda can absolutely be very fast.
	- The problem is the **overall stream-processing workload**, particularly when you need:
		- High throughput
		- State
		- Windowing
		- Event ordering
		- Feature enrichment
		- Consistent processing
	- Flink is designed to keep a continuously running processing topology rather than **repeatedly invoking an independent function** for each event.
	- That can make latency and throughput much more predictable for this type of workload.
- Conclusion: Why choose Flink over Lambda?
> 	Lambda would be a reasonable choice for simple stateless event processing, and the execution limits are one consideration. But the bigger reason I'd choose Flink is that fraud detection requires stateful stream processing. We need to maintain things like transaction windows and behavioral features across events, perform enrichment, and process the stream continuously at high throughput. Lambda would require us to build much of that state-management and stream-processing logic ourselves, typically using something like DynamoDB. Flink provides those stream-processing capabilities natively. So I'd choose Lambda for relatively simple event transformations, but Flink for the more complex stateful streaming workload in this design.
	- The **stateful** nature of the event stream processing is the key consideration. Lambda can't handle this natively.
	- Lambda: "Do something when an event arrives."
	- Flink: "Continuously analyze a stream of events while maintaining state about that stream."

### How would you justify putting API Gateway in front of every event source?
- The justification is that these are **external producers sending events into our system**, so API Gateway provides a controlled ingress point.
- It can provide things such as:
	- Authentication/authorization
	- Request validation
	- Throttling
	- Rate limiting
	- Centralized API management
	- Monitoring
	- Protection of downstream services
- Although API Gateway is the primary ingress point in the proposed design and provides many benefits, it can make the pipeline unnecessarily rigid.
- For example, if a payment process already publishes events directly into Kafka, you could use:
	```
	Payment Processor
	       ↓
	Kafka
	
	Instead of:
	Payment Processor
	       ↓
	API Gateway
	       ↓
	Kafka
	```
	- This avoids an unnecessary network hop and latency.
- Conclusion: Why did you put API Gateway in front of all these sources?
> 	The intent is to provide a standardized, authenticated ingress point for external systems that are submitting events through APIs. It gives us centralized validation, throttling, authorization, and monitoring before events enter the streaming platform. However, I wouldn't require every producer to go through API Gateway. If a source can publish directly to our streaming platform securely and reliably, I'd prefer that path to avoid unnecessary latency and infrastructure.

### How would you guarantee the latency target?
- The key is that you don't really **guarantee** a latency target by choosing a particular service. You design the critical path so that you have a measurable latency budget, eliminate unnecessary synchronous work, and monitor the **p95/p99**, not just the average.
- **Start with defining a latency budget**:
	- For example:
		```
		Transaction arrives
		      ↓
		Ingestion
		      ↓
		Stream processing
		      ↓
		Feature lookup
		      ↓
		ML inference
		      ↓
		Fraud decision
		      ↓
		Allow / Block
		```
	- An internal latency budget could look like:
		```
		Ingestion          200 ms
		Flink processing   500 ms
		Feature lookup     100 ms
		ML inference       500 ms
		Decision            50 ms
		                   ─────
		                   1.35 sec
		```
		- Those numbers are **illustrative**, not requirements from the problem. The important point is leaving enough headroom below the **5-second end-to-end target**.
- **Keep the fraud detection path short**:
	- You **do not** want:
		```
		Transaction
		 ↓
		S3
		 ↓
		Glue
		 ↓
		Redshift
		 ↓
		Query
		 ↓
		ML
		 ↓
		Decision
		```
		- This is an **analytics** pipeline, not a real-time fraud detection pipeline.
	- Instead:
		```
		Transaction
		     ↓
		Kinesis / Kafka
		     ↓
		Flink
		     ↓
		DynamoDB
		     ↓
		SageMaker
		     ↓
		Decision
		```
		- The solution explicitly separates the **Processing & Scoring/Decision Layer** from the data-lake path.
		- The data lake can be populated **asynchronously** without blocking the fraud decision.
- **Keep feature lookups low latency**:
	- The solution uses DynamoDB as the feature store.
	- That's important because Flink might need information such as:
		- Customer's recent transaction count
		- Device history
		- Recent spending
		- Risk features
	- You don't want the scoring path doing a query against a large analytical database.
	- Instead:
		```
		Flink
		  ↓
		DynamoDB
		  ↓
		Features
		```
		- The lookup should be predictable and fast.
		- If the feature store becomes slow, it directly affects fraud latency, so I'd monitor its latency and provision enough capacity for peak traffic.
- **Don't make the data lake synchronous**:
	- This is a subtle but important point. We need to preserve the transaction for:
		- Auditing
		- Analytics
		- Model training
		- Investigation
	- None of those should hold up the fraud detection pipeline.
	- Conceptually:
		```
		                   ┌──> Fraud scoring ──> Decision
		                   │
		Transaction ───────┤
		                   │
		                   └──> Data Lake
		```
- **Avoid synchronous calls to slow external systems**:
	- Suppose you need information from an external partner API:
		```
		Flink
		  ↓
		Bank API
		  ↓
		Wait 2 seconds
		  ↓
		SageMaker
		```
		- That makes your latency dependent on another company's service.
		- Avoid putting external calls on the critical path whenever possible.
	- Instead, maintain relevant information asynchronously in the feature store:
		```
		Partner API
		     ↓
		Feature update
		     ↓
		DynamoDB
		
		Transaction
		     ↓
		Flink
		     ↓
		DynamoDB lookup
		     ↓
		Scoring
		```
		- Now, the scoring path isn't waiting on the external API.
- **Design for peak, not average traffic**:
	- The problem says the system needs to handle **3–5× growth** and unpredictable transaction surges.
	- This matters enormously for latency.
	- One day, the system might average 500 events/sec with 500 ms latency.
	- During a promotion, the system might average 2,500 events/sec with 8 second latency.
	- The system should be tested at:
		- Expected average throughput
		- Expected peak throughput
		- 3x growth
		- 5x growth
	- Each test should ensure latency SLAs are not breached.
- **Monitor P99 latency, not average latency**:
	- P99 and P95 latency should always be the important operational metric because it represents how the system performs when pushed to its limits.
	- CloudWatch alarms should be set on P99 and P95 latency, P50 latency.
- **Measure each component separately**:
	- If the end-to-end latency suddenly increases, you need to understand why.
	- Latency for each step in the pipeline should be monitored separately:
		```
		API Gateway latency
		        +
		Kinesis/Kafka processing delay
		        +
		Flink processing latency
		        +
		DynamoDB lookup latency
		        +
		SageMaker inference latency
		        +
		Decision latency
		```
		- This helps identify where the bottleneck is, giving you a concrete troubleshooting strategy.
- **Monitor streaming metrics**:
	- Fraud detection latency can be directly affected by **consumer lag**, where events are being produced more quickly than they're being processed.
	- This leads to a backup in transaction events, causing slower fraud detection.
	- Monitor:
		- Stream throughput
		- Consumer lag
		- Processing throughput
		- Processing latency
		- Throttling
- **Be careful using the word "guarantee"**:
	- You shouldn't say:
> 	I guarantee every transaction will always be processed in under 5 seconds.
	- Distributed systems can't realistically provide that kind of absolute guarantee under every possible failure condition.
	- Instead, say:
> 	I'd define an SLO around the five-second requirement—for example, a target p99 end-to-end fraud-decision latency—and design and load-test the system to maintain that SLO under expected peak traffic. I'd monitor each stage of the critical path and scale or alert when latency or queue depth approaches the threshold.
- Conclusion: How would you guarantee the latency target?
> 	I'd start by defining a latency budget for the critical path from transaction ingestion through the fraud decision. I'd keep that path as short as possible: Kinesis or Kafka for ingestion, Flink for stateful processing and feature enrichment, DynamoDB for low-latency feature lookups, and SageMaker for model inference. The data-lake and analytics path would be asynchronous so it doesn't block the decision. I'd provision and load-test the system against peak traffic and the expected 3–5× growth, monitor consumer lag and processing throughput, and track p95 and especially p99 end-to-end latency. CloudWatch would provide monitoring and alarms if any component approaches its latency budget. That gives us an enforceable SLO rather than simply assuming the architecture will be fast enough.

### Why use Airflow instead of Step Functions?
- Airflow isn't inherently better than Step Functions, it was just the chosen orchestration tool.
- **Airflow fits the data pipeline**:
	- In this architecture, orchestration is primarily about coordinating data-processing workflows such as:
		```
		Raw data arrives
		      ↓
		Flink processing
		      ↓
		Store processed data
		      ↓
		Transform Bronze → Silver
		      ↓
		Transform Silver → Gold
		      ↓
		Refresh downstream analytics
		```
	- Airflow is designed around DAGs with tasks, dependencies, scheduling, retries, and monitoring. That makes it a natural fit for coordinating multi-step data workflows.
- **Step Functions would also be a legitimate choice**:
	- Step Functions would make a lot of sense if the workflow were primarily an **AWS application workflow**, for example:
		```
		Transaction received
		      ↓
		Validate
		      ↓
		Call Lambda
		      ↓
		Call another AWS service
		      ↓
		If fraud → block
		      ↓
		If legitimate → allow
		```
	- It gives you state-machine semantics and integrates very naturally with AWS services.
	- However, that's not really the primary orchestration problem in this architecture. The **fraud decision itself should happen in the real-time processing path**, not wait for an Airflow DAG or Step Functions workflow.
- **Airflow or Step Functions should not orchestrate the critical fraud path**:
	```
	❌ Transaction
	   ↓
	Airflow
	   ↓
	Flink
	   ↓
	SageMaker
	   ↓
	Decision
	
	OR
	
	❌ Transaction
	   ↓
	Step Functions
	   ↓
	Flink
	   ↓
	SageMaker
	   ↓
	Decision
	```
	- introduces an orchestration layer into a path where we're trying to achieve very low latency.
	- Instead:
		```
		Transaction
		     ↓
		Kinesis / Kafka
		     ↓
		Flink
		     ↓
		DynamoDB + SageMaker
		     ↓
		Fraud Decision
		```
		- Airflow (MWAA) or Step Functions handles **background/data workflows** around that system.
- Conclusion: Why Airflow instead of Step Functions?
> 	The solution uses MWAA for orchestration, although the problem doesn't explicitly require Airflow over Step Functions. I'd choose Airflow if the orchestration requirement is primarily coordinating scheduled, multi-step data pipelines with dependencies between data-processing jobs. Step Functions would be a strong alternative for AWS-native application workflows where we're coordinating individual service calls and state transitions. In either case, I wouldn't put the orchestrator on the critical fraud-detection path because our latency requirement is under five seconds.
	- This is a good answer because you're **defending the architectural choice without pretending the reference solution established a requirement that it didn't**.
	- Remember:
		- Flink = Real-time processing
		- SageMaker = Fraud model inference
		- DynamoDB = Low-latency feature lookup
		- SNS = Notifications
		- S3/Redshift/Snowflake = Durable analytics
		- MWAA/Airflow = Background workflow orchestration
		- The **separation of responsibilities** is more important than individual technology choices.