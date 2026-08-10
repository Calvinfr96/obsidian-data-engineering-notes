# Problem

## Business Context

XYZ Retail is a global retailer operating across both physical stores and digital commerce channels. The company faces increasing challenges integrating large-scale, high-velocity data from multiple systems — resulting in delayed analytics, redundant storage, and fragmented insights.

To remain competitive and data-driven, XYZ Retail aims to modernize its data infrastructure by implementing a cloud-based data lake on AWS or Snowflake that enables real-time insights, unified reporting, and advanced analytics across all business units.

## Business Problem

- Current reporting and analytics processes are siloed across POS, ERP, and e-commerce systems, leading to:
	- Delayed decision-making due to overnight batch ETL and slow data availability.  
	- Inconsistent metrics across departments (sales, marketing, inventory).  
	- High maintenance costs of on-prem systems and multiple ETL tools.  
	- Limited scalability to support new regions, mobile apps, and IoT devices.  
	- Compliance gaps in managing PII across disjointed systems.

## Objectives

- Design an end‑to‑end future‑state architecture diagram integrating a centralized lake, real‑time streaming, and self‑service analytics. Explicitly map the technical and business requirements to technology choices, benefits, and risk mitigations.
	- **Centralized Data Lake**: Consolidate all retail, CRM, and operational data into an AWS or Snowflake based data lake.  
	- **Real-time Ingestion and transformation**: Stream POS , App and e-commerce events in near real-time for timely insights.  
	- **Unified Governance**: Apply centralized security, access control, and data cataloging.  
	- **Advanced Analytics Enablement**: Support predictive analytics, customer segmentation, and ML workloads.  
	- **Self-Service BI**: Combine in-store and online behavior for 360° customer understanding.
    
## Additional Technical & Business Requirements

- Data Sources: 
	- Point of Sale (POS) System Data : Capture in-store transactions and customer interactions.
	- Mobile / Web App Data : Capture online user behaviour and interactions with app features.
	- E-commerce Platform (Shopify-like) : Capture web store transactions and customer engagement.
	- Enterprise Resource Planning (ERP) System : Track operational data—inventory, supply chain, finance, and procurement.
	- Customer Relationship Management (CRM) System : Track leads, customers, and marketing engagement.  
- Daily Volume : 5–10M rows/day (~2 TB/day)  
- Data Format : JSON(APIs), Transactional Data
- Real Time Feeds : POS, Apps and E-Commerce Orders
- Batch Feeds : ERP and CRM datasets  
- Compliance: PII (email, phone, address) → encryption, masking, Role-Based Access Control (RBAC), audit trails
- Growth (1‑3 yrs): 3 – 5 × via new regions & IoT/mobile data
- Existing use‑cases: abandoned‑cart recovery, store‑performance dash, inventory optimization, customer segmentation
- Primary users: business analysts, inventory managers, data scientists, marketing analysts
- Success metrics: report latency ↓ 24 h → 1 h; % self‑service reports; accuracy of real‑time inventory alerts; CSAT uplift

# Problem Breakdown

## Business Problems

- The company currently has:
	- siloed data
	- overnight batch ETL
	- inconsistent reports
	- expensive on-prem systems
	- poor scalability
	- PII compliance issues
- These become your design goals.

## Functional Requirements

- The prompt tells us exactly what must exist:
	- Centralized data lake
	- Real-time ingestion
	- Batch ingestion
	- Unified governance
	- Analytics
	- Machine Learning
	- Self-service BI

## Non-Functional Requirements

- Also provided:
	- 2 TB/day
	- 5–10 million rows/day
	- 3–5x growth
	- Report latency reduced from 24 hours to 1 hour
	- Encryption
	- RBAC
	- Audit logging
	- Support multiple users
- Those determine scalability decisions.

# Solution

![](/photos/_solution_1_architecture_diagram.png)

## Data Ingestion Layer

### Streaming Ingestion
- POS, Mobile App, and E-commerce Platform all require streaming.
- **API Gateway**:
	- Acts as the secure front door.
	- Responsibilities
		- Authentication
		- Throttling
		- Routing
		- HTTPS endpoint
- **Lambda**:
	- Before the data is stored, we want to:
		- Validate JSON
		- Reject malformed events
		- Mask PII
		- Enrich records
		- Handle errors
		- Log failures
- **Kinesis**:
	- Buffering
	- Durability
	- Ordering
	- Multiple consumers
	- Real-time processing
- **Firehose**:
	- Firehose batches events efficiently before writing them into storage.
	- Benefits:
		- Automatic batching
		- Retries
		- Compression
		- Direct S3 delivery

### Batch Ingestion
- ERP and CRM don't require streaming.
- Solution:
	```
	ERP
	
	↓
	
	Glue
	
	↓
	
	S3
	```
	- Glue provides:
		- Managed ETL
		- Scheduling
		- Schema inference
		- Scalable Spark jobs

## Storage / Processing Layer (Data Lake)

- The solution uses the Medallion architecture to store data in three distinct layers within Redshift:
	```
	Bronze
	
	↓
	
	Silver
	
	↓
	
	Gold
	```
- **Bronze**:
	- Raw data.
	- No transformations.
	- Keep original records.
	- Used for:
		- Replay
		- Debugging
		- Auditing
- **Silver**:
	- Cleaned.
	- Examples:
		- Remove duplicates
		- Normalize schema
		- Parse timestamps
		- Standardize customer IDs
- **Gold**:
	- Business ready.
	- Examples:
		```
		Daily Sales
		
		Inventory
		
		Customer Lifetime Value
		
		Top Products
		
		Store Revenue
		```
		- Now, dashboards don't need heavy transformations.

## Analytics

- **Athena**:
	- **Run SQL queries on data stored in S3**.
	- Business analysts can immediately query data.
	- No infrastructure required.
- **QuickSight**:
	- **Create dashboards**.
	- Examples:
		- Store performance
		- Inventory
		- Sales
		- Marketing
- **SageMaker**:
	- **Used for ML**.
	- Possible interview examples:
		- Demand forecasting
		- Recommendation systems
		- Customer segmentation
		- Fraud detection

## Governance

- Don't skip this part of the analysis. Interviewers love hearing about it.
- The prompt explicitly requires:
	- Encryption
	- Masking
	- RBAC
	- Auditing
- The solution includes:
	- IAM:
		- RBAC
	- CloudWatch:
		- Monitoring
		- Logging
- You could also mention:
	- SSE-KMS encryption
	- Lake Formation permissions
	- Glue Data Catalog
	- CloudTrail auditing

## Orchestration

- The solution includes Managed Workflows for Apache Airflow (MWAA):
	```
	ERP load
	
	↓
	
	Glue Job
	
	↓
	
	Silver
	
	↓
	
	Gold
	
	↓
	
	Refresh dashboard
	```
	- Airflow automates the entire pipeline and frees engineers from having to pull manual triggers.
	- This saves a lot of time and is much less error-prone.

## Continuous Integration and Deployment (CI/CD)

- The solution includes:
	- CodeBuild
	- Docker
	- Elastic Container Service (ECS)
- These automate deployment of ingestion services and processing components.

## How The Architecture Solves Business Requirements

| Business Problem       | Solution                               |
| ---------------------- | -------------------------------------- |
| Overnight ETL          | Kinesis + Firehose streaming           |
| Siloed data            | Central S3 data lake                   |
| Inconsistent metrics   | Bronze → Silver → Gold transformations |
| Poor scalability       | Managed AWS services                   |
| Compliance gaps        | IAM, encryption, masking, logging      |
| Self-service reporting | Athena + QuickSight                    |
| Predictive analytics   | SageMaker                              |

## Redshift vs. Snowflake

- **Choose Redshift if**:
	- You're already invested in AWS.
	- You want tight integration with S3, Glue, IAM, and Redshift Spectrum.
	- Workloads are relatively predictable.
	- Your team is comfortable tuning clusters.
- **Choose Snowflake if**:
	- You want minimal operational overhead.
	- Many teams will share the platform.
	- Workloads are bursty or unpredictable.
	- Secure data sharing and governance are priorities.

## Questions

### Why Kinesis Data Stream and Firehose for ingestion? Why not just go directly to S3?
- Kinesis gives us:
	- Buffering
	- Durability
	- Ordering
	- Multiple consumers
	- Real-time processing
- This allows multiple analytics applications to consume the stream simultaneously.

### Why Redshift? Why not DynamoDB? Why not write the data back to S3?
- Redshift wasn't chosen simply as a data store, it was chosen **basen on the way the data would be queried**. The solution places Bronze, Silver, and Gold layers in Redshift (or Snowflake).
- S3 acts as the **staging layer** before data is stored in Redshift.

#### Option 1: Redshift (Solution)
- Think about the primary users from the requirements:
	- Business analysts
	- Inventory managers
	- Marketing analysts
	- Data scientists
- What do all of them want?
	- They want to query the data using SQL:
		```sql
		-- Example 1
		SELECT
			store_id,
			SUM(sales)
		FROM gold.sales
		GROUP BY store_id;
		
		-- Example 2
		SELECT
			customer_segment,
			AVG(order_total)
		```
		- These are all examples of **analytical (OLAP)** queries.
- Redshift optimizes for:
	- Scanning billions of rows
	- Joins across large tables
	- Aggregations
	- Columnar storage
	- BI dashboards
- This matches the stated use cases like store performance dashboards, inventory optimization, customer segmentation, and reducing report latency

#### Option 2: DynamoDB:
- DynamoDB is a completely different type of database.
- It's optimized for:
	- Very fast point lookups
	- Key-value access
	- Millisecond latency
	- Transactional workloads **(OLTP)**
- DynamoDB would perform poorly because:
	- No efficient joins
	- Limited aggregation
	- Designed around partition keys
	- Not intended for large analytical scans
- It's the wrong tool for a data warehouse.

#### Option 3: Another S3 Bucket:
- This is actually a very common architecture:
- Instead of:
	```
	Bronze
	↓
	
	Silver
	
	↓
	
	Gold
	
	↓
	
	Redshift
	```
- Many companies do:
	```
	Bronze (S3)
	
	↓
	
	Silver (S3)
	
	↓
	
	Gold (S3)
	
	↓
	
	Athena
	```
- This is often called a **lakehouse** architecture.
- Likely reasons the solution chose Redshift:
	- **Faster BI queries**:
		- Suppose the Gold layer contains 300 TB of data.
		- Athena needs to scan files from S3.
		- Redshift stores:
			- Column statistics
			- Compressed columnar data
			- Optimized sort keys
			- Distribution keys
		- This usually produces much faster dashboard performance for repeated analytical queries.
	- **Better concurrency**:
		- Imagine:
			- 50 analysts
			- 20 dashboards
			- 10 data scientists
		- All of them query the data simultaneously.
		- Redshift is designed for many concurrent analytical workloads.
	- **Materialized tables**:
		- Gold data often contains:
			- Sales summaries
			- Customer dimensions
			- Inventory facts
		- These fit naturally into a warehouse schema.
- **Could S3 + Athena still work**?
	- Absolutely. An example architecture would look like:
		```
		Raw
		
		↓
		
		S3 Bronze
		
		↓
		
		S3 Silver
		
		↓
		
		S3 Gold (Parquet)
		
		↓
		
		Athena
		```
	- Advantages:
		- Cheaper storage
		- Virtually unlimited scalability
		- Serverless
		- No cluster management
- How to justify Redshift over DynamoDB in an interview:
> 	Because our primary workload is analytical rather than transactional. Business analysts need to run complex SQL queries involving joins and aggregations across terabytes of historical retail data. Redshift is a columnar data warehouse optimized for OLAP workloads, whereas DynamoDB is a key-value database optimized for low-latency operational lookups.
- How to justify not keeping everything in S3 in an interview:
> 	That's a valid alternative. For a cost-sensitive or fully serverless architecture, I'd keep the Bronze, Silver, and Gold layers in S3 using Parquet and query them with Athena. I'd choose Redshift when I expect heavy BI usage, frequent complex joins, high query concurrency, or predictable analytical workloads that benefit from a dedicated warehouse.
	- Redshift should be chosen for:
		- Heavy BI usage
		- Frequent complex joins
		- **High query concurrency**
		- Predictable analytical workloads