# Foundations

## 1. Batch vs. Stream Processing

- Imagine you're building a system that processes customer orders.
	- Should it process one million orders every night?
	- Should it process each order after it's placed?

###  Batch Processing
- Batch processing collects data over a period of time and processes it all at once.
- For example:
	```
	Orders
	
	↓
	
	Collected all day
	
	↓
	
	Midnight Spark Job
	
	↓
	
	Analytics Tables
	```
	- Instead of processing every event immediately, the system waits until enough data has accumulated.
- Examples of batch processing include:
	- Daily sales reports
	- Payroll
	- Monthly billing
	- Historical analytics
	- Data warehouse refreshes
- Advantages:
	- **Efficiency:** Starting a distributed job has overhead. Running one large job is more efficient than running millions of tiny jobs.
	- **Easy Recovery:** If a nightly batch job fails, simply delete the output and run it again.
	- **Historical Analysys:** Batch jobs make it easier to perform analysis on large amounts of data.
- Disadvantages:
	- **Data Freshness:** If a batch job runs every night and finishes at 9AM, customers who placed orders at 9:01AM won't appear in any reports until the next day.
	- **Latency:** There's a 1-day gap between reports for daily batch jobs.

### Stream Processing
- Streaming processes data continuously as it arrives:
	```
	Order
	
	↓
	
	Kafka
	
	↓
	
	Consumer
	
	↓
	
	Database
	
	↓
	
	Dashboard
	```
	- Each event flows through the data pipeline almost immediately.
- Examples of stream processing include:
	- Fraud detection
	- Live dashboards
	- IoT sensors
	- Recommendation systems
	- Stock trading
	- Monitoring systems
- Advantages:
	- **Latency:** Latency is very low compared to batch processing, since data is processed almost immediately when it arrives.
	- **Alerting:** Stream processing is great for monitoring metrics and alerting when metrics breach a defined SLA.
- Disadvantages:
	- **Complexity:** Stream processing is significantly more complex than batch processing.
	- Stream processing needs to handle:
		- Failures
		- Duplicates
		- Out-of-order events
		- Late-arriving data
		- State management
- **Important Note:**
	- Stream processing in most streaming frameworks (like Spark Structured Streaming and Apache Flink) don't literally process one event at a time. Instead, they often process **micro-batches**. For example:
		```
		0.5 seconds worth of events
		
		↓
		
		Process together
		
		↓
		
		Repeat
		
		↓
		
		Repeat
		
		↓
		
		Repeat
		```
		- Instead of processing one million events once per day, you might process:
			- 2,000 events per second.
			- 10,000 events every 5 seconds.
		- To the user, it feels like real time, but the system gains efficiency by grouping events into very small batches.

### Visual Comparison
- Batch:
	```
	□□□□□□□□□□□□
	
	↓
	
	One large job
	
	↓
	
	Results
	```
- Streaming:
	```
	□
	
	↓
	
	Process
	
	↓
	
	□
	
	↓
	
	Process
	
	↓
	
	□
	
	↓
	
	Process
	```
- **Real-World Example:** Every Amazon order creates an event. Some consumers need immediate updates, while others don't.
	- For example, Inventory needs immediate updates to prevent overselling.
	- However, finance can handle batch updates because they primarily use the data for monthly revenue reporting.
	- The same order event is processed using two different workflows.
	- Each workflow corresponds to the business requirements of the consumer.
- **Modern Architectures:** Many companies use both batch and stream processing.
	```
	Application
	
	↓
	
	Kafka
	
	↙        ↘
	
	Streaming     Batch
	
	Alerts        Nightly ETL
	
	Fraud         Reporting
	
	Monitoring    Analytics
	```
	- This is called a **Lambda Architecture** (although many modern systems have evolved beyond the original definition).
	- The important idea is that different consumers have different latency requirements.
- **Data Engineering Example:**
	- Imagine Amazon Ads, where every cache warming event is published.
	- Streaming pipeline:
		- Detect failures
		- Alert on spikes
		- Monitor latency
	- Batch pipeline:
		- Weekly reports
		- Capacity planning
		- Trend analysis
	- It's the same source data, but different processing goals.

### Tradeoffs
| Batch                        | Streaming                 |
| ---------------------------- | ------------------------- |
| High throughput              | Low latency               |
| Simpler                      | More complex              |
| Easier recovery              | Harder recovery           |
| Historical analysis          | Real-time insights        |
| Less infrastructure overhead | Continuous resource usage |

- One is not better than the other. They're optimized for different priorities.
- Streaming is not always better just because it's faster.
	- Imagine a company that processes payrol, like ADP. Would they rather process payroll data every second or every week?
	- No. In this case, streaming adds complexity without adding business value.
- When deciding between the two, engineers must ask: How fresh does the data need to be?
	- The answer may be different for different parts of an application or organization.
	- When data is needed at regular intervals, such as daily or weekly, batch processing is the simpler, more cost-effective approach.
	- When data is needed **immediately**, stream processing offers lower latency at the cost of more complexity.

### Common Interview Questions
1. When would you choose batch over streaming?
> 	I would choose batch when low latency isn't a business requirement and the workload involves processing large volumes of historical data efficiently, such as nightly ETL jobs or generating financial reports. Batch processing is simpler, more cost-effective, and easier to recover if failures occur.
2. When would you choose streaming?
> 	I'd choose streaming when the business needs near real-time insights or actions, such as fraud detection, live dashboards, or monitoring systems. Although streaming systems are more complex, they provide low-latency processing that batch systems cannot.
3. A retail company generates **10 million purchase events per day**. Executives receive a sales report every morning at 8:00 AM. Would you recommend batch or stream processing for this reporting workload?
> 	I would recommend batch processing because the business only requires a report once each morning. Since there is no need for real-time updates, a nightly batch job is more cost-effective, simpler to operate, and well suited for processing the previous day's 10 million purchase events.
4. A payment processing system must detect potentially fraudulent transactions **within five seconds** so suspicious payments can be blocked before they're approved. Would you choose batch or stream processing? What tradeoffs would you be accepting with that decision?
> 	I'd choose stream processing because fraud detection requires near real-time analysis so suspicious transactions can be blocked before they're approved. While this increases operational complexity and infrastructure costs, the low latency is essential to meeting the business requirement.
5. Can you run batch jobs on streaming data?
	- Yes, imagine this architecture:
		```
		Application
		
		↓
		
		Kafka
		
		↙        ↘
		
		Streaming        Batch
		
		Fraud            Nightly ETL
		
		Alerts           Reports
		
		Dashboard        ML Training
		```
		- The same real-time stream of events can feed:
			- A **streaming application** for real-time decisions.
			- A **batch application** that processes all accumulated events overnight.
		- Batch and streaming are **complementary**, not competing technologies.

## 2. ETL vs. ELT

- ETL:
	```
	Extract
	
	↓
	
	Transform
	
	↓
	
	Load
	```
	- Data is transformed **before** it reaches its destination.
- ELT:
	```
	Extract
	
	↓
	
	Load
	
	↓
	
	Transform
	```
	- Data is loaded first, then transformed later.
	- This seems like a small difference, but is actually a huge architectural shift.

### ETL
- Suppose you're extracting customer orders. The raw data looks like:
	```
	OrderID
	
	CustomerName
	
	Address
	
	Phone
	
	Country
	
	Product
	
	Price
	
	Timestamp
	```
- Suppose your data warehouse only needs:
	```
	OrderID
	
	Country
	
	Price
	```
- With ETL:
	```
	Source
	
	↓
	
	Transform
	
	↓
	
	Keep only needed columns
	
	↓
	
	Warehouse
	```
	- The warehouse never sees the raw data.
- ETL was so popular because databases, storage, and compute were very expensive. Companies only wanted to load cleaned, useful data.
- Advantages:
	- Cleaner warehouse.
	- Lower storage.
	- Business rules enforced before loading.
	- Sensitive data removed before storage.
- Disadvantages:
	- Once data is loaded, it needs to be re-extracted and re-transformed if data requirements change.
	- For example, if a customer's phone number was filtered from order data during transformation, but six months down the line, analyzing phone numbers becomes beneficial, you need to back and re-process all of the data.

### ELT
- Now imagine a modern cloud warehouse:
	```
	Source
	
	↓
	
	Load EVERYTHING
	
	↓
	
	Warehouse
	
	↓
	
	Transform with SQL
	```
	- Raw data stays available. It doesn't need to be re-extracted from the source and re-processed.
- ELT has become so popular because Cloud warehousing changed the economics. Storage became relatively inexpensive and compute became elastic.
	- Companies stopped asking: "Can we afford this?"
	- Companies started asking: "Can we afford **not** to store it?"
- **Modern Example:**
	- Imagine Snowflake. All raw data is loaded from a source such as S3:
		```
		Orders
		
		Customers
		
		Products
		
		Logs
		
		Events
		
		Clicks
		```
	- Later, different teams build different transformations based on their specific use cases.
	- Everyone starts from the same raw data.

### Visual Comparison
- ETL:
	```
	Source
	
	↓
	
	Transform
	
	↓
	
	Warehouse
	```
- ELT:
	```
	Source
	
	↓
	
	Warehouse
	
	↓
	
	Transform
	```
- The difference is subtle, but has huge implications.
- **Data Engineering Example:**
	- Suppose Amazon Ads collects cache metrics. The raw data contains:
		- Cache key
		- Timestamp
		- Service
		- Region
		- Host
		- Deployment version
		- Retry count
		- Debug metadata
	- Finance only needs latency and region.
	- With ETL:
		- Everything else is discarded.
		- If business requirements change in the future, it's too late. The data is gone.
	- With ELT:
		- Raw events are still available.
		- You just need to create a new transformation.

### Tradeoffs
- ETL:
	- Pros:
		- Smaller warehouse
		- Lower storage
		- Strong governance
		- Sensitive data removed early
		- Higher data quality before loading
	- Cons:
		- Less flexible
		- Reprocessing may require going back to the source
		- Harder to support new analytical questions
- ELT:
	- Pros:
		- Maximum flexibility
		- Preserve raw history
		- Easy to build new transformations
		- Better for modern analytics
		- Supports many downstream consumers
	- Cons:
		- Higher storage usage
		- Requires governance over raw data
		- Sensitive information must be protected after loading
		- Poorly managed raw zones can become disorganized
- Notice that ELT doesn't eliminate the need for governance—it shifts _where_ governance happens.
	- ETL implements data governance during transformation.
	- ELT implements data governance during loading.
- **Real-World Architecture:** Many modern data lakes often look like this:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	S3 (Raw)
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Curated Tables
	
	↓
	
	Dashboards
	```
	- The raw layer is preserved.
	- Curated data sets are built from it.
	- If business requirements change, raw data can be re-processed.
- ELT fits the Cloud so well because Cloud platforms such as Snowflake, BigQuery, and Databricks are designed to perform transformations efficiently _after_ the data has been loaded.
	- Instead of maintaining a large ETL server that performs all transformations before loading, organizations can leverage the scalable compute available within the warehouse or lakehouse itself.
	- That's one of the biggest reasons ELT has become the dominant architecture for modern analytics platforms.
- ETL is still used in transactions where data is **required** to be transformed before it is stored, such as finance or healthcare.
	- In these sectors, Data Engineers may be required to perform the following actions **before** data is stored:
		- Remove personally identifiable information (PII)
		- Validate data quality
		- Standardize formats
	- These are great use cases for ETL.
	- Another great use case for ETL is when storage or network bandwidth is constrained and you don't want to move unnecessary data.
- ELT did not **replace** ETL. It's more accurate to say:
	- Modern analytics platforms often favor **ELT** because storage is relatively inexpensive and flexibility is valuable.
	- Some workloads still benefit from **ETL**, especially when compliance, governance, or data minimization requirements dictate that data be transformed before it is stored.

| ETL                         | ELT                                    |
| --------------------------- | -------------------------------------- |
| Transform before loading    | Transform after loading                |
| Less flexible               | More flexible                          |
| Smaller destination         | Raw data retained                      |
| Good for strict governance  | Good for exploratory analytics         |
| Traditional data warehouses | Modern cloud warehouses and lakehouses |

### Common Interview Questions
1. Why has ELT become more popular?
>	Modern cloud data platforms provide scalable storage and compute, making it practical to load raw data first and perform transformations later. This gives organizations greater flexibility because they can create new transformations without re-ingesting historical data, while still leveraging the warehouse's compute resources for efficient processing.
2. Your company uses **Snowflake** for analytics. Business teams frequently ask new questions about historical customer data that weren't anticipated when the pipeline was built. Would you recommend ETL or ELT? Why?
> 	I would recommend ELT because business requirements frequently change and there are no strict guidelines governing how data is stored. ELT generally requires storing more raw data, but modern cloud platforms make that tradeoff worthwhile because they provide inexpensive storage and scalable compute. Resource utilization in this scenario would be elevated by regularly needing to re-extract and re-process data for evolving analytical workflows.
3. A healthcare organization processes patient records containing sensitive personal information. Regulations require that certain fields be removed **before** the data is stored in the analytics platform. Would you recommend ETL or ELT?
> 	Since regulations require certain fields be removed before data is stored, ETL would be a better fit because the data is transformed before being loaded into the data warehouse. Although ELT is more flexible and preserves historical raw data, ETL makes it easier to comply with regulations.
4. If ELT is so flexible, why doesn't everyone just store everything forever?
	- **Governance:** Not every team should have access to raw sensitive data.
	- **Compliance:** Regulations like HIPAA or GDPR may require limiting what is stored.
	- **Data quality:** Raw data may be incomplete or malformed and shouldn't always be exposed directly to analysts.
	- **Cost:** Although storage is cheaper than it used to be, it still isn't free—especially at petabyte scale.

## 3. Data Pipelines

- A Data Engineer's primary responsibility is to **build reliable data pipelines**.
- A data pipeline is a system that moves data from one place to another while optionally transforming it along the way.
	- Think of a data pipeline like an assembly line. In an assembly line, raw materials are first assembled, then undergo quality checks, then they are packaged, then they are sent to customers.
	- In a data pipeline, data is ingested from a source, transformed, stored, and consumed.
	- The data changes form as it moves through the data pipeline.

### Typical Pipeline Stages
1. Ingestion: Getting the data
	- Examples of sources include:
		- Database
		- Kafka
		- API
		- Log files
		- IoT devices
		- S3 uploads
	- Nothing has been transformed as a result of the ingestion.
2. Transformation: Data is cleaned
	- Examples of common transformations inclide:
		- Remove duplicates
		- Convert timestamps
		- Standardize formats
		- Join datasets
		- Filter invalid records
		- Aggregate metrics
	- This is where ETL or ELT logic usually lives.
3. Storage: Data is stored
	- Examples of storage include:
		- Parquet
		- Snowflake
		- BigQuery
		- Delta Lake
		- Iceberg
	- This is usually where analysts query the data.
4. Consumption: Data is used
	- Examples of consumption include:
		- Dashboard
		- ML model
		- Reporting
		- Fraud detection
		- Recommendation engine
	- **A pipeline only creates value if someone consumes the output**.

### End-to-End Example
- Imagine an online retailer:
	```
	Customer Purchase
	
	↓
	
	Application
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	
	↓
	
	Power BI Dashboard
	```
	- Every arrow represents a stage in the pipeline.
- Batch Pipeline:
	```
	Orders
	
	↓
	
	Collect all day
	
	↓
	
	Nightly Spark Job
	
	↓
	
	Warehouse
	```
	- Runs on a defined schedule.
	- Simple and reliable.
- Streaming Pipeline:
	```
	Order
	
	↓
	
	Kafka
	
	↓
	
	Consumer
	
	↓
	
	Dashboard
	```
	- Runs continuously.
	- Lower latency.
	- More operational complexity.

### Key Pipeline Features
- **Data Quality:**
	- Pipelines should standardize the values of different fields in the data they ingest.
		- For example, an Age field should never be negative.
		- A Country field should behave like an enum, only accepting a list of pre-defined values.
	- Data Engineers spend a lot of time enforcing data quality in the pipelines they build.
	- It's important to filter out bad data before it's stored. Garbage in means garbage out.
- **Idempotency:**
	- If a data pipeline fails halfway through and needs to be retried, should data be duplicated?
		- No. A good data pipeline is idempotent. This means running it twice **produces the same results**.
		- This is one reason unique keys and deduplication are so important.
- **Monitoring:**
	- Imagine this pipeline:
		```
		Kafka
		
		↓
		
		Spark
		
		↓
		
		Warehouse
		
		↓
		
		Dashboard
		```
		- If spark fails, on-call engineers should be notified immediately.
	- Modern data pipelines monitor:
		- failures
		- latency
		- throughput
		- data freshness
		- missing data
		- duplicate data
	- Monitoring data is just as important as processing it.
- **Retry Strategy:**
	- Suppose an API **temporarily** fails due to a `503 Error`. Should we immediately fail?
		- Usually not. Instead, failed requests / jobs are retried, typically with exponential backoff and jitter to prevent cascading failure.
- **Failure Isolation:**
	- Suppose one downstream dashboard fails. Should the ingestion pipeline stop?
		- Usually not. Good data pipelines isolate failures. For example, suppose data is being streamed into a storage layer, then consumed by a dashboard and ML training model.
		- If the dashboard fails, the ML training model can still continue to run.
		- This prevents one consumer from taking down the whole ecosystem.
- **Real-World Example:**
	- Suppose there is a pipeline that processes cache warming events:
		```
		Application
		
		↓
		
		Event
		
		↓
		
		Kafka
		
		↓
		
		Analytics
		
		↓
		
		Dashboard
		
		↓
		
		Alerts
		```
		- Monitoring could detect:
			- Spike in failures.
			- Increased latency.
			- Region-specific issues.
			- Cache miss trends.

### Tradeoffs
- Well-designed data pipelines aim to balance:
	- Reliability
	- Scalability
	- Cost
	- Freshness
	- Simplicity
	- Maintainability
- Improving one often impacts another. For example, very low latency usually means:
	- Higher infrastructure cost
	- More operational complexity
- A good data pipeline is:
	- Reliable
	- Scalable
	- Observable
	- Recoverable
	- Idempotent
	- Easy to maintain

### Mental Model
Think of a pipeline as answering four questions:
1. **Where does the data come from?**
2. **How is it transformed?**
3. **Where is it stored?**
4. **Who uses it?**

### Common Interview Questions
1. What would you do if your pipeline started producing duplicate records?
> 	I'd first determine where the duplicates are being introduced—during ingestion, retries, or transformations. Then I'd implement or verify idempotent processing using unique identifiers or deduplication logic, while also monitoring the pipeline to ensure the issue doesn't recur.
2. What would you monitor in a production pipeline?
	- **Availability:** Is the pipeline running?
	- **Latency:** Is it taking longer than expected?
	- **Throughput:** Are we processing the expected number of records?
	- **Data quality:** Are there nulls, duplicates, or invalid values?
	- **Freshness:** Is new data arriving on time?
	- **Failures:** Are retries or errors increasing?
3. A nightly pipeline loads customer transactions into a data warehouse. One night, the job fails halfway through after loading **500,000 of 1,000,000** records. The scheduler automatically retries the job. How would you design the pipeline to avoid ending up with **1.5 million records** after the retry?
> 	I would design the pipeline to be idempotent by using upsert logic with the appropriate primary keys to ensure that duplicate data isn't loaded into the warehouse if the same batch of data is processed more than once. I'd also make sure the retry starts from a well-defined checkpoint or processes the entire batch idempotently so partial failures don't result in inconsistent data.
4. Your team says: "Our pipeline is green every day, so monitoring isn't important." Do you agree? What additional metrics or signals would you monitor besides whether the job succeeded or failed?
> 	No. Monitoring is always important. A job could look like its succeeding at the surface level, but could potentially be allowing corrupted data to flow through the pipeline. In addition to job success, I'd monitor data freshness, throughput, latency, and data quality metrics such as null rates, duplicate records, and schema validation failures. That helps detect situations where the pipeline completes successfully but still produces incorrect or incomplete results.
5. What's more important: making a pipeline reliable or making it fast?
> 	It depends on the business requirements, but I'd generally prioritize correctness and reliability first. A fast pipeline that produces incorrect data is often more harmful than a slower pipeline that consistently produces accurate results. Once the pipeline is correct and reliable, I would optimize performance where it provides business value.

# Apache Spark

## 4. Spark Architecture

- **The Problem Spark Solves:**
	- Imagine you have 5 TB of customer transactions.
	- Your laptop can't handle that kind of volume by itself.
	- Even if it could fit on the disk:
		- Not enough RAM.
		- Too slow.
		- Only one CPU.
	- Spark asks: "Why not use hundreds of machines?"
- **Single Machine:**
	- Traditional processing looks like:
		```
		Data
		
		↓
		
		One Computer
		
		↓
		
		Results
		```
		- Everything happens on one machine. Eventually:
			- CPU bottleneck
			- Memory bottleneck
			- Disk bottleneck
- **Spark:**
	- Distributes work across multiple machines.
	- Each processes part of the data simultaneously.
	- This is called **parallel processing**.

### Spark Cluster
- A Spark cluster has several components:
	```
	          Driver
	
	             |
	
	    -------------------
	
	    |        |        |
	
	Executor Executor Executor
	```
- The **Driver** is the brain.
	- It does **not** process all of the data. Instead, it:
		- Creates the execution plan
		- Divides work into tasks
		- Schedules those tasks
		- Coordinates execution
		- Collects results
	- The Driver tells everyone what to do.
- The **Executors** are the workers.
	- They:
		- Read data
		- Perform transformations
		- Execute tasks
		- Store intermediate results
		- Return results to the Driver
	- Each Executor typically has:
		- CPU cores
		- Memory
		- Local storage
	- Unlike the Driver, Executors perform the heavy lifting.
	- **Analogy:**
		- Imagine building a house:
			- The architect (Driver):
				- Creates the blueprint.
				- Assigns the work.
			- The construction workers (Executors):
				- Pour concrete.
				- Install plumbing.
				- Build walls.
- **Partitions:**
	- Suppose you have a machine with 100 GB of data.
	- Spark doesn't send the entire payload to one Executor.
	- Instead, it divides the payload into 10 partitions and sends one partion to each Executor.
	- This allows all Executors to work in parallel, processing the data much more quickly than a single machine.
	- **This is why partitioning is so important**.
- **Tasks:**
	- Each partition becomes a task.
	- Tasks are assigned to each Executor.
	- Spark keeps assigning work until all tasks finish.
- **Cluster Management:**
	- The Cluster Manager is responsible for managing the machines (Executors).
	- Examples:
		- Kubernetes
		- YARN
		- Standalone Spark
	- Architecture:
		```
		Driver
		
		↓
		
		Cluster Manager
		
		↓
		
		Executors
		```
		- The Driver **requests** resources.
		- The Cluster Manager **provides** resources.
- What happens when you run:
	```python
	df.filter(df.country == "USA")
	```
	- The Driver creates the plan. Then:
		```
		Driver
		
		↓
		
		Execution Plan
		
		↓
		
		Tasks
		
		↓
		
		Executors
		
		↓
		
		Results
		```
- **Fault Tolerance:**
	- Suppose Exector 2 crashes while performing a task. Does the job fail?
		- Usually not. Spark knows which partition the Executor was processing.
		- It simply reschedules the task on another Executor.
		- This is one reason Spark is resilient.
- **Why Spark Is Faster Than MapReduce:**
	- Older systems like Hadoop MapReduce wrote intermediate results to disk after nearly every processing stage:
		```
		Read
		
		↓
		
		Map
		
		↓
		
		Write to Disk
		
		↓
		
		Read Again
		
		↓
		
		Reduce
		
		↓
		
		Write Again
		```
	- Spark keeps much of its intermediate data **in memory**, greatly reducing disk I/O for many workloads.
	- That doesn't mean Spark _never_ uses disk—it absolutely does when needed—but minimizing unnecessary disk access is a major performance advantage.
- **Real-World Example:**
	- Imagine Amazon needs to process 500 million purchases. These purchases are stored across 500 partitions.
	- Spark creates 500 tasks, running on 50 Executors. Each Executor processes about 10 tasks.
	- Instead of waiting for one machine to process all 500 million records, dozens of machines work in parallel.
- **Data Engineering Example:**
	- Suppose you're analyzing cache warming events:
		- Logs are processed using Spark. Failed events are filtered.
		- Remaining events are grouped by region.
		- The failure rate is calculated.
	- Spark might split the log data into 100s of partitions. Each executor would analyze a subset of the data.
	- Finally, the Driver would combine the partial results into the final output.
- **Common Misconception:**
	- People often think the Driver processes the data.
	- The Driver **coordinates** tasks. The Executors process the data.
	- The Driver only handles a relatively small amount of metadata and coordination compared to the data processing done by the executors.

### Tradeoffs (More Executors)
- Pros:
	- More parallelism
	- Faster processing
	- Better throughput
- Cons:
	- More resource usage
	- More scheduling overhead
	- More network communication
- More isn't always better. There's an optimal level of parallelism.

### Metal Model
This is flow that every Spark job follows:
```
Application Code

↓

Driver

↓

Execution Plan

↓

Tasks

↓

Partitions

↓

Executors

↓

Results
```

### Common Interview Questions
1. Your Spark job is processing **1 TB** of Parquet data. One executor crashes halfway through processing its assigned partition. What happens next? Why doesn't Spark have to restart the entire job?
> 	Spark doesn't have to restart the entire job because the driver broke down the 1 TB of data into an appropriate number of partitions. Each partition is assigned to a task and tasks are distributed evenly among executors. Spark recognizes when an executor has failed to execute a task and reassigns the failed task to another executor. Because the input data still exists (for example, in S3 or HDFS) and the Driver knows which transformations need to be applied, Spark can simply rerun the failed task on another executor rather than restarting the entire application.
2. A teammate says: "Let's solve our slow Spark job by adding twice as many executors." Would you agree? What factors would you consider before simply increasing the number of executors?
> 	Adding more executors might solve the issue, but probably wouldn't be an appropriate first step. Metrics such CPU Utilization, Memory Utilization, and Latency should be analyzed to determine if the jobs are running efficiently. If they aren't potential code optimizations should be explored before adding more executors.
	- Besides CPU and Memory Utilization, these metrics should also be considered:
		- **Data skew** – Is one executor processing far more data than the others?
		- **Shuffle volume** – Are we moving huge amounts of data across the network?
		- **Partition count** – Do we have enough work to keep all executors busy?
		- **Disk spill** – Are executors running out of memory and spilling intermediate data to disk?
		- **Input size** – Is the dataset itself growing?
3. If I have 8 partitions and 100 executors, will my Spark job be 100x faster?
	- No. Since the data is only split into 8 partitions, Spark only has 8 tasks to work with. At most, it could assign 1 task to each executor, leaving 92 executors sitting idle.
	- **You need enough partitions to create parallel work**.
		- You could divide the data into 100 partitions, but this may not be efficient depending on how large the overall data set is. Adding partitions (tasks) also adds scheduling overhead for the Driver.
		- Spark performance is often about finding the **right balance**.

## 5. RDDs vs. DataFrames vs. Datasets

- The Spark code below uses a **DataFrame** to process data:
	```python
	df.groupBy("country").count()
	```
	- Spark didn't orginally have DataFrames. It started with something called RDDs.
	- Over time, Spark has evolved through three major data abstractions:
		```
		RDD
		
		↓
		
		DataFrame
		
		↓
		
		Dataset
		```
		- Each abstraction was created to solve limitations imposed by the previous one.
- **Resilient Distributed Dataset (RDD):**
	- RDDs were Spark's original programming model.
	- An RDD is simply: A distributed collection of objects partitioned across a cluster.
	- RDDs are considered "resilient" because they keep track of they were created. If an Executor crashes while processing an RDD, Spark can recompute the lost partition.
	- Advantages: RDDs are:
		- Flexible
		- Low-level
		- Good for custom algorithms
		- Fine-grained control
	- Disadvantages:
		- RDDs don't understand the structure of your data.
		- RDDs don't distinguish between object types. They see all fields in each record as a generic object.
		- This limits Spark's ability to optimize your code.
- **DataFrame:**
	- Unlike RDDs, DataFrames allow Spark to recognize the schema of a dataset.
	- With DataFrames, Spark can understand:
		- Data types
		- Column names
		- Relationships
	- Suppose you write:
		```python
		df.select("Country")
		```
		- Spark immediately knows only one column is needed.
- **Catalyst Optimizer:**
	- When using DataFrames, Spark doesn't immediately execute your code. Instead:
		```
		Your Code
		
		↓
		
		Logical Plan
		
		↓
		
		Catalyst Optimizer
		
		↓
		
		Physical Plan
		
		↓
		
		Execution
		```
		- Catalyst rewrites your query into a more efficient version.
		- This is why DataFrames often outperform equivalent RDD code.
	- **Example:**
		- If you write:
			```python
			df.filter(df.country=="USA").select("Sales")
			```
			- Spark only reads `Country` and `Sales`. It ignores all other columns.
			- The Catalyst Optimizer performs these optimizations automatically.
- **Datasets:**
	- Combine RDD flexibility with DataFrame optimization.
	- Datasets provide:
		- Compile-time type safety (in Scala/Java)
		- Catalyst optimization
		- Structured data
	- For example. Consider generic rows in a `Customer` table. Datasets allow Spark to know:
		- `Customer.name`
		- `Customer.country`
		- `Customer.sales`
	- Errors can be caught at compile time.
	- Datasets aren't commonly used in Python because they **require compile-time type information**.
	- Python is **dynamically typed**. 
	- PySpark therefore primarily exposes:
		- RDDs
		- DataFrames
	- Not typed Datasets.

| Feature                  | RDD | DataFrame | Dataset                      |
| ------------------------ | --- | --------- | ---------------------------- |
| Structured               | ❌   | ✅         | ✅                            |
| Schema aware             | ❌   | ✅         | ✅                            |
| Catalyst optimization    | ❌   | ✅         | ✅                            |
| Compile-time type safety | ❌   | ❌         | ✅ (Scala/Java)               |
| Python support           | ✅   | ✅         | Limited/No typed Dataset API |
- **Real-World Example:**
	- Imagine 500 million purchase events stored as Parquet.
	- You need to perform the following query:
		```sql
		SELECT Region,
		AVG(Duration)
		```
		- With a DataFrame, Spark only reads:
			- Region
			- Duration
		- With an RDD, Spark reads the entire object, which is much slower.
- Early Spark developers wrote code like:
	```python
	rdd.map(...)
	   .filter(...)
	   .reduce(...)
	```
- Modern Spark developers write code like:
	```python
	df.filter(...)
	  .groupBy(...)
	  .agg(...)
	```
	- This is because DataFrames allow Spark to optimize its execution plan automatically.
	- This results in:
		- Less code
		- Better performance
		- SQL support
		- Easier maintenance
- **Real Data Engineering Pipeline:**
	```
	Kafka
	
	↓
	
	Spark DataFrame
	
	↓
	
	Transform
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	```
	- Nearly every modern Spark pipeline is built around DataFrames.
	- RDDs still exist, but they're used far less often.

### Tradeoffs
- RDD
	- Pros:
		- Maximum flexibility
		- Low-level control
		- Supports custom algorithms
	- Cons:
		- No optimization
		- Slower for structured data
		- More code
- DataFrame:
	- Pros:
		- Catalyst optimization
		- Schema awareness
		- Better performance
		- SQL integration
		- Easier to read
	- Cons:
		- Less low-level control
		- Some specialized algorithms are harder to express
- Dataset:
	- Pros:
		- Type safety
		- Optimization
		- Structured API
	- Cons:
		- Primarily Scala/Java
		- Rarely used in PySpark

### Mental Models
```
RDD

↓

"I know how to process data."

----------------

DataFrame

↓

"I know what the data looks like."

----------------

Dataset

↓

"I know what the data looks like,
and the compiler verifies my types."
```
- Schema awareness is the key difference between an RDD and a DataFrame / Dataset.
```
RDD

↓

Spark can distribute work.

------------------------

DataFrame

↓

Spark understands the schema.

------------------------

Catalyst

↓

Spark can optimize the execution plan.

------------------------

Parquet

↓

Spark can avoid reading unnecessary data.
```
- RDDs allow Spark to distribute the work.
- DataFrames allow Spark to understand the schema and optimize the execution plan.
- Parquet allows Spark to avoid reading unnecessary data.

### Common Interview Questions
1. Why are DataFrames usually preferred over RDDs?
> 	DataFrames provide schema awareness, allowing Spark's Catalyst Optimizer to generate more efficient execution plans. They also integrate well with columnar storage formats like Parquet, enabling optimizations such as reading only required columns. As a result, DataFrames are typically easier to write, easier to maintain, and perform better than equivalent RDD-based implementations for structured data.
2. You're building a Spark pipeline that processes **billions of structured customer records stored in Parquet**. Would you choose an RDD or a DataFrame?
> 	I would choose a DataFrame because Spark understands the schema and can use the Catalyst Optimizer to generate a more efficient execution plan. Combined with Parquet's columnar storage, Spark can read only the required columns instead of scanning every field, reducing disk I/O and improving performance.
3. A teammate says: "RDDs give us more control, so we should always use them instead of DataFrames." Do you agree? What tradeoffs would you discuss before making that decision?
> 	RDDs should only be used if the lower-level control and flexibility they offer is more important than the schema awareness and optimization provided by DataFrames. For the vast majority of ETL and analytics pipelines, I'd choose DataFrames because they allow Spark to optimize execution automatically. I'd only reach for RDDs if I needed very fine-grained control over distributed processing or was implementing a custom algorithm that doesn't fit naturally into Spark's structured APIs.
	- RDDs today are used mainly for:
		- Fine-grained control over distributed processing.
		- Implementing custom algorithms that are not well suited for Spark APIs.
4. If DataFrames are optimized automatically, does that mean performance tuning is no longer important?
	- Absolutely not. Catalyst can optimize a lot, but it can't fix everything.
	- For example:
		- A poor partitioning strategy
		- Data skew
		- Inefficient joins
		- Too many small files
		- Excessive shuffling
	- Catalyst helps. It doesn't replace good engineering decisions.

## 6. Transformations vs. Actions

- Spark operations fall into two main categories: Transformations and Actions
- **Transformations:**
	- A transformation **describes** work.
	- Examples:
		```python
		df.filter(...)
		df.select(...)
		df.groupBy(...)
		df.join(...)
		```
		- These all **create** a new DataFrame.
		- They don't produce an answer, they produce a **plan**.
		- For example, `df.filter(...)` creates a new DataFrame that **represents** a filtered dataset. The filtering occurs later when the action is executed.
	- Suppose you write:
		```python
		orders = spark.read.parquet("orders")
		
		usa_orders = orders.filter(col("country") == "USA")
		
		sales = usa_orders.select("sales")
		```
		- Spark doesn't actually read the parquet file. It generates an execution plan:
			```
			Read Orders
			
			↓
			
			Filter Country
			
			↓
			
			Select Sales
			```
- **Actions:**
	- An action requires a result and actually **performs** work.
	- Examples:
		```python
		show()
		count()
		collect()
		write.parquet(...)
		save()
		```
		- Transformations cause Spark to develop an execution plan, but not actually execute the plan.
		- Actions like the ones shown above cause spark to execute the plan so it can return a result.
	- Imagine this code:
		```python
		df.filter(col("country") == "USA") \
		  .select("sales") \
		  .groupBy() \
		  .sum()
		```
		- If Spark executed after every line, that would be very inefficient.
		- Instead, Spark waits, builds one complete plan, optimizes it, then runs it once.
		- At the `filter(...)` stage, Spark doesn't know what you want to do with the filtered results, so it waits.
		- It executes the plan when you call `sum()`. At this point, it knows the full workload, so it execute the plan efficiently.
- **Building The Logical Plan:**
	- Imagine writing:
		```python
		orders \
		.filter(...) \
		.select(...) \
		.groupBy(...) \
		.sum()
		```
	- Spark creates:
		```
		Logical Plan
		
		Read
		
		↓
		
		Filter
		
		↓
		
		Select
		
		↓
		
		Group By
		
		↓
		
		Aggregate
		```
- **Catalyst Optimizer:** Once the plan is built, the Catalyst Optimizer examines it. Maybe it realizes certain columns are not needed, so it modifies the plan.
	- It modifies the plan to read only the **required** columns. Then it filters, groups, and aggregates.
	- This is much more efficient.
- **Physical Plan:**
	- After optimization, Spark creates the physical plan. This answers questions such as:
		- Which executors?
		- Which partitions?
		- Which joins?
		- Which tasks?

### Real Examples
- Imagine:
	```python
	orders \
	.filter(col("country")=="USA") \
	.select("sales") \
	.count()
	```
- Spark might internally decide:
	```
	Read only:
	
	Country
	
	Sales
	```
- Instead of:
	```
	CustomerID
	
	Phone
	
	Email
	
	Address
	
	Country
	
	Sales
	
	Timestamp
	
	...
	```
	- This is where Parquet and the Catalysy Optimizer work together.
- Now imagine you accidentally write:
	```python
	df.show()
	
	df.count()
	
	df.write.parquet(...)
	```
	- This results in **three** separate jobs. Each **action** triggers a separate execution.
	- Suppose you write this instead:
		```python
		df.filter(...)
		
		.select(...)
		
		.write.parquet(...)
		```
	- Only one action is performed (`write.parquet(...)`), so there's only one job.
	- This is much more efficient.

### Data Engineering Example
- Imagine the following pipeline:
	```python
	logs \
	.filter(failed) \
	.select("Region","Duration") \
	.groupBy("Region") \
	.avg()
	.write.parquet(...)
	```
	- Spark executes the **entire pipeline** at `write.parquet(...)`.
	- Lazy evaluation allows Spark to ignore columns that are never used.
	- RDDs can't optimize this nearly as well.
- **Real-World Analogy:**
	- Imagine ordering food at a restaurant. You tell the waiter:
		- Appitzer
		- Main course
		- Desert
	- The waiter doesn't run the kitchen after every sentence. They wait until you've finished ordering.
	- The kitchen then prepares everything efficiently.
	- Spark works similarly. Transformations build the order. Actions send the order to the kitchen.

### Tradeoffs (Lazy Evaluation)
- Pros:
	- Better optimization
	- Fewer jobs
	- Less disk I/O
	- Better execution planning
	- Reduced network traffic
- Cons:
	- Errors may not appear until an action is executed
	- It can initially be confusing because writing code doesn't immediately do anything

### Mental Model
```
Transformation

↓

Transformation

↓

Transformation

↓

Logical Plan

↓

Catalyst Optimization

↓

Physical Plan

↓

Action

↓

Execution
```

### Common Interview Questions
1. Why does Spark use lazy evaluation?
> 	Spark delays execution so it can build a complete logical plan before processing any data. This allows the Catalyst Optimizer to combine transformations, eliminate unnecessary work, and generate a more efficient physical execution plan, reducing disk I/O, network communication, and overall execution time.
2. Consider this code:
	```python
	df = spark.read.parquet("orders")
	
	usa = df.filter(col("country") == "USA")
	
	sales = usa.select("sales")
	```
	- Has spark read the paquet file yet?
> 	Spark hasn't read the Parquet file yet because `filter` and `select` are transformations. Instead of executing immediately, Spark builds a logical plan. When an action such as `count()` is called, Catalyst optimizes that plan, creates a physical execution plan, and then the executors read the Parquet data.
1. A teammate write:
	```python
	df.show()
	
	df.count()
	
	df.write.parquet("output")
	```
	- Why might this be less efficient than expected? How would you explain what's happening internally?
> 	`show()`, `count()`, and `write()` are all actions. Each action triggers a separate Spark job, causing Spark to execute the entire lineage up to that point three separate times. Depending on the complexity of the transformations, this could mean repeatedly scanning the input data, recomputing transformations, and consuming additional cluster resources.
1. If I really do need to call both `count()` and `write()`, is Spark forced to recompute everything twice?
	- Not necessarily. If the intermediate DataFrame is expensive to compute and will be reused, you can persist it:
		```python
		df = expensive_transformations.cache()
		
		df.count()
		
		df.write.parquet(...)
		```
		- Now Spark can reuse the cached data instead of recomputing the entire transformation pipeline.
1. Imagine this code:
	```python
	df.filter(col("country") == "USA") \
	  .select("sales") \
	  .filter(col("sales") > 100)
	```
	- Will Spark perform three separate filter operations, exactly as written? Or will the Catalyst Optimizer reduce the workload?
> 	Not necessarily. Since these are transformations, Spark first builds a logical plan. The Catalyst Optimizer can then rewrite that plan before execution. For example, it can perform column pruning so that only the `country` and `sales` columns are read, and it can push filter predicates as close to the data source as possible to reduce the amount of data processed. The final physical execution plan may be quite different from the order in which I wrote the transformations.
	- `select()` determines which **columns** are needed. **It doesn't read the entire column into memory**.
	- `filter()` determines which **rows** are needed.
	- Spark combines both transformations before executing a plan.
	- Spark can optimize the plan by using column pruning and predicate pushdown to reduce the amount of that that is scanned. With predicate pushdown, Spark tries to push filters **as close to the data source as possible**.
	- This is why Parquet format is such a preferred storage format. It allows for both of these optimizations because it stores data in columnar format.

## 7. Lazy Evaluation

- Lazy Evaluation means: Spark delays executing transformations until an action requires a result.
- Instead of executing each line of code immediately, Spark first builds a complete plan of what you want to accomplish.
- **Eager vs. Lazy:**
	- In Python:
		```python
		numbers = [1, 2, 3, 4]
		
		filtered = [x for x in numbers if x > 2]
		```
		- Filtering happens immediately. These means Python is **eager**.
	- In Spark:
		```python
		filtered = df.filter(col("sales") > 100)
		```
		- Nothing has happened yet. Spark has simply initiated building an execution plan, which will be executed when an action is required.
		- **No data has been processed yet**.
		- When an action, such as `filtered.show()`, is called Spark begins:
			- Optimizing
			- Creating tasks
			- Scheduling executors
			- Reading files
			- Processing partitions
		- If Spark executed each transformation immediately, it would result in a lot unnecessary work, such as repeatedly scanning data.
	- Example:
		```python
		orders = spark.read.parquet("orders")
		
		usa = orders.filter(col("country") == "USA")
		
		sales = usa.select("sales")
		
		result = sales.groupBy().avg()
		```
		- With the above code, Spark only begins creating a logical execution plan.
		- Once an action, such as `result.show()`, is called, Spark begins optimizing, creating a phyical plan, and executing that plan to produce results.
- Lazy Evaluation enables optimization because Spark is able to see the *entire* pipeline before executing. This allows it to answer questions such as:
	- Which columns are actually needed?
	- Can any filters be applied earlier?
	- Can operations be combined?
	- Which join strategy is best?
	- How many tasks should be created?
- If Spark executed each line immediately, many of these optimizations wouldn't be possible.
- **Multiple Transformations, One Job:**
	```python
	df.filter(...)
	  .select(...)
	  .groupBy(...)
	  .agg(...)
	  .write.parquet(...)
	```
	- The only action being performed in this pipeline is `.write()`.
	- Everything that comes before it is just building the execution plan.
- **Multiple Actions:**
	```python
	df.filter(...)
	
	df.count() # action
	
	df.show() # action
	
	df.write.parquet(...) # action
	```
	- This creates three jobs, one for each action.
- **Lineage:**
	- Suppose your pipeline consists of:
		```
		Read Orders
		
		↓
		
		Filter USA
		
		↓
		
		Select Sales
		
		↓
		
		Aggregate
		```
		- Spark memorizes the **sequence**, not just the final result.
		- This is how it remembers how to produce the result. It has a roadmap saved. This is lineage.
	- Lineage is what allows spark to only run a portion of a job after it crashes, instead of re-running the full job.
- **Data Engineering Example:**
	- Suppose your pipeline consists of:
		```python
		logs \
		.filter(col("status") == "FAILED") \
		.select("region", "duration") \
		.groupBy("region") \
		.avg("duration")
		.write.parquet(...)
		```
		- Spark waits until the `write()` action to execute the plan. Everything happens in one optimized workflow.

### Tradeoffs
- Advantages:
	- Better optimization
	- Fewer scans of the data
	- Reduced disk I/O
	- Better execution planning
	- Lower network traffic
	- Ability to optimize the entire workflow instead of individual steps
- Disadvantages:
	- Makes debugging more confusing.
	- Imagine running `df = bad_transformation()`, then `df.show()`. You won't know a problem occurred until the action is executed.

### Mental Model
```
Write Transformations

↓

Logical Plan

↓

Lazy Evaluation

↓

Catalyst Optimization

↓

Physical Plan

↓

Action

↓

Execution
```

### Common Interview Questions
1. Why is lazy evaluation one of Spark's biggest performance optimizations?
> 	Lazy evaluation allows Spark to delay execution until it has the complete sequence of transformations. This lets the Catalyst Optimizer analyze the entire workflow, eliminate unnecessary work, combine operations where possible, and generate an efficient physical execution plan. As a result, Spark minimizes data movement, disk I/O, and redundant computation.
2. Suppose you write:
	```python
	df = spark.read.parquet("orders")
	
	filtered = df.filter(col("country") == "USA")
	
	selected = filtered.select("sales")
	```
	- Your program abruptly exits. What work, if any, has Spark actually performed? Why?
> 	Spark has not processed the data yet. Spark has only started to build a logical plan based on the transformations being applied. Once an action, such as show() or writing the data to parquet is performed, it will optimize the logical plan and execute the resulting physical plan.
	- The `read()` command technically does some work. It creates a DataFrame representing the source. It does **not** scan the Parquet file until an action is performed.
1. You're reviewing a teammate's code:
	```python
	daily_sales = load_sales()
	
	daily_sales.count()
	
	daily_sales.show()
	
	daily_sales.write.parquet("output")
	```
	- Assume `load_sales()` performs several expensive joins and aggregations. What performance issue do you see? How would you improve it?
> 	Each action, such as show() and write() cause Spark to recompute the expensive joins and aggregations in separate jobs. To improve performance, I would cache the result of load_sales(), so Spark can reuse the result in subsequent actions. I would also consider eliminating unnecessary actions, such as show().
	- Note: Simply method chaining does **not** allow Spark to optimize three jobs into one job. Each action, regardless of how the code is written, must produce a result. Spark creates separate jobs to produce these results.
	- The only ways to avoid recomputing expensive transformations are to:
		- Cache/persist intermediate results.
		- Eliminate unnecessary actions.
		- Restructure the workflow so you only need one final result.
	- Optimized Code:
		```python
		daily_sales = load_sales().cache()
		
		daily_sales.count()
		
		daily_sales.show()
		
		daily_sales.write.parquet("output")
		```
1. If `count()`, `show()`, and `write()` are all required by the business, can Spark execute them as one job?
	- No. Each action requests a different output:
		- `count()` → a single number.
		- `show()` → rows displayed to the console.
		- `write()` → files written to storage.
	- These are fundamentally different outcomes, so Spark executes separate jobs.
	- The optimization is to **reuse the computed DataFrame**, not to merge the actions.
## 8. Narrow vs. Wide Transformations

- Not all Spark transformations are created equal. Some operations can be performed **independently** on each partition. Others require Spark to move data across the cluster.
- This distinction gives us:
	- Narrow transformations
	- Wide transformations
- This is one of the biggest factors affecting Spark performance.
- **Narrow Transformations:**
	- A narrow transformations means: Each output partition depends on only one input partition. **No data needs to move between executors**.
	- Example:
		```python
		df.filter(col("sales") > 100)
		```
		- Filtering occurs independently in each partition. No communication required. No network traffic. Fast.
		- Other narrow transformations include:
			```python
			select()
			filter()
			withColumn()
			map()
			flatMap()
			```
		- Each executor only needs its own partition to perform its assigned tasks.
- **Wide Transformations (Shuffling):**
	- A wide transformation means: An output partition depends on data from multiple input partitions. **This requires Spark to redistribute data across the cluster**.
		- This redistribution is called a **shuffle**.
	- Example:
		```python
		df.groupBy("country").count()
		```
		- Suppose you're working with sales data. If sales data for `USA` is spread across more than one partition, a single executor cannot perform the aggregation by itself.
		- Spark must first gather all `USA` rows together. This is where shuffling is performed.
	- **Shuffling**
		- Before a shuffle, `USA` data is spread across multiple executors. After a shuffle, `USA` data exists in a single executor.
		- Now each country is together, so Spark can aggregate.
		- Shuffling is expensive because Spark may need to:
			- Serialize data
			- Send it across the network
			- Write intermediate data
			- Read intermediate data
			- Synchronize tasks
		- Compared to a simple filter, that's a lot of work.
		- Common Wide Transformations:
			```python
			groupBy()
			join()
			distinct()
			orderBy()
			repartition()
			```
	- Filtering is a lot cheaper than shuffling because each executor asks, "Which of *my* rows match?" No other executors need to be involved. That's why it's considered narrow.
	- Grouping is expensive because Spark may need to shuffle data so that it exists in a single executor, before that executor performs any work.

### Visualizing Transformations
- Narrow:
	```
	Partition A
	
	↓
	
	Partition A
	
	--------------
	
	Partition B
	
	↓
	
	Partition B
	```
	- One-to-one. Inexpensive.
- Wide:
	```
	Partition A
	
	↘
	
	     New Partition
	
	↗
	
	Partition B
	```
	- Many-to-one. Expensive (due to shuffling requirements).

### Data Skew
- Suppose 95% of sales data belongs to `USA`. Only 5% belongs to other countries.
- After shuffling data to prepare for an aggregation, only one executor is doing 95% of the work. The other executors finish their work quickly and sit quietly while the first executor finishes its work.
- This is called **data skew** and is one of the primary causes of slow Spark jobs.
- Optimizing Spark jobs isn't about eliminating shuffling all together, it's about eliminating **unnecessary** shuffling.

### Tradeoffs
- Narrow Transformations:
	- Pros:
		- No network communication
		- Faster execution
		- Better scalability
		- Less disk I/O
		- Lower memory pressure
	- Cons:
		- Limited to operations that don't require combining data across partitions
- Wide Transformations:
	- Pros:
		- Enable powerful operations like joins and aggregations
		- Necessary for many analytical workloads
	- Cons:
		- Network traffic
		- Serialization overhead
		- Disk spill if memory is insufficient
		- Synchronization delays
		- Vulnerable to data skew

### Mental Model
```
Narrow

Partition A

↓

Partition A

--------------

Wide

Partition A

↘

New Partition

↗

Partition B
```
- Spark Optimization Principle
	```
	Filter Early
	
	↓
	
	Reduce Data Volume
	
	↓
	
	Shuffle Less Data
	
	↓
	
	Faster Job
	```
	- You'll often hear experienced engineers say, "Push filters before joins." Or "Reduce the dataset before aggregating."

### Common Interview Questions
1. Why is `groupBy()` usually more expensive than `filter()`?
> 	`filter()` is a narrow transformation because each partition can be processed independently. `groupBy()` is a wide transformation because rows with the same grouping key may exist in multiple partitions. Spark must perform a shuffle to bring those rows together before it can aggregate them, introducing network communication and additional processing overhead.
2. You're processing a 5 TB Parquet dataset. Which operation would you expect to be more expensive, and why?
	- `df.filter(col("status") == "FAILED")` or `df.groupBy("status").count()`.
> 	Grouping is more expensive if the data for multiple status types is spread across multiple executors. If this is the case, Spark redistributes rows so that all records for the same grouping key end up in the same partition. Filtering can be done independently across many executors.
	- The goal isn't to assign data to an executor, it's to assign work to a **partition**. The Driver node creates one task for each partition and assigns executors one or more tasks to evenly distribute the work.
3. A teammate says: "Our Spark job is slow because of shuffling. Let's just eliminate every shuffle." How would you respond? When are shuffles unavoidable, and when should you try to reduce them?
> 	I wouldn't try to eliminate every shuffle because some operations, such as `groupBy()`, `join()`, and `orderBy()`, require Spark to redistribute data across partitions to produce correct results. Instead, I'd focus on eliminating unnecessary shuffles and making the required ones more efficient—for example, by filtering data earlier, choosing appropriate partitioning strategies, or using broadcast joins when appropriate.
	- A shuffle happens because the **operation requires data movement**. Even if data is perfectly balanced, Spark **still has to shuffle** because rows for each country may be spread across multiple partitions.
	- Data skew happens **after** shuffling. It occurs when one executor ends up being assigned a majority of the work because one partition is significantly larger than the others.
	- Shuffling isn't caused by data skew. The shuffling becomes **expensive because of skew**.
	- When trying to optimize Spark code, you'll typically filter **before** grouping to lower the amount of data that needs to grouped (potentially shuffled). You won't necessarily **replace** grouping with filtering.
4. Suppose you have `customers.join(orders, "customer_id")`. Why is this usually considered a **wide transformation**, even if both DataFrames are already partitioned?
> 	A join is typically a wide transformation because Spark must ensure that all rows with the same join key are located in the same partition before the join can be performed. Even if both DataFrames are already partitioned, they may not be partitioned on the join key or use the same partitioning scheme. Spark often needs to shuffle the data to correctly align matching records.
	- The emphasis here is on **join key**. If you're joining Customers and Orders on `customer_id`, but the two tables aren't both partitioned using that key, Spark will need to shuffle the data so that all data **needed to perform the join** exists in one partition.
	- Spark can't correctly perform the join until it **repartitions both DataFrames by the join key** so that all rows with the same join key are colocated.
	- "Already partitioned" doesn't mean partitioned **correctly**.
5. Can Spark ever perform a join without a shuffle?
	- Yes. Examples include:
		- **Broadcast joins**, where a small table is sent to every executor instead of shuffling the large table.
		- Cases where both DataFrames are already partitioned on the same join key with compatible partitioning, allowing Spark to avoid repartitioning.
		- Broadcast joins are one of the most common Spark optimizations and are the next logical topic after shuffles.

## 9. Shuffling

- In Spark, a shuffle is the process of redistributing data across **partitions** so that related records end up together for the next stage of computation
	- Spark will only perform shuffling **when it is required for computation**.
- Shuffling occurs in two stages:
	1. **Shuffle Write:**
		- Each executor processes its current partition.
		- Spark hashes the partition key to determine which partition each row belongs to in the next stage.
		- The rows are **written** into temporary shuffle files, grouped by their destination partition.
	2. **Shuffle Read:**
		- In this stage, every executor fetches the data it needs.
		- Each executor pulls data from many other executors **over the network**.
		- Only after all of the required data arrives can the next stage begin.
- Before Shuffle:
	```
	Partition 1
	
	Alice
	Bob
	
	----------------
	
	Partition 2
	
	Alice
	Carol
	```
- After Shuffle:
	```
	Partition A
	
	Alice
	Alice
	
	----------------
	
	Partition B
	
	Bob
	
	----------------
	
	Partition C
	
	Carol
	```
- Shuffling is so expensive because Spark needs to perform the following actions:
	1. Serialize data into bytes.
	2. Write it to local disk (if necessary).
	3. Send it across the network.
	4. Receive it on another executor.
	5. Deserialize it.
	6. Continue processing.
- During a shuffle, Spark spends time on:
	- Network transfer
	- Disk I/O
	- Serialization/deserialization
	- Waiting for slower executors
	- Coordination between tasks
- **This is why reducing unnecessary shuffles is one of the biggest performance optimizations**.
- Operations that trigger shuffles:
	```python
	groupBy()
	join()
	distinct()
	orderBy()
	repartition()
	```
	- These are all **wide transformations** because they require data redistribution.

- **Shuffle Partitions:**
	- Spark doesn't shuffle data randomly. It creates a new set of partitions.
	- By default, Spark uses a configurable number of shuffle partitions (often 200 in many environments, though this can vary by Spark version and configuration). For example:
		```python
		spark.conf.get("spark.sql.shuffle.partitions")
		```
		- If this returns 200, it could be an ideal number of partitions, too many, or too few partitions, depending on the size of the dataset.
		- If your workload has 20 MB of data, this is way too many partitions.
		- If your workload has 20 TB of data, this might be too few.
		- This is why **partition tuning** matters.
- **Shuffle Spill:**
	- Suppose an executor receives 150 GB of data during a shuffle, but only has 32 GB of memory.
	- Spark can't hold of the data in memory, so it spills intermediate data to the disk.
	- This is **much slower** than processing everything in memory.
- **Reducing Shuffle Costs:**
	- You can't always eliminate shuffles, but you can often reduce their cost.
	- Common strategies include:
		- Filter data before joins or aggregations.
		- Use broadcast joins when one table is small.
		- Choose appropriate partitioning keys.
		- Avoid unnecessary `repartition()` calls.
		- Reuse partitioned data when possible.
		- Use Adaptive Query Execution (AQE), which can optimize some shuffle behavior at runtime.

### Tradeoffs
- Pros:
	- Enables joins and aggregations across distributed data.
	- Makes many distributed algorithms possible.
- Cons:
	- Network traffic
	- Disk I/O
	- Serialization overhead
	- Potential spills
	- Can amplify data skew
	- Often the **most expensive phase** of a Spark job

### Mental Model
```
Broadcast Join
        │
        └── Avoid shuffle

Partitioning
        │
        └── Reduce future shuffle

Data Skew
        │
        └── Uneven shuffle

Caching
        │
        └── Avoid repeating shuffle

AQE
        │
        └── Optimize shuffle

Repartition
        │
        └── Explicit shuffle
```
- Spark optimization isn't about **avoiding** shuffles, it's about **managing** them appropriately.
- Think of a shuffle like reorganizing books in a library.
- Initially, books are scattered across shelves.
- To organize them by author, you have to:
	1. Pick up the books.
	2. Carry them to new shelves.
	3. Put them down.
	4. Continue organizing.
- That movement—not the final organization—is what costs time. Generally speaking a shuffle involves:
	```
	Serialize
	
	↓
	
	Write
	
	↓
	
	Network Transfer
	
	↓
	
	Read
	
	↓
	
	Deserialize
	```

### Common Interview Questions
1. Why are shuffles expensive?
> 	Shuffles require Spark to redistribute data across the cluster. That involves serializing data, transferring it over the network, potentially writing it to disk, and then deserializing it on another executor. Because of the network and disk I/O involved, shuffles are typically much more expensive than operations that work on local partitions.
2. A Spark job spends **80% of its execution time in shuffle read and shuffle write**. What does that tell you about the workload? What would you investigate before trying to optimize it?
> 	This could be a sign that data is spread across too many partitioned or that data is poorly partitioned (not partitioned according to typical data access patterns). To investigate, I'd look at the partition key and compare it against what the job is trying to accomplish. If the partition key doesn't optimally organize the data, I'd consider modifying the partition key. If the data is spread across too many partitions, I'd consider coalescing the data. If I found that the workload had an excessive number of very small shuffle partitions, I'd consider reducing them with `coalesce()` or adjusting the shuffle partition configuration.
	- Having a lot of partitions doesn't necessarily make a shuffle expensive. Coalescing is mainly used to reduce **scheduling overhead** when partitions are **genuinely** too small.
	- A proper investigation would look at factors such as:
		- Is there a large `groupBy()` or `join()`?
		- Could a broadcast join eliminate one shuffle?
		- Is there an unnecessary `repartition()`?
		- Can I filter data before the shuffle?
		- Is AQE selecting a good join strategy?
		- Is data skew making the shuffle more expensive?
		- Are shuffle partitions appropriately sized?
3. A teammate says: "Shuffles are slow, so we should avoid them completely." How would you respond? Are shuffles always a sign of poor Spark code, or are they sometimes unavoidable?
> 	I would advise them that shuffles can't always be avoided. I'd advise them that, when shuffles are necessary, they should be optimized. Optimization strategies can include filtering early, performing a broadcast join for relatively small tables, and caching (to avoid repeated shuffling). I'd also avoid unnecessary `repartition()` calls, since each repartition introduces another shuffle.
4. Which is generally more expensive: reading 100 GB from local disk, or shuffling 100 GB across the cluster?
	- The answer, generally speaking, is shuffling 100 GB of data across the cluster.
	- This because shuffling isn't just a read. It also invokes a serialization, a write, a network transfer, another read by the destination executor, and a deserialization.
	- This is why Spark engineers work so hard to avoid unnecessary shuffling.

## 10. Data Skew

- Data skew exists because Spark assumes work can be divided evenly among executors.
	- If you have 100 GB of data, Spark will create 10 partitions, assuming it can allocate 10 GB to each partition.
	- Sometimes, due to a poorly chosen partition key, 90% of the data ends up being assigned to one parition while the other 10% is divided eveny among the other 9 partitions.
	- In this case, one executor is doing most of the work, which is very inefficient.
- Data skew occurs when some partitions contain significantly more data than others, creating an imbalance of work. Instead of efficient parallel processing, one executor becomes a bottleneck.
	- Data skew is important because Spark cannot finish until **every single task** completes.
	- Data skew typically comes from uneven partition keys.
- **Symptoms:**
	- One task takes much longer than the others.
	- Most executors become idle.
	- CPU utilization drops late in the job.
	- Spark UI shows one or two tasks still running while the rest are complete.
	- The job appears to "hang" near completion.
	- Detection:
		- Using the Spark UI to review completion time for each task can help pinpoint tasks that run significantly longer than others.
- **Reducing Data Skew:**
	- Choosing a better partitioning key
	- Broadcasting the smaller table (to avoid a shuffle on a skewed join)
	- Filtering or pre-aggregating data before expensive operations
	- **Salting skewed keys**
	- Enabling Spark's Adaptive Query Execution (AQE), which can automatically mitigate some skewed joins
- **Salting:**
	- Salting is the practice of adding a random suffix to a skewed partition key in order to more evenly distribute data across partitions.
	- Later, after the heavy data processing is done, you remove the salt and combine the partial results.
	- This spreads the workload more evenly.
	- Tradeoff:
		- Salting improves parallelism but makes the pipeline more complex because you must perform an additional aggregation to recombine the salted keys.
		- It's usually reserved for cases where skew is severe enough that the performance gains outweigh the added complexity.
- **Adaptive Query Execution (AQE):**
	- In modern Spark, AQE observes the data **during execution** and can make better decisions than the original query plan.
	- One capability is handling skewed joins by splitting oversized partitions into smaller pieces so that more executors can participate.
	- AQE is powerful, but it's not magic. Understanding the root cause of skew is still important.

### Tradeoffs (Eliminating Skew)
- Pros:
	- Better executor utilization
	- Faster overall jobs
	- Less idle time
	- Better cluster efficiency
- Cons:
	- Additional pipeline complexity (e.g., salting)
	- More transformations
	- Sometimes more shuffling
	- May not be worthwhile for small datasets

### Mental Model
```
Balanced

████
████
████
████

↓

Fast

-------------------

Skewed

█
█
█
████████████████████

↓

Slow
```
- Spark is only as fast as its slowest task.
```
Join

↓

Shuffle

↓

Partition by Join Key

↓

One Key Is Huge

↓

One Partition Becomes Huge

↓

One Task Takes Forever

↓

Job Stuck at 99%
```
- Notice how all of these topics connect:
	- Wide transformations
	- Shuffles
	- Partitioning
	- Broadcast joins
	- Data skew
- They're not isolated concepts—they're different pieces of the same execution model.

### Common Interview Questions
1. Why does my Spark job stay at 99% complete for a long time?
> 	One possibility is data skew. If one partition contains significantly more data than the others, most executors finish quickly while one executor continues processing the oversized partition. Because Spark waits for every task to complete, the entire job appears stuck even though only one task remains.
2. You're running a Spark job that groups sales by `customer_id`. Most tasks finish in **20 seconds**, but one task takes **8 minutes**, and the job appears stuck at **99%**. What would you suspect, how would you confirm your suspicion, and what are a few ways you might address it?
> 	I'd suspect data skew. I'd confirm it by looking at the Spark UI to see if one task or partition is processing significantly more data or taking much longer than the others. If skew is present, I'd consider filtering earlier, pre-aggregating, salting skewed keys, or enabling Adaptive Query Execution if appropriate.
	- Looking at executor-level memory utilization is not the best indicator of data skew. Task-level metrics pinpoint the issue much more clearly.
3. Your teammate says: "Let's just increase the number of executors. That should fix the slow task." Would you agree? Why or why not?
> 	Increasing the number of executors solves issues related to resource limitations at the cluster level. It does not solve resource limitations at the executor level. Furthermore, adding more executors or increasing the size of existing executors doesn't solve the real issue: resource distribution. If data isn't being distributed evenly among executors, scaling won't solve the issue.
4. How do you know whether a slow Spark job is caused by insufficient cluster resources or by data skew?

| Insufficient Resources          | Data Skew                          |
| ------------------------------- | ---------------------------------- |
| Most tasks are slow             | Most tasks finish quickly          |
| All executors stay busy         | Many executors become idle         |
| CPU utilization stays high      | CPU utilization drops near the end |
| Scaling the cluster often helps | Scaling alone usually doesn't help |
| Work is evenly distributed      | One or a few partitions dominate   |

## 11. Partitioning in Spark

- Spark divides large datasets into partitions. Each partition represents a unit of parallel work. This is why partitioning is fundamental to Spark; No partitioning. = No parallelism.
- A partition is simply a subset of the dataset that Spark processes independently.
- Generally speaking, you want your data to be divided into at least N partitions, where N is the number of executors that are available to process data. If, for example, your data was divided into 2 partitions, but your Spark cluster had 10 executors available, 8 of them would sit idle.
- On the other hand, too many partitions is an issue because each partition contains very little data. Spark spends more resources allocating work among executors (scheduling tasks, starting tasks, coordinating work) than it does processing the data.
- The goal is to have partitions that are:
	- Large enough to justify scheduling and orchestration overhead.
	- Small enough to allow for good parallelism.
- **Repartition:**
	- Spark provides the `repartition()` method to increase or redistribute partitions.
	- When data is repartitioned, Spark performs a **shuffle** to redistribute data evenly.
	- `repartition()` is expensive because it is considered a **wide transformation**. Wide transformations require more network communication, which is expensive.
	- Although repartitioning is expensive, doing so before grouping can be beneficial because it allows Spark to group and aggregate more efficiently when it reaches that stage in the execution plan.
- **Coalesce:**
	- Spark provides the `coalesce()` method to reduce partitions **without a shuffle**, whenever possible. This makes it much cheaper to reduce the number of partitions in a dataset.
	- Reducing partitions could help alleviate the **small files problem** in a dataset, where data is spread across many small partitions. Storage systems dislike millions of tiny files.

| Operation       | Shuffle?                   | Typical Use                         |
| --------------- | -------------------------- | ----------------------------------- |
| `repartition()` | Usually Yes                | Increase or redistribute partitions |
| `coalesce()`    | Usually No (when reducing) | Reduce partitions efficiently       |

### Tradeoffs
- More Partitions:
	- Pros:
		- Better CPU utilization
		- More parallelism
		- Better load balancing
	- Cons:
		- More scheduling overhead
		- More task coordination
		- More metadata
		- Too many tiny tasks
- Less Partitions:
	- Pros:
		- Lower scheduling overhead
		- Larger sequential reads
		- Fewer output files
	- Cons:
		- Less parallelism
		- Idle executors
		- Longer-running tasks

### Mental Model
```
Large Dataset

↓

Partitions

↓

Tasks

↓

Executors
```
- More partitions create more opportunities for parallel work—but only up to the point where scheduling overhead outweighs the benefit.

### Common Interview Questions
1. When would you use `repartition()` instead of `coalesce()`?
> 	`repartition()` is appropriate when I need to increase the number of partitions or redistribute data evenly across the cluster, even though it requires a shuffle. `coalesce()` is preferred when reducing the number of partitions because it typically avoids a full shuffle, making it more efficient.
2. Your Spark job runs on a cluster with **32 executor cores**, but your input DataFrame has only **8 partitions**. Would increasing the number of partitions likely improve performance? Why or why not?
> 	Yes, I'd likely increase the number of partitions because with only 8 partitions and 32 executor cores, Spark can only execute 8 tasks in parallel, leaving many cores idle. I'd still avoid creating an excessive number of partitions, since that increases scheduling overhead, but increasing from 8 to something closer to the available parallelism would likely improve throughput.
	- The question somewhat implies that there is enough data in the partitions to make efficient use of the entire Spark cluster.
3. A teammate replaces every call to `df.coalesce(20)` to `df.repartition(20)` because "they both produce 20 partitions." Would you agree? What tradeoffs would you explain?
> 	I would explain that coalesce and repartition solve different problems. `coalesce()` is typically used when reducing the number of partitions, especially before writing output, because it can avoid a full shuffle while reducing the number of output files. If partitions are too large and data isn't being distributed evenly among executors, repartition would be appropriate.
4. Suppose you have `df.repartition("customer_id")` immediately before `df.groupBy("customer_id").count()`. Why might the `groupBy()` now be **much cheaper** than if you had skipped the `repartition()`?
> 	Because `repartition("customer_id")` organizes the data by the same key that `groupBy()` will use. After repartitioning, rows with the same `customer_id` are already colocated within the same partition, so Spark has much less work to do during the aggregation. While `repartition()` itself performs a shuffle, doing it once on the appropriate key can reduce or eliminate additional data movement required by later operations.
	- Repartitioning a dataset requires a **full shuffle**. Doing it before a `groupBy` is efficient because it allows Spark to focus on the aggregation instead of moving data around. It's not necessarily because "more of the partitioning is performed up front".
	- The answer isn't to "shuffle earlier." The answer is to "shuffle once into the right layout so later operations become cheaper."
	- Repartitioning before writing can also be an effective strategy for making downstream Spark jobs more efficient.

## 12. Caching / Persisting

- Suppose you execute the following code:
	```python
	filtered = df.filter(...)
	
	filtered.count()
	
	filtered.write.parquet(...)
	```
	- When spark executes the physical plan, it will filter the data twice. This is because two actions are being called on the filtered DataFrame.
	- This happens because Spark is **lazy**. When you run `df.filter(...)`, Spark doesn't actually store the result anywhere. It only remembers: "If someone asks for `filtered`, here's how to compute it." This "recipe" is called **lineage**.
	- If you optimize the code do cache the filtered DataFrame: `filtered = df.filter(...).cache()`
		- The expensive filter only runs when one of the actions is executed.
		- The second action reads from memory.
		- The `cache()` command tells Spark: "If this DataFrame gets computed, keep the result around because I'm probably going to use it again."
		- `cache()` **does not immediately cache anything**. This is because Spark is still lazy. Nothing happens until an action is executed.

### Cache vs. Persistence
- Spark actually provides two APIs for storing data in memory.
- Cache:
	```python
	df.cache()
	```
	- This is shorthand for the default persistence level, which typically stores data in memory when possible.
- Persist:
	```python
	from pyspark import StorageLevel
	
	df.persist(StorageLevel.MEMORY_AND_DISK)
	```
	- Persistence lets you choose where Spark stores the data.
	- Examples:
		- Memory only
		- Memory and disk
		- Disk only
- **When to Use Disk:**
	- Suppose you have 400 GB of cached data, but the executor only has 128 GB of memory.
	- Spark can spill the remaining cached memory to the disk.
	- It's slower than memory, but much faster than recomputing everything.
- **Why Not Cache Everything:**
	- More caching does not always mean faster runtime.
	- If you cache so much data that the memory fills, Spark starts:
		- Evicting cached data
		- Increasing garbage collection
		- Spilling to disk
	- Memory is limited. These background actions degrade performance.
- **When to  Unpersist:**
	- Cached data stays in memory until Spark removes it or your application ends.
	- When you're done using cached data, run: `df.unpersist()`.
	- This frees memory for other computations.
	- In long-running applications, forgetting to unpersist data that is no longer needed can reduce memory available for future work.
- **Good Candidates For Caching:**
	- Cache data when it is:
		- **Expensive to compute**
		- **Reused multiple times**
		- Referenced by multiple actions
		- Shared across multiple stages of a pipeline

### Real Example
```python
clean_events = (
    raw_events
        .filter(...)
        .join(...)
        .groupBy(...)
        .cache()
)

clean_events.count()

clean_events.write.parquet(...)

clean_events.select(...).show()
```
- Three actions are performed on `clean_events`.
- Without caching, Spark recomputes everything three times.
- With caching, Spark makes one computation and reads from memory three times.

### Tradeoffs
- Pros
	- Avoids recomputation
	- Speeds up repeated actions
	- Reduces repeated I/O
	- Reduces repeated shuffles

- Cons
	- Consumes executor memory
	- Can increase garbage collection
	- May spill to disk
	- Can hurt performance if overused

### Mental Model
- Think of caching like meal prep. Without caching:
	```
	Cook Dinner
	
	↓
	
	Eat
	
	↓
	
	Cook Same Dinner Again
	
	↓
	
	Eat
	```
- With caching:
	```
	Cook Once
	
	↓
	
	Store
	
	↓
	
	Reuse
	```
	- The benefit comes from avoiding repeated work, not from the size of the meal.
- Many people think they should always cache large data sets or expensive computations. This is not always the case. You should only cache data that you plan on **using more than once**.
	- A relatively small DataFrame that's reused many times might be an excellent caching candidate.
- Connection to Previous Topics:
	```
	Transformations
	
	↓
	
	Lazy Evaluation
	
	↓
	
	Lineage
	
	↓
	
	Action
	
	↓
	
	Recompute?
	
	↓
	
	Cache
	```

### Common Interview Questions
1. When would you cache a DataFrame?
> 	I cache a DataFrame when it's expensive to compute and will be reused multiple times. Caching avoids recomputing the lineage for every action, improving performance. I avoid caching one-time intermediate results because they consume memory without providing any benefit.
2. You have the following code. Would you recommend using caching? Why or Why not?
	```python
	sales = (
	    spark.read.parquet("sales")
	         .filter("amount > 100")
	         .join(customers, "customer_id")
	)
	
	sales.count()
	
	sales.write.parquet("output")
	```
> 	I would recommend caching the DataFrame since it is being used to perform an aggregation and being written to a parquet file. Caching allows Spark to perform the computation once an action is performed, store it in memory, then retrieve it for subsequent actions without the need for re-computation. Since the DataFrame is reused by multiple actions, caching prevents Spark from recomputing the entire lineage—including the filter and join—for each action.
> 	- Try to explicitly mention **re-computing the lineage** in your answer. 
1. A teammate says: "We should cache every DataFrame because memory is faster than recomputing." How would you respond? What tradeoffs would you explain?
> 	I would mention that caching every DataFrame would introduce memory bottlenecks. DataFrames that are used once should not be cached. If every DataFrame is reused, those that involve the most expensive computation should be prioritzed over those that involve relatively simple computation. If memory becomes exhausted, Spark may evict cached data or spill it to disk. At that point, the benefits of caching diminish, and excessive caching can even slow the application due to increased garbage collection and memory pressure.
2. Suppose you have:
	```python
	df.cache()
	
	df.count()
	
	df = df.filter("price > 100")
	```
	- Is the filtered DataFrame cached? Why or why not?
> 	No. `cache()` only applies to the DataFrame on which it was called. After `df.count()`, Spark materializes and caches that DataFrame. Calling `filter()` creates a new DataFrame with its own lineage. Spark can use the cached parent to avoid recomputing earlier transformations, but the filtered DataFrame itself isn't cached unless I explicitly call `cache()` on it.
	- Spark DataFrames are **immutable**. Every transformation creates a **new logical DataFrame**.

## 13. Window Functions

- Window functions allow you to solve analytical problems without writing inefficient self-joins or multiple aggregations.
- A window function performs a calculation across a group of related rows while preserving each individual row. Unlike group by, rows are preserved, not collapsed.
- Window functions are useful because many business questions require preserving individual records (rows). For example:
	- Running totals
	- Rankings
	- Top N customers
	- Previous purchase
	- Next purchase
	- Moving averages
	- Latest event per user
- None of the above examples are solved with `groupBy` alone. `groupBy` can only provide aggregates for unique values of a particular column or set of columns. Window functions allow you to perform "running aggregations" on individual rows, not groups of rows.
- **Spark Syntax:**
	```python
	from pyspark.sql.functions import rank
	from pyspark.sql.functions import sum
	from pyspark.sql.window import Window
	
	window = (
	    Window
	        .partitionBy("customer_id")
	        .orderBy("purchase_date")
	)
	
	df.withColumn(
	    "running_total",
	    sum("sales").over(window)
	)
	
	rank().over(
    Window.orderBy(desc("sales"))
	```
	- Calculates a running sales total for each customer, ordered by `purchase_date`.
	- Ranks customers in descending order based on sales.
- **Rank vs. Dense Rank vs. Row Number:**
	- `rank()`: Skips numbers when there are ties (e.g. 1, 2, 2, 4).
	- `dense_rank()`: Does not skip numbers when there are ties (e.g. 1, 2, 2, 3).
	- `row_number()`: Does not handle ties. Assigns strictly sequential rankings, regardless of ties (e.g. 1,2, 3, 4).
- **Lead vs. Lag:**
	- `lag()`: Provides the value of the previous row (first row is `null`).
	- `lead()`:  Provides the value of the following row (last row is `null`).
		- Useful for:
			- Session analysis
			- Time between events
			- Churn calculations

### Tradeoffs
- Pros:
	- Preserve rows
	- Avoid self-joins
	- Express complex analytics clearly
	- Often more efficient than multiple joins
- Cons:
	- Can require sorting within each partition
	- Large windows consume memory
	- Wide windows may shuffle data
	- Overusing windows can make queries harder to read

### Mental Model
```
groupBy()

Rows

↓

Aggregate

↓

Fewer Rows

-----------------------

Window

Rows

↓

Calculate

↓

Same Number of Rows
```

### Interview Questions
1. When would you use a window function instead of `groupBy()`?
> 	I'd use a window function when I need calculations across related rows while preserving each individual row. `groupBy()` aggregates and collapses rows, whereas window functions let me compute values like rankings, running totals, or previous events without losing row-level detail.
	- The key distinction between `groupBy()` and window functions is **maintaining row-level detail**.
2. Suppose you have an Orders table and your manager asks: "Show me each order along with that customer's total lifetime spending." Would you use `groupBy()` or a window function? Why?
> 	Since the manager wants to see each order (row) alongside the customer's (cumulative) total lifetime spending, I would use a window function instead of groupBy(). groupBy() would collapse all of the rows while performing the aggregation. A window function (partitioned by customer_id and ordered by purchase_date) preserves row-level detail while performing the aggregation.
3. You need to return **only the most recent login event for each user**. Would you use:
	- `rank()`
	- `dense_rank()`
	- `row_number()`
	- Why is that the best choice?
> 	I would use row_number() in this case because only the most recent login event needs to be returned, meaning tie-handling isn't required.
4. Suppose your manager asks: "For every login, show how many minutes have passed since the previous login." Which window function would you use?
	```python
	from pyspark.sql.window import Window
	from pyspark.sql.functions import lag, col
	
	window = (
		Window
		.partitionBy("user_id")
		.orderBy("login_timestamp")
	)
	
	df = df.withColumn(
		"previous_login",
		lag("login_timestamp").over(window)
	)
	
	df = df.withColumn(
		"minutes_since_previous",
		(
			col("login_timestamp").cast("long") -
			col("previous_login").cast("long")
		) / 60
	)
	```
> 	I would use `lag()` because I need to compare each login event with the previous login for the same user. I'd define a window partitioned by `user_id` and ordered by `login_timestamp`, use `lag(login_timestamp)` to retrieve the previous login time, and then calculate the time difference between the current and previous login.


# Distributed Processing

## 14. Fault Tolerance & Checkpointing

- Imagine you're processing 5 TB of clickstream data. Your job has been running for 45 minutes when executor 7 crashes.  Should Spark restart the entire job or recover lost work?
	- Restarting would waste 45 minutes.
	- Spark takes the second approach.
- **Fault Tolerance** allows Spark to recover from failures without restarting the entire application. This is one of its biggest advantages over older distributed systems.
- Remember, when you write Spark code, it doesn't execute it line-by-line. First, it reads all of the code and builds a lineage graph that it then uses to create an execution plan.
	- This lineage graph records **how to produce the data**, not the data itself.
	- When an executor fails and some partitions disappear, Spark knows how those partitions were created. It simply reruns the **necessary** tasks using the lineage graph.
- For example, imagine Spark created the following schedule as part of its execution plan:
	```
	Read
	
	↓
	
	Filter
	
	↓
	
	GroupBy
	
	↓
	
	Partition 3
	```
	- If an executor crashes and partition 3 is lost, Spark knows how rebuild **just** partition 3, without starting the whole job over again.
	- Other partitions are left untouched.
- Spark's Lazy Execution model also allows for fault tolerance. No code is actually executed until an action is called. Before then, Spark only spends time building an execution plan. This is an important part of its recovery model.
	- The problem with this model of execution is that Spark could lose track of where it left off in a task that has a long chain of execution, such as:
		```
		Read
		
		↓
		
		Filter
		
		↓
		
		Join
		
		↓
		
		GroupBy
		
		↓
		
		Window
		
		↓
		
		Join
		
		↓
		
		Aggregate
		
		↓
		
		Output
		```
		- This would be especially problematic if an executor crashed near the end of this task. All of the work would have to be redone. Spark just wouldn't need to redo **every single task**, it would only need to redo this large one.
- **Checkpointing** saves an intermediate result to reliable storage so Spark can restart from that point instead of replaying the entire lineage.
	```
	Read
	
	↓
	
	20 transformations
	
	↓
	
	Checkpoint
	
	↓
	
	20 more transformations
	
	↓
	
	Failure
	
	↓
	
	Restart here
	```
	- Spark only needs to redo 20 transformations, instead of 40.
- Checkpoints are typically stored in reliable, distributed storage, such as:
	- HDFS
	- Amazon S3
	- Azure Data Lake Storage
	- Google Cloud Storage
- The checkpoint survives executor failures because it lives outside the executors.
- While checkpointing can be very useful when handling large transformations, it isn't free. Spark needs to:
	- Materialize the data
	- Write it to storage
	- Read it later if needed
- If you checkpoint every transformation, your job becomes much slower.
- **Checkpointing is most useful when:**
	- Lineage becomes very long.
	- Recomputation would be expensive.
	- Long-running iterative workloads.
	- Streaming applications that run continuously.

- **Spark Streaming:**
	- Checkpointing is especially important for streaming jobs. Imagine a streaming job that has been running continuously for 6 months.
		- Spark can't grow the lineage forever.
		- Streaming engines periodically checkpoint state so they can recover efficiently after failures.
- **Fault Tolerance vs. Caching:**
	- Cache:
		- Stores an expensive DataFrame in memory, so it can be reused later.
		- If the cache disappears because an executor fails, Spark can recompute it from lineage.
	- Checkpoint:
		- Recovers *after* performing an expensive transformation, so it doesn't need to be recomputed.
		- Checkpoint **breaks the lineage** because the checkpoint becomes the new starting point.
- **Fault Tolerance vs. High Availability:**
	- Fault Tolerance asks: "Can the system recover after something fails?"
	- High Availability asks: "Can the system continue serving users with minimal interruption?"
	- Spark's lineage and checkpointing are primarily **fault-tolerance** mechanisms.

### Tradeoffs
- Lineage:
	- Pros:
		- No need to save every intermediate result.
		- Efficient recovery for short pipelines.
		- **Enables lazy evaluation**.
	- Cons:
		- Long lineage can make recomputation expensive.
- Checkpointing:
	- Pros:
		- Faster recovery.
		- Shorter lineage.
		- Essential for long-running streaming jobs.
	- Cons:
		- Additional storage I/O.
		- More expensive writes.
		- Should be used selectively.

### Mental Model
- Connection to Previous Topics:
	```
	Read Less
	        │
	        ▼
	Partition Pruning
	
	        │
	        ▼
	Predicate Pushdown
	
	        │
	        ▼
	Shuffle
	
	        │
	        ▼
	Caching
	
	        │
	        ▼
	Fault Tolerance
	
	        │
	        ▼
	Checkpointing
	```
- Imagine writing a research paper:
	- Without Checkpointing:
		```
		Page 1
		
		↓
		
		Page 100
		
		↓
		
		Computer crashes
		
		↓
		
		Rewrite everything
		```
	- With Checkpointing:
		```
		Save after Page 50
		
		↓
		
		Continue writing
		
		↓
		
		Computer crashes on Page 100
		
		↓
		
		Resume from Page 50
		```

### Common Interview Questions
1. Why does Spark need checkpointing if it already has lineage?
> 	Spark's lineage allows it to recompute lost partitions after a failure, which works well for shorter jobs. However, if the lineage becomes very long or the computation is expensive, recomputing everything from the beginning can take a long time. Checkpointing saves an intermediate result to reliable storage, allowing Spark to recover from that point instead of replaying the entire lineage.
2. A Spark job has a very long transformation pipeline. One executor fails near the end of execution. Explain how Spark recovers by default. Then explain how checkpointing changes the recovery process.
> 	Spark recovers by re-producing the lost data using the lineage graph. Checkpointing can enhance the recovery process by saving intermediate results to reliable storage. This allows spark to perform re-computation without starting from the beginning of the original lineage. The checkpoint becomes the new beginning.
3. A teammate suggests replacing every `cache()` call in your Spark jobs with checkpointing because: "Checkpointing is more reliable." Would you agree? What are the different purposes of **caching** and **checkpointing**, and when would you use each?
> 	Caching and checkpointing aren't mutually exclusive. Caching helps to avoid expensive recomputation during the execution of a task. Checkpointing helps recover from failures in long-running jobs by reducing the amount of lineage that must be recomputed. TThey can be used together, but they solve different problems. Caching improves performance by avoiding repeated computation, while checkpointing improves fault tolerance by limiting how much work must be recomputed after a failure.
	- Jobs aren't *designed* to fail, so checkpointing can't really be considered an optimization in the same way that caching is. Checkpointing actually **reduces performance slightly** because writing checkpoint data to durable storage is additional work. You accept that overhead because it improves recoverability.
4. If Spark already has lineage, why doesn't it simply keep the lineage forever instead of implementing checkpointing?
> 	Because very long lineage graphs become increasingly expensive to recompute after failures. In long-running jobs, especially streaming applications, lineage could grow indefinitely. Checkpointing truncates the lineage by materializing the current state to reliable storage, making recovery much faster and preventing lineage from growing without bound.

## 15. Exactly-once vs. At-least-once Processing

- Imagine your system processes credit card transactions:
	- A customer buys a $2,000 laptop.
	- While processing the payment, the worker crashes.
	- There are two unacceptable assumptions that could be made:
		- Assume the transaction succeeded and the customer potentially gets a free laptop.
		- Assume the transaction failed and risk charging the customer twice.
	- This is the fundamental problem of reliable distributed systems.
- Suppose a streaming pipeline crashes after saving the results of the computation, but before acknowledging the evemt:
	```
	Receive Event
	
	↓
	
	Process Event
	
	↓
	
	Save Result
	
	↓
	
	💥 Crash
	
	↓
	
	Acknowledge Event
	```
	- Kafka (or another message broker) never received an acknowledgement from the producer.
	- When the consumer restarts, it delivers the event again and it gets processed twice as a result.
- **Delivery Guarantees:**
	- At-most-once: An event is processed zero or one time.
		- Pros:
			- No duplicates
		- Cons:
			- Data loss
		- Good for:
			- Metrics
			- Logging
			- Non-critical telemetry
	- At-least-once: An event is processed one or more times.
		- Pros:
			- No lost events
		- Cons:
			- Duplicate processing
		- This is the most common guarantee in distributed systems because losing data is usually worse than processing it twice.
	- Exactly-once: The final effect is as if each event were processed exactly once.
		- **Imprtant Note:** This does **not** necessarily mean the event was literally processed only one time.
		- In the event of a crash, the event may be retried, but the **final result** only appears once.
		- Exactly-once processing isn't a magical feature. It's usually a combination of techniques, such as:
			- Idempotent operations
			- Transactions
			- Deduplication
			- Checkpointing
			- Offset management
- **Idempotency:**
	- Suppose an event said: "Increase balance by $100." Running it twice would be bad.
	- Instead, the event is made idempotent by having it say: "Set balance to $500." Now, the event can be run multiple times and **produce the same final result**.
- **Why At-least-once Is So Popular:**
	- Businesses tend to prefer accidentally charging someone's card, then **detecting and correcting** the duplicate event.
	- This is much better than never receiving any payment.
	- Many production systems intentionally choose **at-least-once** delivery and **make downstream processing idempotent**.
- **Spark Structured Streaming:**
	- Suppose Spark reads Kafka. Spark maintains:
		- Offsets
		- Checkpoint state
	- If Spark crashes, it restarts from the checkpoint state.
	- Depending on the sink and configuration, events may be replayed.
	- This is why downstream processing often needs to be idempotent or transactional.
- **Kafka Example:**
	- Imagine Kafka offsets (similar to checkpoints).
	- Spark begins processing events, starting from the offset.
	- Before committing offset, Spark crashes and restarts.
	- Kafka sends duplicate events to Spark, starting from the original offset.
	- **Duplicates happen unless the system is designed to handle them**.

### Tradeoffs
- At-most-once:
	- Pros:
		- No duplicates
		- Low latency
	- Cons:
		- Lost data
- At-least-once:
	- Pros:
		- Reliable
		- No lost events
	- Cons:
		- Duplicates
- Exactly-once:
	- Pros:
		- Correct final results
		- Best user experience
	- Cons:
		- Most complex
		- More coordination
		- Higher overhead

### Mental Model
- Connection to Previous Topics:
	```
	Retries
	      │
	      ▼
	Duplicates
	      │
	      ▼
	Idempotency
	      │
	      ▼
	Checkpointing
	      │
	      ▼
	Fault Tolerance
	      │
	      ▼
	Exactly-once Semantics
	```

### Common Interview Questions
1. Which guarantee is best?
> 	It depends on the business requirements. For telemetry or monitoring, at-most-once may be acceptable. For most data pipelines, at-least-once combined with idempotent processing provides a good balance of reliability and complexity. Exactly-once is valuable when duplicate effects are unacceptable, such as financial transactions, but it comes with additional implementation complexity.
2. You're building a pipeline that processes credit card payments. Which model would you choose and why? What tradeoffs influence your decision?
> 	When processing financial transactions, I would most likely choose exactly-once processing. Although this comes at the cost of higher complexity and maintenance overhead, it is worth the guarantee that the final effect of each event is applied only once., even if the event is processed more than once. If exactly-once processing wasn't available, I would choose at-least-once processing and design downstream services to be idempotent.
3. A teammate says: "Exactly-once means an event is literally processed only one time." How would you respond? Why is that statement not necessarily true in distributed systems?
> 	Exactly-once means an event only produces one result. It doesn't guarantee that an event is only processed once.
4. If an event can be processed multiple times, how can the result still be exactly once?
> 	Because the system detects or prevents duplicate effects. That can be done through idempotent operations, transactional writes, deduplication using unique event IDs, or coordinating message acknowledgments with committed results. Even if the processing logic executes more than once, only one successful effect is visible.
5. Is exactly-once always better than at-least-once?
> 	Not necessarily. Exactly-once provides stronger correctness guarantees but increases complexity and coordination overhead. If the application can tolerate duplicate processing through idempotent operations, at-least-once delivery is often simpler, performs better, and is sufficient for many production systems.

## 16. Watermarks & Late Data

- Data can arrive late to a streaming pipeline for many reasons, including:
	- Slow mobile devices
	- Network outages
	- Retry mechanisms
	- Kafka consumer lag
	- Temporary system failures
	- Clock differences between systems
- Suppose Spark is counting visits to a website every minute. Events are arriving continuously and `Visits Per Minute` needs to be computed.
	- Imagine Spark publishes a result of `150 Visits Per Minute` at 12:00, but late data arrived that affects the computation.
	- The correct answer is actually `151 Visits Per Minute`.
	- What should Spark do? There are only three choices:
		1. **Ignore it** and keep the original result. Simple, but now your results are inaccurate.
		2. **Update the old result**. This is the correct approach, but now downstream systems must handle the discrepency.
		3. **Wait forever**. Never publish any results. Clearly impossible.
- **Streaming Window:**
	- Let's visualize time:
		```
		Events
		
		↓
		
		12:00
		
		↓
		
		12:01
		
		↓
		
		12:02
		
		↓
		
		12:03
		```
		- Suppose we're computing `One-minute windows`.
		- Spark groups events by **event time**, not arrival time.
		- For example:
			```
			Arrives
			
			12:03
			
			↓
			
			Event Time
			
			12:00
			```
			- Even though it arrived late, it still belongs in the `12:00 window`.
- **Event Time vs. Processing Time:**
	- **Event time** describes when the event actually **occurred**.
	- **Processing time** describes when the event was **received** by Spark.
- A **watermark** is a threshold that tells Spark how long to wait for late events before considering a window complete.
	- For example, suppose we configure:
		```
		Watermark
		
		10 minutes
		```
		- If the current **event time** is 12:20, Spark will asume everything before 12:10 is finished.
		- Any event older than 12:10 is now discarded.
		- As long as the **event time** of a given event is **older than** the watermark, Spark will process it.
	- Without Watermarks:
		- Spark keeps aggregation windows open forever.
		- Nothing gets processed.
		- Memory usage grows out of control.
	- With Watermarks:
		- Spark waits long enough to capture most delayed events while eventually closing old windows and **freeing state**.
	- **Important Note:** Watermarks do **not** make events arrive on time. They simply define how long Spark is willing to wait for late events to arrive.
- **Spark Example:**
	```python
	events \
	  .withWatermark("event_time", "10 minutes") \
	  .groupBy(window("event_time", "1 minute")) \
	  .count()
	```
	- This means:
		- Group events into one-minute windows.
		- Wait up to ten minutes for late events.
		- Then finalize the window.

### Tradeoffs
- Small Watermark:
	- Pros:
		- Lower memory
		- Faster final results
	- Cons:
		- More late events discarded
- Large Watermark:
	- Pros:
		- More accurate
	- Cons:
		- More memory
		- Longer before results become final

### Mental Model
- Connection to Previous Topics:
	```
	Retries
	      │
	      ▼
	Late Events
	      │
	      ▼
	Exactly-once
	      │
	      ▼
	Checkpointing
	      │
	      ▼
	Watermarks
	      │
	      ▼
	Streaming State
	```
- Lots of design decisions involve balancing two competing goals:

| Goal A      | Goal B       |
| ----------- | ------------ |
| Accuracy    | Latency      |
| Reliability | Simplicity   |
| Memory      | Completeness |
| Throughput  | Correctness  |

### Common Interview Questions
1. Why do streaming systems need watermarks?
> 	Because events don't always arrive in chronological order. Watermarks allow the system to balance accuracy and resource usage by waiting for late events up to a configurable threshold. Once the watermark passes a window, Spark can finalize the result and release the associated state.
2. You're computing `Sales Per Minute` using Spark Structured Streaming. A mobile device loses connectivity for **five minutes** and then sends all of its buffered events. How does Spark decide whether those events should still be included in the correct one-minute windows? What role does the watermark play?
> 	Spark decides whether those events should still be included in the correct one-minute windows based on the event time of the buffered events and the watermark configuration. If the event is older than the watermark, it is discarded. Otherwise it is added to the appropriate window and used to calculate the final results. Once the watermark passes a window, Spark considers that window complete, allowing it to release the state associated with that window.
3. A teammate says: "Let's configure a 24-hour watermark so we never lose late events." Would you agree? What tradeoffs would you explain?
> 	I would mention that a 24-hour watermark would impose significant constraints on executor memory, even though it would produce a higher guarantee of accurate final results. It would also delay the availability of the final results. I would encourage them to analyze the distribution of event delays—for example, determine how many events arrive within 1 minute, 5 minutes, 10 minutes, and so on. This would allow them to configure the watermark more appropriately.
	- Using an average delay instead of analyzing the distribution could be problematic  if there are significant outliers in the data. Engineers more often use percentiles (like P99 Latency) than simple averages.
4. If late events are discarded after the watermark, doesn't that make the results wrong?
> 	Potentially, yes. Watermarks intentionally trade perfect completeness for bounded memory usage and timely results. The appropriate watermark depends on the business requirements. Some applications prefer fast, nearly complete results, while others are willing to wait longer to include more late events.

## 17. Backpressure

- Backpressure is a condition where data is produced faster than it can be consumed, causing work to accumulate.
- **Streaming Pipeline Example:**
	- Suppose Kafka receives 100,000 events/sec, while Spark can only process 60,000 events/sec.
	- This causes Kafka to begin accumulating messages at a rate of 40,000 events/sec.
	- Consumer lag increases.
	- This is a manifestation of backpressure.
- **Consumer Lag:**
	- This is one of the easiest ways to observe backpressure.
	- When producers continue writing faster than consumers can process, the lag keeps growing.
- **Common Causes:**
	- Heavy computation
	- Slow storage (writes)
	- Data skew
	- External systems (API calls)
- **Why Is Backpressure Dangerous?**
	- Suppose a producer continues forever.
	- The lag continues to grow, filling up the queue.
	- Eventually:
		- Memory pressure
		- Disk pressure
		- High latency
		- Timeouts
		- Failure
- **How Do Systems Respond?**
	- Option 1: Slow the producers so that production matches consumption.
	- Option 2: Scale the consumers so that consumption matches or exceeds production.
	- Option 3: Optimize processing so that the consumer can speed up without additional resources.
		- Potential bottlenecks:
			- Unnecessary shuffling
			- Bad partitioning
			- Missing broadcast join
			- Slow UDF
	- Option 4: Buffer to gracefully handle temporary spikes in production.
		- Sustained overload eventually exhausts the buffer.
- **Backpressure Isn't Always Bad:**
	- Backpressure can actually serve as a **protective mechanism**.
	- Consumers can't always meet the demands of producers in real distributed systems.
	- Backpressure allows systems to preserve data rather than immediately dropping it.
- **Streaming Systems:**
	- Different systems implement backpressure differently. Examples include:
		- Kafka buffers messages on brokers.
		- Spark Structured Streaming adjusts micro-batch processing.
		- Flink has built-in backpressure propagation.
		- Reactive systems may explicitly signal producers to slow down.
	- The core concept remains the same. **The consumer can't keep up**.

### Tradeoffs
- Large Buffers:
	- Pros:
		- Absorb traffic spikes.
		- Reduce data loss.
	- Cons:
		- Higher latency.
		- More memory or disk usage.
		- Longer recovery after spikes.
- Small Buffers:
	- Pros:
		- Lower latency.
		- Less memory.
	- Cons:
		- Less tolerant of traffic bursts.
		- Greater risk of dropped data if overload persists.

### Mental Model
- Connection to Previous Topics:
	```
	Producer
	
	↓
	
	Kafka
	
	↓
	
	Consumer
	
	↓
	
	Checkpointing
	
	↓
	
	Exactly-once
	
	↓
	
	Watermarks
	
	↓
	
	Backpressure
	```

### Common Interview Questions
1. What would you do if Kafka consumer lag kept increasing?
> 	First I'd determine why the consumers can't keep up. I'd look for bottlenecks such as expensive processing, data skew, slow downstream systems, or insufficient consumer capacity. Depending on the cause, I might optimize the processing pipeline, increase consumer parallelism, or investigate whether producers are generating data faster than expected. The solution depends on identifying the bottleneck rather than simply adding more hardware.
2. A Kafka topic is receiving **150,000 events per second**, but your Spark Structured Streaming application consistently processes only **90,000 events per second**. Consumer lag continues to grow throughout the day. How would you investigate the problem? What are some possible solutions?
> 	I would investigate whether consumers are operating efficiently by looking into bottlenecks such as data skew, excessive shuffling, or expensive processing, then optimizing accordingly. If that wasn't the issue, I'd review metrics to see if producers are operating normally or if there's a traffic spike. If there's a traffic spike, I would consider buffering or scaling up consumers. I'd also investigate downstream dependencies such as the data warehouse or external APIs because a slow sink can create backpressure even when Spark has available compute capacity.
	- Backpressure can be caused by issues at the producer level, consumer level, or other downstream dependencies.
3. A teammate says: "If we see backpressure, we should immediately double the number of Spark executors." Would you agree? Why or why not? When would scaling out help, and when might it not solve the problem?
> 	I would not agree. Backpressure is a normal mechanism for handling situations where producers temporarily outpace consumers. It becomes a problem when the backlog continues growing and the system can't recover, leading to increased latency, resource exhaustion, or failures. Mitigating the issue requires correctly identifying where the bottleneck is and acting accordingly.
4. How would you know whether adding executors will actually help?
> 	It depends on whether the workload can benefit from additional parallelism. If the bottleneck is CPU-intensive processing and the work is evenly distributed across partitions, adding executors can increase throughput. However, if the bottleneck is data skew, a slow downstream system, or an external API, additional executors may sit idle or simply wait on the same bottleneck.
	- Adding resources primarily helps when the primary issue is insufficient parallelism (not caused by issues such as data skew).

# Optimization

## 18. Predicate Pushdown (Spark perspective)

- Predicate pushdown exists because it allows Spark to efficiently query Parquet files by only scanning the rows it needs to.
- In databases, a **predicate** is simply a filtering condition. For example: `WHERE amount > 100`.
- Predicate pushdown means pushing filtering logic as close to the data source as possible so unnecessary data is never read into Spark.
	- **This doesn't mean it allows Spark to filter faster**.
	- Predicate pushdown simply allows Spark to **avoid reading unnecessary data**.
- Parquet is mainly what makes predicate pushdown possible.
	- In addition to its columnar storage format, Parquet also assigns metadata to groups of rows.
	- For example:
		```
		Country
		
		Min = Canada
		
		Max = Mexico
		```
		- If your query asks for `country = 'USA'`, Spark immediately knows it should skip that row group.
	- This is so powerful because reading data from storage is expensive, even with cloud storage.
- File Format Support:
	- Excellent:
		- Parquet
		- ORC
	- Limited or None:
		- CSV
		- JSON
		- Plain text
	- This is because CSV doesn't store metadata about its contents. This requires Spark to read all of the data to find matching rows.
- Predicate Pushdown vs. Filter Early:
	- Filter Early:
		- Reduce data before expensive operations.
			```python
			filter()
			
			↓
			
			join()
			```
	- Predicate Pushdown:
		- Don't read unnecessary data from storage.
		- Only possible with the write storage format, such as Parquet or ORC.
- You can only verify predicate pushdown occurred by using the `df.explain()` method. You should see something like:
	```
	PushedFilters:
	Country = USA
	```

### Tradeoffs
- Pros:
	- Less disk I/O
	- Less network traffic
	- Less CPU spent processing irrelevant rows
	- Faster scans
	- Lower memory usage
- Cons:
	- Depends on the data source and file format
	- Not every predicate can be pushed down
	- Complex expressions or user-defined functions (UDFs) may prevent pushdown

### Mental Model
Think of Library
- Without predicate pushdown:
	```
	Take Every Book
	
	↓
	
	Search Inside
	
	↓
	
	Keep One
	```
- With predicate pushdown:
	```
	Use Library Index
	
	↓
	
	Pull Only One Book
	```
- Connection to Previous Topics:
	```
	Parquet
	
	↓
	
	Metadata
	
	↓
	
	Predicate Pushdown
	
	↓
	
	Less Data Read
	
	↓
	
	Less Shuffle
	
	↓
	
	Faster Job
	```
	- Reducing the amount of data you read often reduces the amount of data you later shuffle.

### Common Interview Questions
1. Why is Parquet often faster than CSV?
> 	One reason is predicate pushdown. Parquet stores metadata that allows Spark to skip reading row groups that cannot satisfy a filter predicate. CSV lacks that metadata, so Spark generally has to read every row before applying the filter.
	- Columnar storage, compression, and column pruning are also important advantages.
2. Suppose you have the following query:
	```python
	(
	    spark.read.parquet("transactions")
	         .filter("country = 'USA'")
	         .groupBy("customer_id")
	         .sum("amount")
	)
	```
	- How could predicate pushdown improve the performance of this query? What work is Spark avoiding?
> 	Predicate pushown improves performance by allowing spark to avoid reading unnecessary data from storage. Since the data is stored in Parquet format, Spark uses the associated metadata to skip reading irrelevant groups of rows.
	- It's important to distinguish between reading from storage (disk) and reading from memory (RAM). Reducing the amount of that read from **storage** reduces disk I/O overhead, not memory overhead.
3. A teammate says: "We filter immediately after reading the data, so predicate pushdown doesn't matter." How would you respond? What is the difference between **filtering early** and **predicate pushdown**?
> 	Filtering early reduces the amount of data processed by downstream transformations like joins and aggregations, but Spark has already read the data from storage. Predicate pushdown reduces disk or cloud storage I/O by allowing the data source to skip reading data that cannot satisfy the filter in the first place."
	- It's important to explain **where** the optimization occurs. Predicate pushdown optimizes data read from disk or cloud storage, not the amount of data that is filtered afterward.
4. Suppose you have two queries:
	- Query A:
		```python
		spark.read.csv("transactions.csv") \
		    .filter("country = 'USA'")
		```
	- Query B:
		```python
		spark.read.parquet("transactions.parquet") \
		    .filter("country = 'USA'")
		```
	- Both apply the same filter. Why is Query B often dramatically faster?
		- **Predicate pushdown** can skip reading irrelevant row groups.
		- **Columnar storage** means only the required columns need to be read (column pruning).
		- **Compression** reduces the amount of data read from storage.
		- These optimizations combine to significantly reduce I/O.

## 19. Broadcast Joins

- Joins are expensive because they require a lot shuffling. In order to join two tables, Spark needs to make sure all data with the same join key exists in the same partition.
- Now image Table A is 5 TB in size and Table B is 200 MB. Should 5 TB of data be shuffled across the cluster to perform the join?
	- Probably not. Instead, Spark asks: "Why don't I send the small table to every executor?"
	- This is known as a **broadcast join**. Instead of moving both datasets, Spark sends the small dataset to every partition.
	- Now, every executor has the data needed to join the tables, without shuffling all of the data.
- Broadcast joins significantly reduce the amount of network traffic that would otherwise be spent on shuffling large amounts of data across a cluster.
- Spark has a configuration called a **broadcast threshold** that allows it to automatically choose a broadcast join when one side of the join is below that threshold.
	- You don't always have to specify it yourself.
	- However, it's still important to understand when it makes sense and how to encourage it when needed.
- **Explicit Broadcast:**
	```python
	from pyspark.sql.functions import broadcast
	
	orders.join(
	    broadcast(customers),
	    "customer_id"
	)
	```
	- This tells Spark to treat the `customers` table as the small table and to broadcast it to all executors.
- When to use broadcast:
	- Customer dimension table
	- Product catalog
	- Country lookup table
	- Currency lookup table
	- State abbreviations
	- Small configuration tables
	- Generally:
		- Thousands or even a few million rows **may** still be considered small enough, depending on row width and cluster memory.
		- The important factor is **memory footprint**, not just row count.
- When you should **not** use broadcast:
	- Suppose you needed to join a 5 TB `Orders` table with a 2 TB `Customers` table.
	- Broadcasting 2 TB of data to every executor would be disastrous. This would likely:
		- Run out of memory
		- Cause garbage collection issues
		- Fail the job

### Broadcast Join vs. Shuffle Join
- Broadcast Join:
	```
	Large Table
	
	↓
	
	Stay Put
	
	----------------
	
	Small Table
	
	↓
	
	Broadcast
	
	↓
	
	Every Executor
	```
	- Only the small dataset moves.
- Shuffle Join:
	```
	Large Table
	
	↘
	
	Shuffle
	
	↗
	
	Small Table
	```
	- Both datasets move.

### Tradeoffs
- Pros:
	- Avoids shuffling the large table
	- Much less network traffic
	- Often dramatically faster
	- Great for fact-to-dimension joins
- Cons:
	- Requires enough memory on every executor
	- Poor choice if the "small" table is actually large
	- Broadcasting itself has a cost, so it isn't free

### Mental Model
```
Normal Join

Move Both Tables

↓

Join

------------------------

Broadcast Join

Move Small Table

↓

Local Join
```
- If one table is tiny, don't move the giant one.

### Common Interview Questions
1. When would you use a broadcast join?
> 	I'd use a broadcast join when one side of the join is small enough to fit comfortably in each executor's memory. Instead of shuffling both datasets, Spark broadcasts the small table to every executor, allowing each executor to perform the join locally. This avoids expensive network shuffling of the large dataset and can significantly improve performance.
2. Suppose you're joining 10 TB of clickstream data and 20 MB of country lookup data. Would you use a broadcast join? Why or why not?
> 	I would use a broadcast join. Broadcasting 20MB of data to each executor is a lot less expensive than shuffling up to 10 TB of data when performing a regular join. Each executor receives the broadcast table in memory and performs the join against its local partitions of the large dataset.
3. A teammate says: "Broadcast joins are always faster than regular joins, so we should broadcast every table." How would you respond? What tradeoffs would you explain?
> 	Broadcast joins should only be used when one table is significantly larger than the other and broadcasting the second table would not impose memory constraints on each executor. Broadcast joins can become impractical because every executor must keep the broadcast table in memory. As the table grows, memory consumption and garbage collection become much bigger concerns than the network cost alone.
4. Suppose you have a 5 TB Orders table and a 400 MB Customers table. Would you broadcast the Customers table?
	- **It depends**. The decision ultimately depends on factors such as:
		- Executor memory
		- Number of executors
		- Spark's broadcast threshold
		- Whether other workloads are competing for memory
		- The size of each row (400 MB compressed on disk might be much larger in memory)
## 20. Partition Pruning

- Partition pruning exists to allow Spark to skip entire partitions of data because it knows they cannot satisfy the query.
	- While predicate pushdown helps avoid reading unnecessary *rows*, partition pruning helps avoid reading unnecessary *files or directories*.
	- Don't confuse storage partitions with data partitions. The partitions used in pruning are storage partitions, related to how the data is organized at the storage layer.
- **Predicate Pushdown vs. Partition Pruning:**
	- Predicate pushdown occurs *while* a file is being read.
	- Partition pruning occurs *before* opening and reading a file. It occurs at the directory level, skipping entire directories or files. **Partition pruning happens before predicate pushdown**.
- Partition pruning is possible because Spark understands the directory layout.
	- Folders are treated as partition columns.
	- Spark doesn't have to inspect the data inside the folder to determine which folders match.
- When Partition Pruning **Doesn't** work:
	- Suppose your query is `WHERE YEAR(order_date) = 2026` instead of `WHERE year = 2026`.
	- Spark may not be able to prune partitions because it has to compute the `YEAR()` function first.
	- Similarly, wrapping partition columns in complex expressions or UDFs can prevent pruning.
	- A general rule of thumb is: Filters directly on the partition columns are the easiest for Spark to prune.
- **Choosing Good Partition Columns:**
	- Good columns have:
		- Frequently used filters
		- Moderate cardinality
		- Relatively even distribution

### Tradeoffs
- Pros:
	- Reads fewer files
	- Less I/O
	- Faster scans
	- Lower cloud storage costs
	- Scales well for large datasets
- Cons:
	- Requires thoughtful partition design
	- Too many partitions create metadata overhead
	- Poor partition choices can hurt performance
	- Only helps when queries filter on partition columns

### Mental Model
- Without Partition Pruning:
	```
	Enter Every Room
	
	↓
	
	Look Around
	
	↓
	
	Find One Box
	```
- With Partition Pruning:
	```
	Know the Correct Room
	
	↓
	
	Walk There Directly
	
	↓
	
	Open Only That Room
	```
- Connection to Previous Topics:
	```
	Directory
	
	↓
	
	Partition Pruning
	
	↓
	
	Relevant Files
	
	↓
	
	Parquet Reader
	
	↓
	
	Predicate Pushdown
	
	↓
	
	Relevant Row Groups
	
	↓
	
	Column Pruning
	
	↓
	
	Relevant Columns
	
	↓
	
	Spark Processing
	
	↓
	
	Shuffle
	
	↓
	
	Results
	```
- Choosing a good partition key involves balancing three factors.
	1. **Query Patterns:** Can Spark effectively prune partitions?
	2. **Cardinality:** How many unique values exist?
	3. **Data Distribution:** Will some partitions be enormous while others are tiny?

### Common Interview Questions
1. Why partition data by date?
> 	Many analytical queries filter by date. Partitioning by date allows Spark to prune entire partitions, so it only reads the relevant files instead of scanning the entire dataset. This significantly reduces I/O and improves query performance.
2. Suppose an analyst runs:
	```sql
	SELECT *
	FROM events
	WHERE year = 2026
	  AND month = 7
	```
	- Explain how partition pruning improves this query. What work is Spark avoiding?
> 	Partition pruning improves this query by allowing spark to only scan files within a specific subdirectory of the entire dataset. Spark will be able to avoid reading files from other years and other months within the 2026 directory, greatly reducing disk I/O overhead. Because the data is partitioned by year and month, Spark can determine from the directory structure which partitions satisfy the query. It only reads the files under `year=2026/month=07` and never opens the files in the other partitions.
3. A teammate suggests partitioning a 500 TB file by `user_id` because every user gets there own folder. Would you agree? Why or why not?
> 	I would emphasize that choosing a partition key should be based on current data access patterns, rather than an arbitrary organizational preference. If analysts routinely analyze data for individual users, user_id would be a good partition key. If analysts routinely analyze data for groups of users based on another attribute, user_id would be a poor partition key because it doesn't tell Spark which directories it can avoid reading. I'd also consider the cardinality of the partition column. Even if user_id matches the access pattern, creating hundreds of millions of partitions would introduce significant metadata overhead and likely create many small files. I'd look for a partition key that balances query performance with a manageable number of partitions.
	- It's important to recognize that optimizing for query performance can come at the cost of increased metadata.
4. Suppose an analyst almost always runs queries such as:
	```sql
	WHERE order_date BETWEEN
	'2026-07-01'
	AND
	'2026-07-31'
	```
	- Would you partition by dat, month, or year?
		- There's no single correct answer. For example:
			- If each day contains hundreds of gigabytes, partitioning by **day** may make sense because queries can prune down to exactly the days they need.
			- If each day contains only a few megabytes, daily partitions could create too many small partitions, so **month** might be more appropriate.
			- A common compromise for large datasets is a hierarchy like `year/month/day`, which gives flexibility for a variety of date-range queries.
		- Start with how the data will be queried, then design the storage layout to support those queries.

## 21. Bucketing 

- Bucketing means assigning rows to a fixed number of buckets using a hash of one or more columns.
- For example: `bucket = hash(customer_id) % 8`
	- If the result is 0, the row goes into `bucket_00000`.
	- If the result is 5, the row goes into `bucket_00005`.
	- Every row with the same `customer_id` always hashes to the same bucket.
- Bucketing can help when two large tables need to be joined. If both tables are bucketed the same way, Spark knows how to match buckets from each table to perform the join, potentially reducing the amount of shuffling that is required.
- **Bucketing vs. Partitioning:**
	- Partitioning creates **directories**, which is useful for partition pruning.
	- Partitioning answers: Which directory should this row go into?
	- Bucketing creates **buckets**, which can be helpful for performing joins and some aggregations.
	- Bucketing answers: Within that directory, which bucket should this row go into?
- Bucketing and partitioning can be used together. For example:
	```
	sales/
	
	year=2026/
	
	    bucket0
	
	    bucket1
	
	    bucket2
	```
	- Now you get:
		- Partition pruning by `year`.
		- Efficient joins by `customer_id`.

### Tradeoffs
- Bucketing shifts work from **read time** to **write time**.
- Without bucketing:
	```
	Cheap Writes
	
	↓
	
	Expensive Reads
	```
- With bucketing:
	```
	More Expensive Writes
	
	↓
	
	Potentially Cheaper Reads
	```
- This is why bucketing is useful when the same datasets are joined repeatedly.
- Pros:
	- Can reduce shuffle for repeated joins.
	- Useful for frequently joined large tables.
	- Avoids creating enormous numbers of partitions.
- Cons:
	- More expensive writes.
	- Fixed bucket count.
	- Operational complexity.
	- Less valuable when broadcast joins or AQE solve the problem.

### Limitations
- Bucketing isn't a silver bullet. Some considerations include:
	- The number of buckets is fixed when the data is written.
	- If bucket counts don't match between datasets, the optimization may not apply.
	- Modern Spark features like Adaptive Query Execution (AQE) and broadcast joins can reduce the need for bucketing in many cases.
	- Maintaining bucketed tables adds complexity to data pipelines.
- Because of this, many modern data platforms rely less on bucketing than they did in the past.

### Mental Model
- Connection to Previous Topics:

| Optimization       | Reduces                           |
| ------------------ | --------------------------------- |
| Partition Pruning  | Files read                        |
| Predicate Pushdown | Data read within files            |
| Broadcast Join     | Shuffle                           |
| Bucketing          | Future shuffle for repeated joins |
| Coalesce           | Small files                       |
| Caching            | Recomputation                     |

| Technique        | Organizes By                              | Purpose                    |
| ---------------- | ----------------------------------------- | -------------------------- |
| **Partitioning** | Query filters (date, region, environment) | Read fewer files           |
| **Bucketing**    | Join keys (customer_id, product_id)       | Shuffle less during joins  |
| **Sorting**      | Ordered values                            | Faster range scans, merges |
| **Broadcasting** | Table size                                | Eliminate shuffle entirely |

### Common Interview Questions
1. When would you use bucketing?
> 	Bucketing is useful when large datasets are repeatedly joined on the same key. By hashing rows into a fixed number of buckets during writes, Spark may be able to perform future joins with less data redistribution. However, because it increases write complexity and modern Spark optimizations often reduce its benefits, I'd reserve it for workloads where repeated joins justify the additional maintenance.
2. Your data warehouse joins the same two **500 GB** tables on `customer_id` dozens of times every day. Would bucketing be worth considering? Why or why not? What factors would influence your decision?
> 	Bucketing would be worth considering because the tables are large and joined repeatedly on a daily basis. Before implementing bucketing, I would consider alternatives such as using the Adaptive Query Engine. If this could not meaningfully improve performance, I would emphasize carefully designing the bucketing strategy to ensure it produces the maximum benefit for current and future workloads. Because both tables are bucketed on `customer_id`, Spark may be able to perform the join with less data redistribution, reducing shuffle costs across repeated executions.
3. A teammate says: "Instead of bucketing, let's partition the tables by `customer_id` so matching rows are always together." How would you respond? What problems might that introduce compared to bucketing?
> 	I wouldn't recommend partitioning by `customer_id` because it typically has extremely high cardinality. That would create millions of partitions and likely millions of small files, making the dataset difficult to manage and query efficiently. Bucketing is designed for this scenario because it groups rows into a fixed number of buckets regardless of how many unique customer IDs exist.
	- Partitioning by a lower-cardinality key and bucketing by customer_id might be a better approach. Remember, partitioning decides how many directories there are. Bucketing determines how many files or subdirectories go inside that directory.

## 22. Small Files Problem

- File size in distributed systems is very important because Spark needs to do more than just read bytes. It must also:
	1. Discover the file
	2. Read its metadata
	3. Schedule a task
	4. Open a network connection
	5. Read the contents
	6. Close the file
- This overhead exists for **all files**, regardless of size.
- There are several common causes that create small files, including:
	- **Too many output partitions:** When writing data to a file, Spark typically creates one file per output partition. If you `repartition()` a dataset across an excessive number of partitions before writing, Spark will end up creating a lot of small files.
	- **Streaming Jobs:** Imagine a streaming pipeline that writes data to a file every minute. If the pipeline only processes 50 MB of data per minute, it will end up creating 1440 small files every day.
	- **Over-Partitioning:** When you partition by a key with high cardinality, you end up creating a lot of small files.
- **Why small files are bad:**
	- **Metadata Overhead:** Spark must first discover every file. On cloud storage, this means:
		- Listing objects
		- Reading metadata
		- Planning tasks
		- Sometimes, the metadata scan becomes slower than the data scan.
	- **Task Scheduling:** Remember, Spark assigns one partition to one task. Thousands of tiny files become thousands of tiny tasks. The cluster ends up spending more time scheduling tasks than it does executing them.
	- **Object Storage Performance:** Cloud storage systems such as S3 are optimized for relatively large objects. Millions of tiny objects creates significant request overhead and can increase costs because many cloud providers charge per request in addition to storage.
	- **Poor Compression:** Compression algorithms work better on larger blocks of data. Very small files compress less efficiently.
	- **Parallelism isn't always better:** Parallelism must be balanced with scheduling overhead when tuning performance of a Spark job.
- **Fixing The Small File Problem:**
	- **Coalesce:** Performing `coalesce()` on a DataFrame before writing it can reduce the number of files written to the storage layer. This is one of the most common reasons you'll see `coalesce()` used near the end of a Spark job.
	- **Repartition Thoughtfully:** Choose a partition count that balances parallelism with reasonable file sizes.
	- **Compaction:** Many production systems periodically merge small files into larger ones. Modern table formats like Delta Lake, Apache Iceberg, and Apache Hudi provide built-in support for file compaction.
- **What's a Good File Size:**
	- **There's no universal answer.** In many production data lakes, engineers aim for Parquet files in the **hundreds of megabytes** range (often around 100–500 MB), but the right size depends on workload, storage, and cluster characteristics. The key idea is to avoid both extremes:
		- **Very tiny files** create excessive overhead.
		- **Very large files** can reduce parallelism because fewer tasks can process them simultaneously.

### Tradeoffs
- Large Files:
	- Pros:
		- Lower metadata overhead
		- Fewer tasks
		- Better compression
	- Cons:
		- Reduced parallelism
		- Longer recovery if a task fails
		- Individual tasks may take much longer
- Small Files:
	- Pros:
		- More parallelism
		- Smaller units of work
	- Cons:
		- Metadata overhead
		- Scheduler overhead
		- More storage requests
		- More planning time
		- Lower compression efficiency

### Mental Model
```
Too Many Partitions
        │
        ▼
Too Many Output Files
        │
        ▼
Small Files
        │
        ▼
More Tasks
        │
        ▼
More Scheduler Overhead
        │
        ▼
Slower Jobs
```
- This is why partitioning decisions have long-term consequences.

### Common Interview Questions
1. Why are many small Parquet files bad?
> 	Spark incurs overhead for every file it processes, including listing the file, reading metadata, scheduling a task, and opening the file. When a dataset contains thousands or millions of very small files, that overhead can dominate execution time. Consolidating files into reasonably sized Parquet files improves planning efficiency and overall performance while still allowing adequate parallelism.
2. A daily Spark job writes **150,000 Parquet files**, each about **200 KB**. The next morning, analysts complain that queries are becoming slower every week. What do you suspect is happening, and how would you improve the situation?
> 	Queries are likely becoming smaller due to the scheduling and metadata overhead associated with small files. To improve the situation, I would suggest coalescing the DataFrame at the end of the Spark job, re-considering the partition count by balancing parallelism and file size, or compacting small files into larger files. Because the job runs daily, each execution adds another large batch of small files. Even if each day's output is manageable, the cumulative number of files continues to grow, so planning and metadata overhead increase over time.
3. A teammate proposes solving the small files problem by writing **one giant 2 TB Parquet file** instead. Would that solve the problem? What new issues might it introduce?
> 	Creating one giant 2 TB file would solve the small file problem, but introduce other problems, such as reduced parallelism, longer recovery if a task fails, increased latency, and increased memory utilization.
	- Parallelism decreases because many executors sit idle while few perform the heavy lifting.
4. If small files are bad, why doesn't Spark automatically combine them into larger files every time it writes data?
> 	Automatically combining files isn't always desirable because the optimal number and size of output files depends on the workload. Spark doesn't know how the data will be queried later, how much parallelism future jobs need, or whether downstream systems expect a particular partitioning scheme. Compaction is often treated as a separate optimization step so engineers can balance write performance, read performance, and operational requirements.

## 23. Spark Performance Tuning

- Performance tuning matters because data pipelines need to be **scalable**. Specifically, they need to be able to handle increasing data load while maintaining optimal performance.
- Diagnosing performance issues in Spark involves a multi-step approach:

### Step 1: Don't Guess
- Suppose you're asked: "Your Spark job that normally takes 10 minutes is now taking an hour. How would you investigate?"
	- The **wrong** answer is to immediately propose a solution, such as adding more executors.
	- A better approach is to start by **investigating** the issue. For example: "First, I'd identify where the bottleneck is."

### Step 2: Check The Spark UI
- The Spark UI can help answer questions such as:
	- Which stage is slow?
	- Which tasks are slow?
	- Are all executors busy?
	- Is there excessive shuffle?
	- Is one task much slower than the others?
	- Are tasks spilling to disk?
	- Is CPU low while one task is still running?

| Symptom                          | Possible Cause                                                       |
| -------------------------------- | -------------------------------------------------------------------- |
| One task much slower             | Data skew                                                            |
| Many tiny tasks                  | Too many partitions                                                  |
| Few tasks, idle executors        | Too few partitions                                                   |
| Huge shuffle read/write          | Wide transformations                                                 |
| Same lineage repeatedly executed | Missing cache                                                        |
| Out of memory                    | Broadcast too large, excessive caching, insufficient executor memory |
| Thousands of tiny output files   | Too many partitions before write                                     |

- **Example 1:**
	- Spark UI shows:
		```
		Stage 4
		
		Shuffle Read
		
		900 GB
		```
		- Seeing this should immediately raise the question: "Why am I shuffling almost a terabyte?"
		- Possible causes include:
			- Unnecessary joins
			- Unnecessary repartitioning
			- Wrong partition key
			- Broadcast join opportunity
- **Example 2:**
	- Spark UI shows:
		```
		Tasks
		
		1 sec
		
		1 sec
		
		1 sec
		
		180 sec
		```
		- This is most likely caused by data skew.
- **Example 3:**
	- Spark UI shows:
		```
		32 Executor Cores
		
		8 Tasks
		```
		- There are two few partitions in the dataset.
- **Example 4:**
	- Spark UI shows:
		```
		200,000 Tasks
		
		Average Duration
		
		40 ms
		```
		- There are too many partitions. This can lead to excessive scheduling and orchestration overhead.

### Optimization Checklist
When diagnosing a slow Spark job, walking through the following checklist can help identify the root cause:
1. Is the workload balanced?
	- Look for:
		- Data skew
		- Uneven partition sizes
2. Is there unnecessary shuffling?
	- Can I:
		- Broadcast?
		- Filter early?
		- Partition differently?
3. Am I re-computing expensive work?
	- Would caching help?
4. Are partitions sized appropriately?
	- Too many: `coalesce()`
	- Too few: `repartition()`
5. Is memory a bottleneck?
	- Look for:
		- Disk spill
		- Garbage collection
		- Executor OOM
		- Broadcast size
6. Is the execution plan reasonable?
	- Use `df.explain()` and review:
		- Join strategy
		- Broadcast
		- Number of exchanges (shuffles)

### Importance of `explain()`
- `explain()` allows Spark to tell you how it plans to execute your query.
- For example you might discover `SortMergeJoin` when you expected `BroadcastHashJoin`.
- That's an immediate clue that Spark isn't using the join strategy you expected.
- **Real Data Engineering Example:**
	- Imagine you look at the Spark UI and discover:
		```
		Shuffle
		
		5 TB
		```
		- 5 TB of data is being shuffled. Why?
		- The investigation reveals a 10 MB table has grown to 3GB.
		- Spark can no longer broadcast it to all executors.
		- The execution plan changes from a broadcast join to a shuffle-based join, dramatically increasing network traffic.

### Tradeoffs
Performance tuning is about balancing resources. Sometimes improving one area hurts another.

| Optimization     | Tradeoff                 |
| ---------------- | ------------------------ |
| More partitions  | More scheduling overhead |
| Fewer partitions | Less parallelism         |
| Broadcast join   | More executor memory     |
| Cache            | Consumes memory          |
| Repartition      | Shuffle cost             |
| Coalesce         | Reduced parallelism      |

### Mental Model
```
Slow Spark Job
        │
        ▼
Check Spark UI
        │
        ├── One slow task?
        │       ▼
        │   Data Skew
        │
        ├── Large shuffle?
        │       ▼
        │   Broadcast?
        │   Better partitioning?
        │
        ├── Repeated computation?
        │       ▼
        │   Cache
        │
        ├── Idle executors?
        │       ▼
        │   Too few partitions
        │
        ├── Millions of tiny tasks?
        │       ▼
        │   Too many partitions
        │
        └── Unexpected join strategy?
                ▼
            explain()
```
- Think of Spark tuning like diagnosing a car. You don't replace the engine because the check-engine light came on. Instead you:
	1. Observe the symptoms.
	2. Diagnose the cause.
	3. Apply the appropriate fix.
	- Spark tuning follows the same principle.

### Common Interview Questions
1. How do you optimize a slow Spark job?
> 	I wouldn't optimize blindly. I'd first use the Spark UI and execution plan to identify the bottleneck. I'd look for symptoms like data skew, excessive shuffling, poor partition sizing, repeated re-computation, or memory pressure. Then I'd choose the appropriate optimization, such as broadcasting a small table, caching reused DataFrames, adjusting partitions, or addressing skew. The right optimization depends on the specific bottleneck.
2. A Spark job that normally completes in **12 minutes** suddenly starts taking **45 minutes** after a new feature is deployed. Walk me through how you would investigate the issue before making any changes.
> 	First, I would check the Spark UI and look at which tasks are running slowly. I'd also check to see if all executors are busy. If just one of the tasks is running slowly and all other executors are idle, the most likely cause would be data skew. If this was not the case, I'd look for excessive shuffling or repeated expensive computations. If this was the case, I'd look into repartitioning or caching. Since the slowdown happened immediately after a new feature was deployed, I'd first compare what changed. Did the feature introduce a new join, additional aggregations, repartitioning, or significantly increase the amount of data being processed? Then I'd use the Spark UI and `explain()` to see whether the execution plan changed.
	- Since the changes happened **after a new feature release**, the investigation should focus on what changes that feature introduced.
3. You inspect the Spark UI and notice:
	- Most executors are idle near the end of the job.
	- One task is still running.
	- Shuffle read for that task is much larger than the others.
	- What would you suspect? How would you confirm your hypothesis? What would you consider doing to improve performance?
> 	The most likely cause for this scenario would be data skew. All tasks are finishing very quickly while one task is taking very long because it must process a lot of data. To confirm this hypothesis, I'd check the Spark UI for tasks execution times and look for an outlier among the data. If my hypothesis was confirmed, I'd consider repartitioning as a potential solution. Depending on the root cause, I might repartition the data, choose a better partition key, pre-aggregate earlier in the pipeline, use salting if a single key is dominating, or enable Adaptive Query Execution if appropriate.
	- Try to mention more than one solution to data skew, besides repartitioning. Don't forget about Adaptive Query Execution.