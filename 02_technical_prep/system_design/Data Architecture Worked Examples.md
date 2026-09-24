---
aliases:
  - "system_design"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/System Design/system_design.md"
imported: 2026-09-24
---

# Data Architecture Worked Examples

Review: [[02_technical_prep/system_design/Distributed Systems Fundamentals|Distributed Systems Fundamentals]] · [[02_technical_prep/system_design/Data Warehousing|Data Warehousing]] · [[02_technical_prep/system_design/Data Processing Systems (Spark)|Data Processing Systems (Spark)]]

## Contents

- [[#Most Common AWS Data Architecture Patterns]]
- [[#Real-Time and Batch Processing AWS Architecture]]
- [[#Data Architecture Modernization]]
- [[#Data lake assessment additions]]
- [[#Modularity, encapsulation, and abstraction]]

## Most Common AWS Data Architecture Patterns
- Event-Driven Data Processing With AWS Lambda, SNS and SQS:
  - AWS S3 (Source) > Amazon Event Bridge Rule > AWS SNS > AWS SQS > AWS Lambda > AWS S3 (Target)
  - SNS and SQS are used as an intermediary to Lambda to avoid processing issues caused by concurrency limitations associated with Lambda. SQS also allows for fault tolerance through the use of a DLQ.
  - SNS is used in addition to SQS to allow the potential for other subscribers to process the same data.
  - Pros:
    - Fully Serverless – No need to manage infrastructure, scaling is automatic.
    - Cost-Efficient – You only pay for what you use, no idle compute cost.
    - Scalable & Reliable – SQS ensures that messages are not lost, even under high loads.
    - Decoupled Architecture – SNS allows multiple subscribers (e.g., other teams or pipelines) to process the same data.
    - Built-in Resilience – Dead Letter Queue (DLQ) helps retry failed processing attempts.
  - Cons:
    - Not Suitable for Large Files – AWS Lambda has a memory limit (6GB) and execution time limit (15 mins).
    - Limited Concurrency – Default concurrent executions are limited (1000 per region).
    - Higher Latency for Bulk Processing – Not ideal for batch jobs; processes data file-by-file.
- Batch Processing Data Pipeline With AWS Glue:
  - AWS S3 (Source) > AWS Glue Catalog + Crawler > AWS Glue Job + Amazon Event Bridge Rule > AWS S3 (Target)
  - The AWS Glue Crawler is used to assist with writing data from S3 to the Glue Catalog.
  - Pros:
    - Ideal for Large Datasets – Can handle GB to TB-scale data.
    - Serverless Spark Execution – Uses Apache Spark, making it highly scalable.
    - Schema Detection – AWS Glue Crawlers automatically infer schema and populate Glue Catalog.
    - Supports Complex Transformations – Can join, filter, and enrich data before storing it.
    - Flexible Scheduling – Can be triggered via AWS Glue Workflow or EventBridge.
  - Cons:
    - Higher Cost for Small Data Loads – Can be expensive for small datasets compared to Lambda.
    - Longer Execution Time – Batch processing may introduce delays before data is available.
    - Learning Curve – Glue jobs require knowledge of PySpark/Scala, which may not be beginner-friendly.
- Serverless Dashboard Powered by AWS Athena:
  - AWS S3 (Source) > AWS Glue Catalog > AWS Athena > Amazon Quicksight Dashboard > Business Users 
  - Pros:
    - Low-Cost Analytics – Athena follows a pay-per-query model, eliminating the need for dedicated servers.
    - Seamless Integration with S3 – Works directly on data stored in Amazon S3 without the need for ETL.
    - No Infrastructure to Manage – Fully managed, no provisioning of servers or clusters.
    - Interactive Dashboards – QuickSight provides easy-to-use BI reporting without heavy setup.
  - Cons:
    - Slower Query Performance on Large Datasets – Querying large raw datasets in S3 can be slow.
    - Limited Query Optimization – Not as performant as Redshift for complex joins across massive datasets.
    - Query Cost Can Add Up – Since Athena charges per query, frequent queries can become costly.
- Migrate a Database With AWS Data Migration Service (DMS):
  - SQL Server (Source) > AWS Schema Conversion Tool and DMS > Amazon RDS (Target)
  - Pros:
    - Supports Heterogeneous Migrations – Can migrate from different database engines (e.g., SQL Server → PostgreSQL).
    - Minimal Downtime – Can perform ongoing replication while the source database is still in use.
    - Automated Schema Conversion – AWS Schema Conversion Tool helps transform database objects.
    - Secure & Managed – Fully managed, ensuring encryption and built-in monitoring.
  - Cons:
    - Not All Schema Conversions Are Automatic – Some complex stored procedures or custom functions may need manual changes.
    - Potential Data Loss for Unsupported Features – Some database-specific features might not translate well to the target database.
    - Network Bandwidth Issues – Large migrations may require significant network bandwidth, slowing down the process.
- Process Streaming Data From Amazon Kinesis Stream:
  - Amazon Kinesis Datastream (Source) > AWS Lambda or Amazon Kinesis Data Analytics or Amazon Kinesis Data Firehose > AWS S3 (Target)
  - Pros:
    - Low-Latency Data Processing – Can process real-time data streams with milliseconds latency.
    - Multiple Processing Methods – Supports Lambda, Kinesis Data Analytics (Apache Flink), and Kinesis Firehose.
    - Event-Driven Architecture – Can trigger actions immediately when new data arrives.
    - Auto-Scaling – Kinesis scales automatically to handle high traffic loads.
    - Supports Multiple Consumers – Different teams can process the same data independently.
  - Cons:
    - Complexity in Managing Streaming Jobs – Requires tuning for shard counts, retention policies, and error handling.
    - Higher Cost for Continuous Streams – Always-on processing incurs higher costs compared to batch solutions.
    - Limited Data Retention – Kinesis only retains data for 24 hours (default), extendable to 7 days.
- Conclusion:
  - For real-time event-driven processing, use AWS Lambda + SNS + SQS.
  - For large-scale batch processing, use AWS Glue.
  - For low-cost, serverless analytics, use Athena + QuickSight.
  - For migrating databases with minimal downtime, use AWS DMS.
  - For real-time streaming analytics, use Amazon Kinesis.
## Real-Time and Batch Processing AWS Architecture
- Scenario:
  - You have a website that sells articles, you want to study the customer behavior, analyze click-stream data and present the results on a real time and a batch processing dashboard.
  - Users access the website via a mobile application or a website.
  - Different micro-services will connect to the data.
  - The architecture can be divided into ingestion, transform, and load stages.
- Ingestion Stage:
  - The data from the mobile application and website is collected in real-time using kinesis data firehose, and this raw data is stored in an S3 bucket.
- Transform Stage:
  - In the transform part, we are using amazon kinesis data analytics and firehose service. The kinesis data analytics service is used to run SQL queries on the stream data from the incoming kinesis firehose service.
- Load Stage:
  - The load part of the pipeline can be further split into 2 parts, real-time processing and batch processing.
  - Real-Time Processing: Every time new data is available after transformation in the kinesis firehose section, the lambda function can be triggered to put the new data into the dynamodb database.
  - Batch Processing: Once the data is transformed in the transform step, it is stored in a separate S3 bucket. We will also create a glue crawler on this bucket to create data catalogue and tables, these glue catalogue tables can then be loaded onto various endpoints.
- Conclusion:
  - An architecture that describes a data lake system that can support different users (Data engineers, product owners, stakeholders, data scientists, BI engineers, and executive managers), for different use cases.
  - The services used in these architectures are highly scalable, and fully managed, thus we don't need to worry about setting up or managing the infrastructure.
## Data Architecture Modernization
- Business Context: Client wants to modernize their data architecture using modern and simpler cloud service offerings. The business wants to implement a single source of truth to enable data interoperability.
- Legacy State:
  - Data ingestion pipelines using AWS Glue Jobs are connected to different types of sources such as MangoDB and Oracle DB.
  - Pipelines are built using AWS Glue Batch and UC4 Jobs.
  - Glue acts as a conveyor belt, moving data from the source into S3 while performing transformations using python scripts.
  - Once the data is in S3, AWS Glue Batch is used to transform and migrate the data into snowflake, where the data undergoes further layers of transformation.
- Challenges:
  - Complex AWS Glue-based architecture.
  - Limited cross-stream collaboration.
  - Multiple documentation and data owners.
  - Absence of unified database/repository.
- Future State:
  - 8 fixed ingestion patterns that cover all use cases.
  - Data contracts for ingestion lead to standardization and inclusion of business units into platform.
  - Core data products designed by platform team in collaboration with business.
  - Delivered data products developed and owned by business teams.
  - Collaboration among engineering and governance teams for data management.
  - Batch data processing is performed using AWS DMS. From DMS, the data is sent to a lambda function through a kinesis stream. The lambda function performs data transformations, then sends the data to S3 through kinesis firehose.
  - Once the data has landed in S3, Snowpipe is used to migrate the data to snowflake, where it undergoes further transformation as it is prepared for analysis. The data in snowflake is organized using the medallion architecture pattern.
  - Real-time data streaming skips DMS and goes straight to kinesis stream.
- A data contract outlines how data is exchanged between two parties. It defines the architecture, format, and rules of exchange in a distributed data architecture.

#

#

#

#

#

## Data lake assessment additions

- Assessing and Validating Data Lake Architecture:
    - Clearly define business objectives and use cases.
    - Understand data-driven goals and requirements.
    - Identify and analyze data sources and understand diversity of data formats.
    - Evaluate data governance policies and compliance requirements.
    - Assess how the data lake will fit into the overall data architecture to ensure a seamless flow of data.
    - Assess whether cloud-based, on-premises, or hybrid deployment is best.
    - Ensure data lake can handle varying loads of data and analytics effectively.

- Available Technologies:
    - Hadoop Distributed File System (HDFS): Stores and manages large volumes of data across multiple nodes. Offers scalability, fault tolerance, and compatibility with Hadoop ecosystem tools.
    - Apache Spark: Fast and general-purpose computing system that can quickly process large-scale data. Often used with data lakes for analytics and processing tasks. Offers in-memory processing, supports multiple programming languages, compatible with various data sources.
    - AWS S3: Scalable object storage service. Offers scalability, durability, and integration with other AWS service.
    - Azure Data Lake Storage: Scalable and secure data lake solution that supports analytical and operational workloads. Offers integration with other Azure services, hierarchical namespace, and fine-grained access control.
    - Google Cloud Storage: Cloud-based object-storage service. Offers scalability, durability, and integration with other Google Cloud Platform (GCP) services.



## Modularity, encapsulation, and abstraction

- Modularity:
  - Modularity involves building data pipelines and components that can be developed, tested, and deployed independently. This approach facilitates easier updates, scalability, and reusability of code and can lead to more resilient data systems that are easier to maintain and scale.
- Encapsulation:
  - Encapsulation involves designing data models that hide the complexities of data transformations and storage details from the end-user. This can simplify data access and manipulation, making it easier for analysts and other stakeholders to utilize data without needing to understand the intricacies of underlying data storage or schema.
- Abstraction:
  - Abstraction involves  providing simplified views of complex data processes, such as abstracting the details of a multi-stage ETL process into a single, high-level operation. This helps manage complexity, especially when dealing with large-scale data infrastructures, and allows data engineers to focus on higher-level system design and optimization.
