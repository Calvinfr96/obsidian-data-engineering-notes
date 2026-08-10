# Problem

## Business Context

MediCore Health operates multiple hospitals and diagnostic centers. The company’s on-premises SQL Server–based data warehouse is struggling to scale with the increasing data volume from Electronic Health Records (EHR), billing, and appointment systems.

Your task is to design and prototype a cloud-based data warehouse solution leveraging AWS/Azure (Cloud Platform) and Snowflake (Data Warehouse) that modernizes MediCore’s analytics infrastructure — improving performance, scalability, and compliance with healthcare data regulations.

## Business Problem

- MediCore Health currently faces:
	- Performance and Scalability: The existing on-premises SQL Server data warehouse struggles with the daily influx of data (~1 TB/day). Query performance degrades significantly as data grows, and scaling infrastructure requires costly hardware upgrades.
	- Manual and Error-Prone ETL: Data ingestion and transformation rely on SSIS workflows and manual scripts. Frequent data source changes (new hospital systems) result in job failures and inconsistent datasets.
	- Data Quality and Governance Gaps: Missing or duplicate patient IDs, inconsistent billing codes, and schema drift cause reporting inaccuracies. There’s no central metadata catalog or lineage tracking.
	- Regulatory and Compliance Pressure: MediCore must maintain HIPAA compliance, including PHI encryption, access auditing, and data retention policies. The legacy system lacks fine-grained access control and audit trails.
	- Operational Bottlenecks: Monthly and quarterly reports for regulatory bodies take 6–8 hours to generate, often missing submission deadlines.
    

## Objectives

- MediCore Health aims to modernize its data warehouse architecture to a cloud-native, scalable, and secure analytics platform. Explicitly map the technical and business requirements to technology choices, benefits, and risk mitigations.
	- Cloud Migration: Move the existing warehouse to Snowflake for scalability and elasticity.  
	- Automated Data Pipelines: Replace SSIS jobs with orchestrated ELT pipelines for scheduling, dependency management, and error handling.
	- Data Quality Framework: Integrate validation and profiling checks to ensure high-quality, trusted datasets.  
	- Compliance & Security: Encrypt all PHI data at rest and in transit, use role-based access control (RBAC), and enable audit logging for all data queries.  
	- Self-Service Analytics: Empower analysts with secure, near-real-time dashboards, connected directly to curated data models.    

## Additional Technical & Business Requirements

- Data Sources: 
	- EHR (Electronic Health Records) systems : is the central system that stores a patient’s complete medical history, diagnoses, treatments, medications, allergies, and encounter details across visits.
	- Billing Systems (Payments and Claims Data) :  handles all financial transactions related to patient care.
	- Appointment Systems (Schedules, Bookings, and Wait Times) : manages booking and allocation of patient visits to doctors, departments, and facilities (e.g., radiology, labs, surgery).
	- Lab Information Systems (LIS)  : manages the ordering, tracking, and results of diagnostic tests — blood work, imaging, pathology, etc.
- Data Frequency:
	- EHR Systems: Batch ingestion daily.
	- Billing Systems: Incremental CDC (change data capture) ingestion daily.
	- Appointment Systems: Daily batch.
	- Lab Information Systems: Near real-time ingestion.
- Daily Volume : ~1 TB/day from multiple hospitals  
- Data Format : CSV, JSON, HL7 (for medical data exchange)
- Ingestion Frequency : Daily batch ingestion; near real-time ingestion for lab results
- Compliance: HIPAA-compliant setup — encryption (AES-256), masking, access control, and audit logs
- Success metrics: Report generation time ↓ from 6 hours → < 30 minutes; Data availability ↑ to 99.9%; Quality score ≥ 95%

# Problem Breakdown

## Business Problems

- Performance and Scalability:
	- The existing on-premises SQL Server data warehouse struggles with the daily influx of data.
	- Query performance degrades significantly as data grows.
	- Scaling infrastructure requires costly hardware upgrades.
- Manual and Error-Prone ETL:
	- Data ingestion and transformation rely on SSIS workflows and manual scripts.
	- Frequent data source changes (new hospital systems) result in job failures and inconsistent datasets.
- Data Quality and Governance Gaps:
	- Missing or duplicate patient IDs, inconsistent billing codes, and schema drift cause reporting inaccuracies.
	- There’s no central metadata catalog or lineage tracking.
- Regulatory and Compliance Pressure:
	- MediCore must maintain HIPAA compliance, including PHI encryption, access auditing, and data retention policies.
	- The legacy system lacks fine-grained access control and audit trails.
- Operational Bottlenecks: Monthly and quarterly reports for regulatory bodies take 6–8 hours to generate, often missing submission deadlines.

## Functional Requirements

- Move the existing warehouse to Snowflake for scalability and elasticity.
- Replace SSIS jobs with automated, orchestrated ELT pipelines for scheduling, dependency management, and error handling.
- Integrate validation and profiling checks to ensure high-quality, trusted datasets.
- Encrypt all PHI data at rest and in transit, use role-based access control (RBAC), and enable audit logging for all data queries.
- Empower analysts with secure, near-real-time dashboards, connected directly to curated data models.

## Non-Functional Requirements

- Daily Volume : ~1 TB/day from multiple hospitals  
- Data Format : CSV, JSON, HL7 (for medical data exchange)
- Ingestion Frequency : Daily batch ingestion; near real-time ingestion for lab results
- Compliance: HIPAA-compliant setup — encryption (AES-256), masking, access control, and audit logs
- Success metrics: Report generation time ↓ from 6 hours → < 30 minutes; Data availability ↑ to 99.9%; Quality score ≥ 95%

## Data Sources

- EHR (Electronic Health Records) systems : is the central system that stores a patient’s complete medical history, diagnoses, treatments, medications, allergies, and encounter details across visits.
	- Batch ingestion daily.
- Billing Systems (Payments and Claims Data) :  handles all financial transactions related to patient care.
	- Incremental CDC ingestion daily.
- Appointment Systems (Schedules, Bookings, and Wait Times) : manages booking and allocation of patient visits to doctors, departments, and facilities (e.g., radiology, labs, surgery).
	- Daily batch.
- Lab Information Systems (LIS)  : manages the ordering, tracking, and results of diagnostic tests — blood work, imaging, pathology, etc.
	- Near real-time ingestion.

# Brainstorming

## Ingestion Layer

- Ingestion layer must support daily batch ingestion with CDC, as well as real time streaming.

1. **AWS DMS (Most Common)**:
	- AWS **Database Migration Service (DMS)** is usually the first service to consider when the source is a **relational database**.
	- Typical architecture:
		```
		MySQL / PostgreSQL / Oracle
		          │
		          │ CDC
		          ▼
		       AWS DMS
		          │
		          ▼
		   S3 / Kinesis / Redshift
		          │
		          ▼
		   Glue / Athena / EMR
		```
	- DMS can perform Full Load + CDC, meaning:
		```
		Initial snapshot
		      ↓
		Existing data copied
		      ↓
		CDC starts
		      ↓
		INSERT / UPDATE / DELETE events
		      ↓
		Target
		```
		- This is useful when you need to migrate an existing database and then continuously synchronize subsequent changes.
2. **AWS MSK (Kafka-Based CDC)**:
	- If your architecture already uses Kafka, you can use **Amazon MSK** with CDC tools such as Debezium.
	- This is particularly useful when **multiple downstream systems need the same change events**.
	- Typical architecture:
		```
		Database
		   │
		   ▼
		Debezium
		   │
		   ▼
		Amazon MSK
		   │
		   ├──► S3
		   ├──► OpenSearch
		   ├──► Redshift
		   └──► Other consumers
		```
3. **Kinesis Data Streams**:
	- Kinesis is a good choice when you need **real-time, high-throughput event streaming** without operating Kafka yourself.
	- Useful for **AWS-native** streaming.
	- Typical architecture:
		```
		Source
		  │
		  ▼
		CDC producer
		  │
		  ▼
		Kinesis Data Streams
		  │
		  ├──► Lambda
		  ├──► Glue Streaming
		  ├──► Firehose
		  └──► Custom consumers
		```
4. **AWS Glue (Incremental Processing)**:
	- **AWS Glue** is more focused on the processing side than CDC capture itself.
	- Glue can transform CDC records, deduplicate them, apply business logic, and maintain curated datasets.
	- Typical architecture:
		```
		Database
		   │
		   ▼
		DMS CDC
		   │
		   ▼
		S3 Raw Zone
		   │
		   ▼
		Glue
		   │
		   ▼
		S3 Curated Zone / Redshift
		```

### CDC Summary
- Don't confuse **incremental ingestion** with **CDC**. CDC is a mechanism for capturing incremental changes.
- **Incremental ingestion** can simply mean:
	```sql
	WHERE updated_at > last_processed_timestamp
	```
- CDC captures actual database changes, including:
	- `INSERT`
	- `UPDATE`
	- `DELETE`
- CDC is generally preferable when you need: 
	- Low latency
	- Deletes
	- Exact change history
	- Don't have a reliable `updated_at` column.
- Common Pattern:
	```
	                 Initial load
	Database ──────────────────────────► S3
	   │
	   │ CDC
	   ▼
	 AWS DMS
	   │
	   ▼
	 Incremental changes
	   │
	   ▼
	 S3 / Kinesis / MSK / Redshift
	```
	- The initial load goes directly to S3.
	- Subsequent incremental changes are routed through DMS to S3 / Kinesis / MSK / Redshift.
	- DMS captures changes by reading the database commit logs.
		- The source database's transaction log is the source of truth for CDC—not `updated_at`.
	- When reading the logs, DMS remembers its position by maintaining **CDC state/checkpoint information**, essentially recording how far it has successfully processed in the source transaction stream.
	- A CDC system generally needs to account for **at-least-once delivery/reprocessing**. If DMS restarts around a checkpoint boundary, downstream consumers may potentially see a change again.
	- Therefore, a robust architecture usually makes the **target/application of CDC idempotent**.
	- For example:
		```
		DMS
		 │
		 │ CDC event
		 ▼
		S3 / Kinesis
		 │
		 ▼
		Processing
		 │
		 ▼
		MERGE/UPSERT using primary key
		```
- Mental Model:
	- Initial load = copy the existing data.
	- CDC = continuously read the database change log.
	- Checkpoint = remember how far DMS has processed that log.
	- Idempotency = protect downstream systems from reprocessing.

### Real-Time Streaming With DMS
- **AWS DMS can be used for near-real-time CDC streaming**, but there's an important nuance:
	- DMS is a CDC replication service, not a general-purpose streaming platform like Kafka or Kinesis.
- Typical architecture:
	```
	Source Database
	     │
	     │ Transaction log
	     ▼
	  AWS DMS
	     │
	     │ CDC events
	     ▼
	┌────┴─────────┐
	│              │
	S3          Kinesis
	│              │
	▼              ▼
	Data Lake   Streaming
	```
- DMS continuously reads the source database's transaction log, so there is no need to regularly query the database for changes. There will still be **some replication latency**—typically seconds or potentially more depending on workload, network, task configuration, and target processing—so I'd call it **near-real-time**, rather than strict zero-latency real-time.

| Requirement                      | Better choice                       |
| -------------------------------- | ----------------------------------- |
| Replicate database changes       | **DMS**                             |
| Database → S3 CDC                | **DMS**                             |
| Database → Redshift CDC          | **DMS**                             |
| Database → Kinesis               | **DMS + Kinesis**                   |
| General event streaming          | **Kinesis**                         |
| Kafka ecosystem / many consumers | **MSK**                             |
| Complex stream processing        | **Kinesis/MSK + stream processing** |

- For example, if the requirement was: "Whenever an order changes in PostgreSQL, propagate that change to several downstream services within seconds." A possible architecture could look like:
	```
	PostgreSQL
	    │
	    │ WAL / CDC
	    ▼
	   DMS
	    │
	    ▼
	Kinesis Data Streams
	    │
	    ├──► Order Service
	    ├──► Analytics
	    ├──► Notifications
	    └──► Data Lake
	```
	- Here **DMS handles database CDC**, while **Kinesis handles the actual event-stream distribution**.
- Conclusion:
	- DMS supports near-real-time CDC, and it can be part of a real-time streaming architecture.

### Batch Processing With DMS
- DMS can be used to support both batch processing and near-real-time streaming. However, it's important to distinguish **what DMS itself does** from what the rest of the pipeline does.
- DMS supports two main ingestion modes:
	- Full Load (Batch-Like):
		- DMS can copy existing data from a source to a target:
			```
			Source DB
			   │
			   │ Full Load
			   ▼
			  DMS
			   │
			   ▼
			S3 / Redshift / Target DB
			```
			- For example, you might have 500 million existing rows. DMS reads and transfers that existing dataset. That's essentially a **bulk/batch ingestion operation**.
	- CSC (Continuous):
		- After the initial load, DMS can continue capturing changes:
			```
			             Full Load
			Database ───────────────► DMS ───► Target
			   │
			   │
			   └── CDC ──────────────► DMS ───► Target
			```
		- A very common architecture is:
			```
						  INITIAL
			Database ───────────────► DMS ─────► S3
			   │
			   │
			   │ CONTINUOUS CDC
			   └────────────────────► DMS ─────► S3
			```
- If you only wanted to process incoming changes in batches once every hour, not in real-time, you could architect something like:
	```
	Database
	   │
	   │ CDC
	   ▼
	DMS → S3
	       │
	       │ every hour
	       ▼
	     Glue
	       │
	       ▼
	   Data Warehouse
	```
	- Here **DMS is continuously capturing changes**, while **Glue processes them in batches** every hour.
- If the requirements were simply: "Every night, find all rows that changed during the day and load them."
	- You likely wouldn't need DMS at all. A timestamp/watermark approach should be sufficient:
		```sql
		SELECT *
		FROM orders
		WHERE updated_at > :last_watermark
		  AND updated_at <= :current_watermark;
		```

|Requirement|Approach|
|---|---|
|One-time bulk migration|**DMS Full Load**|
|Continuous database changes|**DMS CDC**|
|Hourly/daily batch processing of CDC data|**DMS → S3 → Glue/ETL**|
|Hourly incremental extraction using timestamp|**ETL + watermark**|
|Real-time event streaming|**DMS → Kinesis/MSK**|
- **Key idea:** DMS can perform the **initial bulk load** and **continuous CDC**. If you need actual scheduled batch transformations, services such as **AWS Glue** are typically responsible for that part.

### Alternative CDC Implementations (AWS)
| AWS service                                         | Role in CDC                                                                            | Best for                                             |
| --------------------------------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **AWS DMS (Database Migration Service)**            | Captures ongoing database changes from transaction logs and replicates them to targets | **Most common choice** for database CDC              |
| **Amazon Kinesis Data Streams**                     | Ingests and streams change events in real time                                         | High-throughput event streaming                      |
| **Amazon MSK (Managed Streaming for Apache Kafka)** | Kafka-based CDC event pipeline                                                         | Kafka ecosystems and complex streaming architectures |
| **Amazon EventBridge**                              | Routes change events between applications/services                                     | Application-level/event-driven CDC                   |
| **AWS Glue**                                        | ETL/streaming transformations and data integration                                     | CDC pipelines into data lakes/warehouses             |
| **Amazon Redshift**                                 | Can consume CDC data through supported ingestion patterns                              | Analytics/data warehouse targets                     |
| **Amazon Aurora / RDS**                             | Source databases whose logs can be captured by DMS or other CDC tools                  | Relational database sources                          |
| **Amazon DynamoDB Streams**                         | Captures item-level changes in DynamoDB                                                | **DynamoDB CDC**                                     |
| **Amazon Kinesis Data Firehose**                    | Delivers streaming CDC events to destinations such as S3, Redshift, and OpenSearch     | Simplified streaming delivery                        |
| **Amazon S3**                                       | Stores CDC events/snapshots for data lakes                                             | Durable CDC history and downstream processing        |
- Typical Architectures:
	1. Database to Data Warehouse:
		```
		RDS/Aurora
		    ↓
		AWS DMS (CDC)
		    ↓
		S3 / Redshift
		```
	2. Database to Real-Time Applications:
		```
		RDS/Aurora
		    ↓
		AWS DMS
		    ↓
		Kinesis Data Streams
		    ↓
		Lambda / consumers
		```
	3. DynamoDB CDC:
		```
		DynamoDB
		    ↓
		DynamoDB Streams
		    ↓
		Lambda
		    ↓
		Kinesis / S3 / OpenSearch / other services
		```
	4. Kafka-Based CDC:
		```
		Database
		   ↓
		Debezium / AWS DMS
		   ↓
		Amazon MSK
		   ↓
		Kafka consumers / analytics / applications
		```
- Choosing The Best Option:
	- **Need CDC from RDS/Aurora → another database/data warehouse?** → **AWS DMS**
	- **Need CDC from DynamoDB?** → **DynamoDB Streams**
	- **Need a high-volume real-time event pipeline?** → **Kinesis Data Streams**
	- **Already standardized on Kafka?** → **Amazon MSK + Debezium/DMS**
	- **Need to land CDC data in a data lake?** → **DMS → S3**, often combined with Glue
	- **Need application/event routing rather than database-log CDC?** → **EventBridge**
- The key distinction is that **DMS and DynamoDB Streams are CDC mechanisms**, while **Kinesis/MSK/EventBridge are primarily ways to transport and distribute the captured changes**.

## Storage Layer (Data Lake)
- DMS would migrate raw data from the various databases to S3.
- S3 offers server-side encryption using AWS-managed KMS keys for data at rest.

## Transformation Layer (Data Warehouse)
- Data would be loaded into Snowflake from S3.
- Once data lands in Snowflake, it will undergo three levels of transformation using the Medallion architecture:
	- Bronze:
		- Raw data.
		- No transformations.
		- Keep original records.
		- Used for:
		    - Replay
		    - Debugging
		    - Auditing
	- Silver:
		- Cleaned.
		- Examples:
		    - Remove duplicates
		    - Normalize schema
		    - Parse timestamps
		    - Standardize customer IDs
	- Golde:
		- Business-ready, aggregated data.

# Solution

![](/photos/_solution_3_architecture_diagram.png)

## Separate Batch and Near-Real-Time Ingestion

- EHR and Appointment systems data is retrieved in daily batches, while Billing and Lab systems data is retrieved in near-real time. This leads naturally to two separate ingestion pathways.
- This is much better than trying to force every source through the same ingestion mechanism, especially when the sources have different ingestion requirements.

### CDC for Billing Data
- Billing is different from EHR and Appointments because it requires daily, incremental CDC ingestion.
- Instead of repeatedly extracting the entire billing database, it captures changes using the database's transaction log.
- After the initial full load, only changed records flow through DMS to the staging layer.
- DMS supports continuous data replication and seamless migration from on-premises data stores to cloud.
- This reduces unnecessary data movement and helps keep the warehouse current.

### Near-Real -Time Ingestion for Lab Data
- The Lab Information System is the one source that needs near-real-time ingestion.
- The solution gives Kinesis and Kafka as streaming options:
	```
	Lab System
	     ↓
	Kinesis / Kafka
	     ↓
	Firehose / streaming ingestion
	     ↓
	Snowflake
	```
	- This is important because not every requirement needs a real-time architecture.
	- You don't need to turn the entire warehouse into a streaming system just because **one source** needs near-real-time data.

## Data Warehousing

- The problem explicitly says the target is to **move the existing warehouse to Snowflake for scalability and elasticity**.
	- Redshift or S3 could be viable alternatives, but the problem explicitly states data should be stored in Snowflake.
- The reasoning behind choosing Snowflake is:
	- Existing problem:
		```
		On-prem SQL Server
		       ↓
		Hardware limits
		       ↓
		Performance degradation
		       ↓
		Expensive scaling
		```
	- Target solution:
		```
		Snowflake
		       ↓
		Cloud-based warehouse
		       ↓
		Elastic compute
		       ↓
		Scalable analytics
		```
		- Zero infrastructure management
		- Built-in security and governance
		- Advanced analytics/ML
		- Secure data sharing
		- **Multi-cloud flexibility**
- ELT vs. ETL:
	- The objective explicitly states: "Replace SSIS jobs with **orchestrated ELT pipelines**."
	- Traditional ETL:
		```
		Extract
		   ↓
		Transform
		   ↓
		Load
		```
		- Transformation occurs in a staging layer **before** data enters the warehouse.
	- Preferred ELT:
		```
		Extract
		   ↓
		Load
		   ↓
		Transform
		```
		- You first load the data into Snowflake and then perform transformations **using Snowflake's compute**.
		- This fits this architecture very well because Snowflake is being used as the central data platform.

### Bronze Layer
- Data is loaded with minimal transformation.
- The purpose is **preserving the source data**.
- Useful for:
	- Auditing
	- Troubleshooting
	- Reprocessing
	- Lineage
	- Recovering from transformation errors

### Silver Layer
- Here, the data is cleaned and standardized by addressing issues such as:
	- Inconsistent formats
	- Duplicate records
	- Invalid values
	- Schema inconsistencies
	- Standardized codes
- This directly addresses the data quality problems in the prompt.

### Gold Layer
- Analytics-ready data.
- This is where we build business-oriented models.
- For example:
	- Patient utilization
	- Hospital revenue
	- Appointment wait times
	- Lab turnaround time
	- Claims analytics
- The source describes Gold as **aggregated, analytics-ready** data.

### DBT for Transformation
- The Snowflake portion of the solution diagram shows **DBT** between the Snowflake layers.
- This is an important design choice. Instead of putting all transformations into Glue, we leverage Snowflake's compute and use DBT to manage SQL-based transformations.
- Benefits of using DBT for transformations include:
	- Modular SQL-based transformations
	- Automated data lineage/documentation
	- Testing and data quality validation
	- Snowflake integration
	- Observability/governance

## Staging Layer

- The solution introduces an S3 **staging area** between ingestion and Snowflake.
- The staging area is used because it provides a controlled landing point before the data enters the warehouse.
- It can be used for:
	- Validating incoming files/events
	- Temporarily holding data
	- Replaying failed loads
	- Decoupling ingestion from warehouse processing
- The solution uses AWS Glue for the batch sources, before they're placed in the staging area.
- Benefits of using Glue to process the data include:
	- Serverless architecture
	- **Less infrastructure and manual pipeline maintenance**
	- Fully managed ETL
	- Automated schema discovery
	- Data cataloging
	- Scalable transformation
	- Security/compliance support
- Glue also provides data cataloging within the staging area, using a Glue Crawler to automatically scan the data, determine the schema, and create or update metadata tables in the AWS Glue Data Catalog.
	- This metadata can be used for services such as:
		- Amazon Athena
		- Amazon EMR
		- Amazon Redshift
		- JDBC databases
		```
		Staging
		   ↓
		Crawler
		   ↓
		Glue Data Catalog
		```
		- This addresses the governance/metadata problem.

## Data Governance

### Data Quality
- The problem specifically says:
	- Missing/duplicate patient IDs
	- Inconsistent billing codes
	- Schema drift
	- Reporting inaccuracies
- Validation needs to be performed at multiple stages. For example:
	```
	Raw
	 ↓
	Validation
	 ↓
	Silver
	 ↓
	Data quality tests
	 ↓
	Gold
	```
	- Patient ID: `patient_id IS NOT NULL`
	- Duplicate patients: Check uniqueness constrains
	- Billing codes: Validate against expected reference codes
	- Schema: Detect unexpected columns or data type changes
- DBT supports transformations, testing, and data lineage.

### HIPAA Security
- The prompt explicitly requires:
	- Encryption at rest
	- Encryption in transit
	- Masking
	- RBAC
	- Audit logging
- IAM is used for access control for raw data landing in AWS.
- CloudWatch is used for monitroing and logging.

## Orchestration

- The architecture includes MWAA for orchestration. This coordinates pipeline dependencies.
- For example:
	```
	EHR ingestion
	      ↓
	Bronze load
	      ↓
	Data quality checks
	      ↓
	Silver transformation
	      ↓
	Quality checks
	      ↓
	Gold transformation
	      ↓
	Dashboard availability
	```
	- If the Silver transformation fails, Airflow can prevent the Gold transformation from running.
	- That's the value of orchestration here.

## Analytics

- Once data is available in the Gold layer, several BI tools can be used for analytics.
- For example:
	- Snowflake/Snowsight
	- Tableau
	- Power BI
	- Streamlit
- The key requirement is **self-service analytics**. The goal isn't for engineers to manually create every report. Instead, analysts use the curated Gold datasets to create Dashboards using SQL.
- This supports the objective of secure, near-real-time dashboards connected directly to curated data models.
- Snowflake provides scalable warehouse compute, while curated Gold models avoid forcing every dashboard to repeatedly perform expensive transformations.

## Summary

| Decision          | Why?                                         |
| ----------------- | -------------------------------------------- |
| **Snowflake**     | Scalable cloud data warehouse                |
| **Glue**          | Managed batch ingestion/ETL                  |
| **CDC (DMS)**     | Efficient incremental billing ingestion      |
| **Kinesis/Kafka** | Near-real-time Lab ingestion                 |
| **Staging**       | Decouple ingestion from warehouse processing |
| **Bronze**        | Preserve raw data                            |
| **Silver**        | Clean/standardize data                       |
| **Gold**          | Analytics-ready business models              |
| **dbt**           | SQL transformations, testing, lineage        |
| **MWAA**          | Orchestrate dependencies                     |
| **IAM**           | RBAC/security                                |
| **CloudWatch**    | Monitoring/logging                           |
| **BI tools**      | Self-service analytics                       |
- Requirements:
> 	The existing SQL Server warehouse can't keep up with roughly 1 TB/day, the SSIS pipelines are becoming difficult to maintain, data quality and governance are poor, and regulatory reports take 6–8 hours. The target is a cloud-native Snowflake warehouse with automated pipelines, strong data quality and HIPAA controls, and reporting under 30 minutes.
- Data Sources:
> 	I'd first classify the sources based on their ingestion requirements. EHR and appointments are daily batch, billing requires CDC, and the lab system needs near-real-time ingestion.
- Ingestion:
> 	I'd use Glue for the batch sources, CDC for incremental billing changes, and Kinesis or Kafka for the near-real-time lab data.
- Staging:
> 	I'd land the incoming data into a staging area so ingestion is decoupled from downstream warehouse processing.
- Warehouse:
> 	I'd load the data into Snowflake using a Bronze, Silver, and Gold structure.
- Transformation:
> 	I'd use DBT to perform SQL-based transformations and data-quality tests inside Snowflake.
- Governance:
> 	I'd implement RBAC, encryption, masking, cataloging, and audit logging because we're dealing with PHI and HIPAA requirements.
- Analytics:
> 	Finally, analysts and business users can query the curated Gold models through Snowflake and connect BI tools such as Tableau or Power BI.
## Questions

### CDC isn't an AWS service. Which AWS service would perform this function?
- AWS DMS would most likely be used to implement CDC from the on-premises billing database to the staging layer.
	```
	On-prem PostgreSQL / Oracle
	          │
	          │  CDC
	          ▼
	      AWS DMS
	          │
	          ▼
	     S3 / Staging
	          │
	          ▼
	      Snowflake
	```
	- DMS connects to the database and captures changes, such as:
		- INSERT
		- UPDATE
		- DELETE
	- This is much more efficient than repeatedly copying the entire database. This operation would only be performed during the initial full load.
- DMS makes sense because the billing database requires **incremental CDC ingestion**. 

### Could DMS be used to perform all of the ingestion?
- DMS could potentially handle all or most of the ingestion, but it shouldn't be used for every source listed in the design.
- The key is that **DMS is primarily a database migration/replication service**, so it's a particularly good fit when the source is a database and you want full-load + CDC. It isn't a universal ingestion service for every kind of source.
- For MediCore's sources:

| Source       | Requirement    | DMS?                 | My choice     |
| ------------ | -------------- | -------------------- | ------------- |
| EHR          | Daily batch    | **Yes, potentially** | Glue or DMS   |
| Billing      | CDC            | **Yes**              | **DMS**       |
| Appointments | Daily batch    | **Yes, potentially** | Glue or DMS   |
| Lab          | Near real-time | **Possibly**         | Kinesis/Kafka |

- If EHR and Appointments are relational databases, DMS could absolutely replicate their data into the cloud. In fact, that could simplify the architecture:
	```
	EHR ──────────────┐
	Billing ──────────┼──> DMS ──> S3/Staging ──> Snowflake
	Appointments ─────┘
	
	Lab ──> Kinesis/Kafka ────────────────┘
	```
	- DMS could be configured to perform Full Load + CDC on these sources.
	- For example, you might initially migrate all historical EHR data and then use ongoing replication if you wanted the cloud copy to stay current.
- the EHR system produces **CSV/JSON/HL7 files** rather than **exposing a relational database that DMS can replicate**, Glue becomes much more appropriate.
	- DMS is great for moving and replicating database data; Glue is more general-purpose ETL/ingestion.
- Conclusion:  "Why don't you just use DMS for everything?"
> 	We could use DMS for the relational database sources, particularly Billing because we specifically need CDC. But I wouldn't force DMS onto every source. EHR and Appointment data are specified as batch workloads, and Glue gives us more flexibility for file-based ingestion, schema discovery, cataloging, and transformations. For the Lab system, which requires near-real-time ingestion, I'd use a streaming technology such as Kinesis or Kafka. So I'd choose the ingestion mechanism based on the source and its data-delivery requirements rather than trying to standardize everything on DMS.

### Could S3 or Redshift be used in place of Snowflake?
- Both could be used as alternatives, but they aren't **equivalent replacements for Snowflake**.
1. S3 Alternative:
	- S3 could be used to build a **data lake**, instead of a traditional data warehouse:
		```
		Sources
		   ↓
		Ingestion
		   ↓
		S3
		   ├── Bronze
		   ├── Silver
		   └── Gold
		        ↓
		     Athena
		        ↓
		   BI / Analytics
		```
	- This would be attractive if the primary goals were:
		- Inexpensive long-term storage
		- Massive scalability
		- Storing raw data in its original form
		- Supporting multiple processing engines
		- Minimizing infrastructure costs
	- S3 by itself is **object storage, not a data warehouse**. You'd need additional services for things Snowflake provides as the analytical platform, such as SQL query execution, warehouse-style optimization, and management of analytical workloads.
	- Conclusion: "Why use Snowflake instead of S3?"
> 	S3 could absolutely serve as the underlying data lake, but it wouldn't by itself replace the warehouse functionality. We'd need something like Athena or another query engine on top of S3. Since the requirement explicitly calls for modernizing the existing data warehouse and providing fast self-service analytics, Snowflake gives us a dedicated analytical warehouse rather than requiring us to assemble those capabilities around object storage.
2. Redshift Alternative:
	- Redshift is a much more direct Snowflake alternative. You could build:
		```
		Sources
		   ↓
		AWS ingestion
		   ↓
		S3 / Staging
		   ↓
		Redshift
		   ├── Bronze
		   ├── Silver
		   └── Gold
		        ↓
		   Power BI / Tableau
		```
	- Redshift is also a cloud data warehouse, so it can perform the role that Snowflake is playing in this architecture.
	- The major difference is that **Snowflake is explicitly required by the problem**, while Redshift would be your alternative architectural choice.
	- If MediCore were heavily AWS-oriented, Redshift could be a very reasonable choice.
	- Conclusion: "Why use Snowflake instead of Redshift or S3?"
> 	S3 could be used as the underlying data lake, but it isn't a direct replacement for Snowflake's warehouse functionality—we'd need a query engine such as Athena on top of it. Redshift is a much more direct alternative because it's also a cloud data warehouse. If the organization were heavily AWS-centric, I would seriously consider Redshift. However, this particular problem explicitly specifies Snowflake as the target warehouse, with scalability, elasticity, governance, and multi-cloud flexibility being important benefits.

#### Comparing Alternatives

|S3|Redshift|Snowflake|
|---|---|---|---|
|Primary role|Object storage/data lake|Data warehouse|Data warehouse|
|Replaces SQL Server warehouse?|Not by itself|Yes|Yes|
|Requires query engine for analytics?|Yes|No|No|
|Excellent raw-data storage|**Yes**|Less ideal|Possible, but not its primary role|
|SQL analytics|Athena/etc.|**Yes**|**Yes**|
|AWS-native|**Yes**|**Yes**|No|
|Multi-cloud flexibility|N/A|More AWS-centric|**Strong**|
|Reference solution|No|No|**Yes**|
- S3 and Snowflake are often used **together**, with S3 acting as the raw landing zone and Snowflake acting as the data warehouse. The more direct comparison is between Snowflake and Redshift?

### What is the main purpose of the Glue job? Does it perform the encryption / masking of PHI data?
- **The main purpose of the Glue job in this solution is ingestion/ETL for the batch data sources**, not PHI encryption or masking.
- The architecture shows Glue handling the **batch ingestion** from the EHR and Appointment systems into the staging area. The problem specifies those two sources as daily batch workloads, while Billing uses CDC and the Lab system uses near-real-time ingestion.
- Glue's main responsibilities can include:
	- Extracting the source data
	- Converting/loading data into the staging area
	- Schema discovery
	- Data transformation
	- Data quality processing
	- Cataloging metadata
- **The provided solution does not explicitly assign encryption or masking to Glue**. Instead, security is shown as a **cross-cutting concern**, with IAM for role-based access control and the Snowflake layer providing security/governance capabilities.
- **Encryption and Masking are two different things**:
	- Encryption protects data while being stored or transmitted. PHI data would be encrypted at rest in S3. Access would be restricted with IAM RBAC and Bucket Policies.
		- IAM Policy:
			- Identity-based policy (user, group, or role)
			- Controls what the identity can do
			- Grants access to the attached identity
			- Example: "This role can read from S3."
			- Can cover multiple AWS resources
			- Cross-account access requires corresponding resource policy
			- **Use when controlling permissions for an application's/users' identities**
		- Bucket Policy:
			- Resource-based policy (S3 bucket)
			- Controls who can access the bucket/object and what they can do with it.
			- Grants access to other AWS accounts, IAM principals, services, etc.
			- Example: "This bucket allows this role/account to read."
			- Primarily applied to the specific bucket and its objects.
			- Commonly used to grant cross-account access.
			- **Use when controlling controlling access at the bucket/resource level, especially cross-account or service-specific conditions**
	- Masking changes what a user or application is allowed to see. An example of masking would be redacting a user's email address like so: `email = p**************@example.com`.
- You don't necessarily want to permanently destroy the original value during ingestion. For PHI, you'd generally want to carefully control **where the unmasked value exists and who can access it**, rather than simply masking everything in Glue.
	- Unmasked PHI data could exist in the S3 raw landing zone and Snowflake Bronze layer.
	- Sensitive fields could also be kept available in a tightly controlled Silver representation and enforce **column-level/role-based access in Snowflake**, depending on who needs the actual PHI.
	- Raw PHI should be tightly restricted; analytical users should generally work with masked/de-identified data unless they have a legitimate need for the underlying PHI.
- Security is applied **throughout the pipeline**, while masking/de-identification is a data transformation and access-control concern.
- Conclusion: "How is PHI data managed throughout the pipeline?"
> 	I'd retain the raw PHI in a highly restricted, encrypted landing area for auditability and reprocessing. During the Bronze-to-Silver transformation, I'd apply the necessary masking or de-identification, and I'd use Snowflake's access controls to ensure analysts only see the fields they're authorized to access. The exact masking stage isn't specified by the reference architecture, so I'd clarify that based on the organization's HIPAA data-access requirements.

#### Breakdown of Responsibilities
| Component         | Primary responsibility                              |
| ----------------- | --------------------------------------------------- |
| **Glue**          | Batch ingestion + ETL/schema/cataloging             |
| **DMS/CDC**       | Incremental database changes                        |
| **Kinesis/Kafka** | Near-real-time Lab ingestion                        |
| **Snowflake**     | Central warehouse + analytics + security/governance |
| **IAM**           | RBAC/access control                                 |
| **CloudWatch**    | Logging/monitoring                                  |
| **dbt**           | SQL transformations + data quality testing          |
| **MWAA**          | Pipeline orchestration                              |

### How is the data encrypted at rest and in transit?
1. Encryption at rest:
	- Every persistent location containing PHI should be encrypted.
	- For example:
		```
		EHR
		 ↓
		S3 Staging       ← encrypted
		 ↓
		Snowflake Bronze ← encrypted
		 ↓
		Snowflake Silver ← encrypted
		 ↓
		Snowflake Gold   ← encrypted
		```
	- S3 Encryption:
		- For the raw/staging data, I'd use **S3 server-side encryption**, preferably with an AWS KMS customer-managed key if the compliance requirements call for control over key management.
		- Conceptually:
			```
			PHI
			 ↓
			S3
			 ↓
			SSE-KMS
			 ↓
			Encrypted object
			```
		- IAM/KMS policies then determine which roles can access the encrypted data or use the key.
	- Snowflake Encryption:
		- The Snowflake data is also encrypted at rest by default. The solution identifies Snowflake as the central warehouse and specifically calls out security and governance capabilities.
		- Similar to AWS, data in Snowflake is encrypted at rest using Snowflake-managed keys or Customer-managed keys.
2. Encryption in transit:
	- Whenever PHI data travels between components, TLS/HTTPS should be used to encrypt the data.
	- For example:
		```
		EHR
		 │
		 │ TLS
		 ▼
		Glue
		 │
		 │ TLS
		 ▼
		S3
		
		AND
		
		S3
		 │
		 │ TLS
		 ▼
		Snowflake
		 │
		 │ TLS
		 ▼
		BI Tool
		```
	- Similarly, for the near-real-time Lab pipeline:
		```
		Lab System
		    │
		    │ TLS
		    ▼
		Kinesis / Kafka
		    │
		    │ TLS
		    ▼
		Snowflake
		```
	- The goal is that **PHI should never be transmitted as plaintext between services**.
	- **Never confuse encryption with IAM**:
		- Encryption protects the data if someone obtains the underlying storage/network traffic.
		- IAM determines who is allowed to access the data in the first place.
		- They should be used together, but it's important to note they serve two different functions.
	- S3 and Snowflake provide TLS transporting data between components.
- Conclusion: "How does the pipeline handle encryption and masking of PHI data?"
> 	I'd encrypt PHI at rest in every persistent layer, including the S3 landing area and Snowflake, using managed encryption and appropriate KMS/key-management controls. For data in transit, I'd require TLS for communication between the source systems, ingestion services, S3, Snowflake, and downstream analytics tools. Encryption would be combined with least-privilege IAM and Snowflake RBAC so that only authorized roles can access PHI. Finally, I'd mask or de-identify sensitive fields before exposing curated data to general analytics users, and audit access to the data.

### Can the Glue Data Catalog be used by Snowflake?
- Yes, **but with an important distinction**: the Glue Data Catalog can be used as a metadata/catalog source alongside Snowflake, but the provided solution does **not** explicitly explain a Snowflake ↔ Glue Data Catalog integration.
- The Glue Data Catalog primarily provides **metadat about datasets**, such as:
	- Table definitions
	- Columns
	- Data types
	- Locations
	- Schemas
- The catalog is primarily created to address the stated **data quality/governance problem**: MediCore currently has no central metadata catalog or lineage tracking.
- Conclusion:
	- "What is the purpose of the Glue Data Catalog?"
> 	Glue Data Catalog can catalog the AWS-side datasets, particularly the S3 landing data. Snowflake would maintain metadata for the warehouse itself. If we need a unified metadata and lineage experience across both environments, we'd need to explicitly integrate those catalogs or choose a governance/catalog solution that spans both.
		- The Glue Data Catalog describes the raw data in S3.
		- When the raw S3 data is transported to Snowflake, Snowflake preserves metadata describing its own tables.
	- "We already have Glue Catalog. Why do we need another catalog?"
> 	The Glue Data Catalog can serve as the metadata catalog for the AWS-side data, particularly the S3 landing data. Snowflake maintains metadata for the warehouse itself. We could integrate the two if we need a unified catalog, but I wouldn't assume that Glue Catalog automatically becomes Snowflake's catalog. I'd clarify whether the organization's governance requirement is for a single enterprise-wide catalog or separate catalogs for the lake and warehouse.