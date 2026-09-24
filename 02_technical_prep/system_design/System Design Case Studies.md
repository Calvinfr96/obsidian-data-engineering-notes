---
aliases:
  - "system_design_case_studies"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/System Design/system_design_case_studies.md"
imported: 2026-09-24
---

# System Design Case Studies

Review: [[02_technical_prep/system_design/System Design Interview Prep|System Design Interview Prep]] · [[02_technical_prep/system_design/Solving System Design Interview Questions|Solving System Design Interview Questions]]

## Contents

- [[#Migrating an Enterprise Data Warehouse (EDW) to Azure Databricks]]
- [[#Migrating AWS Glue to Snowflake]]
- [[#Migrating and Archiving ADAS Workloads to AWS]]
- [[#Migration from Legacy Postgres Data Warehouse to Snowflake]]
- [[#Understanding Data Mesh Through a Real-World Analogy]]
- [[#Fortune 500 Health Service Provider Achieves 30% Productivity Boost with Azure Databricks]]

## Migrating an Enterprise Data Warehouse (EDW) to Azure Databricks
This case study highlights the journey of a leading global retailer in modernizing its Enterprise Data Warehouse (EDW) by migrating from an on-premises SQL Server 2014 system to Azure Databricks. The retailer operates 3,000 offline stores across 40 countries, generating massive volumes of data daily. This migration addressed critical challenges such as outdated infrastructure, slow data processing, and limited scalability.

### Legacy Architecture: Understanding The Initial System
- The legacy data ecosystem consisted of the following components:
  - Order Management System (OMS)
  - Point of Sale (POS)
  - ETL (Extract, Transform, Load)
  - Enterprise Data Warehouse (EDW)
  - Business Intelligence (BI) Tools, Traffic, Inventory, and Staff Metrics
- Challenges:
  - Infrastructure Limitations: The aging SQL Server struggled with storage capacity and slower processing.
  - Complex Data Integration: Adding new sources required significant manual effort and customization.
  - Historical Data Access: Lack of retained raw data limited trend analysis and reporting flexibility.
  - Documentation Issues: The fragmented documentation slowed troubleshooting and updates.
  - Scalability: Physical servers couldn’t meet the growing data demands, especially with global operations.

### Migrated Architecture: Azure Databricks Solution
- The migration transitioned the legacy EDW to a cloud-based platform on Azure Databricks, leveraging the Medallion Architecture to organize data into Bronze, Silver, and Gold layers.
  - Bronze Layer:
    - Raw data is ingested directly from various sources like REST APIs, Kafka, and JSON files.
    - Historical data is preserved for reprocessing and future analysis.
  - Silver Layer:
    - Data is cleaned, validated, and transformed into a structured format.
    - Ensures consistency and prepares data for analytics.
  - Gold Layer:
    - Aggregated and summarized data, ready for BI tools like Power BI and Tableau.
    - Supports business-specific insights such as sales trends and customer behavior analysis.
- Key Technologies:
  - Azure Data Factory: Used for orchestrating data workflows from ingestion to processing.
  - Apache Spark on Azure Databricks: Enabled fast, scalable data transformations.
  - Azure Synapse Analytics: Provided seamless data warehousing for analytics.
  - Power BI: Delivered real-time dashboards and interactive reports.

### Benefits of Migration
- Resolved Challenges:
  - Scalability: Azure Databricks dynamically scales resources, eliminating physical server limitations.
  - Real-Time Analytics: Spark-powered transformations support near real-time insights.
  - Historical Data Access: The Bronze layer preserves raw data, enabling future trend analyses.
  - Faster Integration: New data sources can be onboarded in weeks instead of months.
  - Cost Efficiency: Cloud-based pay-as-you-go pricing significantly reduces costs.
- Quantifiable Impacts:
  - Integration Speed: Reduces time for onboarding new tools from months to weeks.
  - Processing Time: Real-time capabilities replace batch processing delays.
  - Data Quality: Enhanced validation frameworks ensure consistent and accurate data.
- Key Takeaways:
  - Adopt Modular Architectures: The Medallion Architecture simplifies data processing and aligns with business needs.
  - Preserve Historical Data: Retaining raw data in the Bronze layer enables future use cases.
  - Emphasize Scalability: Cloud platforms like Azure Databricks grow with business needs effortlessly.
  - Automate Workflows: Tools like Azure Data Factory streamline data ingestion and transformations.

## Migrating AWS Glue to Snowflake
This case study explores how a leading client optimized their data processing workflows by migrating from AWS Glue to Snowflake, a cloud-native data warehouse. By leveraging Snowflake’s advanced capabilities, the client achieved significant cost savings, improved performance, and simplified their data architecture.

### Previous Architecture
- Data Sources: Amazon S3 (source), Amazon S3 (destination).
- ETL Processing: AWS Glue with PySpark/Scala.
  - Serverless data integration service that helps businesses extract, transform, and load data from different data sources.
  - The Glue Crawler automatically detects raw data from the source and creates a schema and metadata based on that data.
  - The Glue Catalog stores the information using the inferred schema from the glue crawler.
  - Glue Jobs process and clean the data stored in the glue catalog. Glue jobs use PySpark or Scala scripts to apply data transformations such as cleaning, deduping, and structuring.
  - Manual or automated triggers determine when glue jobs are executed.
- Challenges:
  - High operational overhead
  - Delayed data availability.
  - Complex manual interventions.

### Migrated Architecture: Snowflake
Snowflake’s cloud-native architecture was chosen as the ideal solution to address the above challenges, offering seamless integration with AWS services and enhanced performance.
- Key Features of Snowflake:
  - Integration with Amazon S3:
    - Historical Data Migration: Utilizes the `COPY INTO` command for efficient transfer.
    - Real-Time Ingestion: Implements Snowpipe for continuous data loading.
  - Performance Optimization:
    - Transitions from PySpark-based ETL to SQL-based transformations within Snowflake.
    - Centralizes data management for faster processing.
  - Cost Efficiency:
    - Adopts Snowflake’s pay-per-second pricing model, reducing overall costs.
  Improved Scalability:
    - Handles large-scale datasets with ease, enabling growth without architectural changes.
- Data Sources: Amazon S3.
- ETL Processing: Snowflake with SQL transformations.
- Data Ingestion:
  - Historical data: `COPY INTO` command.
  - Real-time data: Snowpipe.
- Benefits:
  - Fully automated workflows.
  - Real-time data availability.
  - Simplified architecture with reduced manual effort.

### Results and Value Creation
- Cost Savings:
  - Monthly savings of $3,000 by eliminating Glue-specific overhead.
  - Optimized resource usage with Snowflake’s granular pricing model.
- Time Efficiency:
  - Reduced daily data ingestion and processing time from 24 hours to 12 hours.
  - Improved data availability for D-1 business reporting.
- Scalability and Performance:
  - Seamlessly scaled to handle 500 GB daily data volumes.
  - Enhanced query performance with Snowflake’s computational efficiency.

### Lessons Learned
- Choose the Right Tools: Evaluate data solutions based on scalability, cost, and integration capabilities.
- Optimize Costs Continuously: Regularly review resource utilization and adapt pricing models.
- Simplify Architectures: Minimize complexity to reduce maintenance and operational overhead.
- Embrace Automation: Automate processes for greater efficiency and accuracy.

## Migrating and Archiving ADAS Workloads to AWS
A global automotive OEM at the forefront of developing Advanced Driver Assistance Systems (ADAS) faced mounting challenges in managing their massive data workloads. With terabytes of data generated daily from sensors, cameras, and simulations, they needed an effective cloud-based solution to migrate, store, and archive their data securely and cost-effectively. This case study demonstrates how the OEM partnered with AWS to build a scalable, future-proof data management strategy that addressed their challenges and set the foundation for autonomous vehicle innovation.

### Challenges
Managing ADAS workloads on-premises was becoming increasingly difficult due to:
  - Massive Data Volumes: ADAS workloads include data-intensive tasks like object detection, lane detection, and sensor fusion, which generate terabytes of data every day.
  - Data Transfer Limitations: Remote testing sites lacked high-speed internet, making it challenging to move data to centralized storage.
  - Cost Inefficiencies: High-performance active storage for long-term archival data was prohibitively expensive.
  - Data Integrity and Security: The sensitive nature of ADAS data, including intellectual property, required robust security measures during migration and storage.
  - Regulatory Compliance: Strict legal and regulatory requirements demanded long-term retention of data in a secure, auditable manner.

### Solution Architecture
The AWS-based architecture included the following components:
  - Data Ingestion: ADAS data from test sites was ingested using AWS Snowball (offline) and AWS DataSync (online).
  - Storage in Amazon S3: Data was first stored in Amazon S3 buckets, organized and tagged by ADAS projects for easy identification.
  - Archival to S3 Glacier: Lifecycle policies automated the movement of infrequently accessed data to S3 Glacier tiers for cost-effective storage.
  - Data Access: Engineers and analysts accessed archived data using AWS Glue and Athena for analysis and compliance reporting.

### Results
The OEM experienced transformative results after implementing AWS’s solution:
  - Scalability: Seamlessly managed petabyte-scale workloads, enabling continuous ADAS development.
  - Cost Savings: Reduced long-term storage costs by leveraging S3 Glacier tiers.
  - Enhanced Security: End-to-end encryption and tamper-proof Snowball devices ensured data integrity during migration.
  - Accelerated Insights: Automated workflows and quick retrieval options reduced the time required for R&D and analysis.
  - Regulatory Compliance: Lifecycle policies ensured adherence to data retention and privacy regulations, supporting the OEM’s legal and operational needs.

## Migration from Legacy Postgres Data Warehouse to Snowflake
A leading delivery services company in the US sought to modernize its existing Old Generation (OG) Postgres data warehouse by transitioning to a scalable cloud-based data warehouse on Snowflake. The company was dealing with multiple data ingestion challenges, latency issues, frequent outages, and a lack of Change Data Capture (CDC) capabilities. The goal of this migration was to reduce downtime, improve data freshness, enhance scalability, and enable near real-time analytics. This case study details the business problem, challenges with the existing solution, the modern architecture on Snowflake, and the key benefits achieved post-migration.

### Business Problem
The client’s existing PostgreSQL-based data warehouse was outdated, inefficient, and unable to support growing data needs. The major pain points included:
  - High Latency
  - Frequent Data Outages
  - Manual Interventions
  - Lack of Change Data Capture (CDC)
  - Onboarding Third-Party Data Sources

### Current Architecture
The client’s architecture was monolithic and inefficient, causing multiple operational and scalability issues, including:
  - Data Sources & Ingestion Bottlenecks:
    - Data was sourced from 20+ applications, streaming platforms, and in-house RDBMS systems.
    - Over 100 tables were manually managed, leading to integration challenges.
  - ETL Processing in Postgres:
    - PostgreSQL was not optimized for modern ETL workflows, causing slow transformations.
    - There was no real-time data streaming, and processing workloads caused frequent failures.
  - Data Latency Issues:
    - Batch-based ingestion delayed insights.
    - The company lacked automation, causing significant downtime.
  - High Maintenance Costs:
    - Maintaining on-premises PostgreSQL infrastructure was costly and inefficient.
    - The manual effort for monitoring and fixing failures consumed IT resources.

### Future State: Cloud-Based Snowflake Architecture
To address these limitations, the company migrated to a modern cloud data architecture on Snowflake, using Fivetran, Kafka, and AWS S3 for scalable data ingestion:
  - Cloud Data Ingestion Pipeline:
    - Fivetran was implemented to extract data from PostgreSQL and move it to S3.
    - Kafka and AWS SNS (Simple Notification Service) were used to capture events and trigger data movement.
    - Segment API and Custom Connectors were integrated to stream additional third-party sources.
  - Automated Processing in Snowflake:
    - Snowpipe enabled automated ingestion of new and changed data into Snowflake.
    - Business logic and transformations were executed in Snowflake’s scalable compute environment.
  - Real-Time Data Streaming & CDC:
    - Kafka’s streaming mechanism enabled real-time CDC.
    - Any new, updated, or deleted records were captured in near real-time, eliminating the need for full refreshes.
  - Data Marts for Analytics:
    - Processed data was organized into Data Marts for Business Intelligence (BI) and Data Science teams.
    - Business users gained real-time access to dashboards, reports, and advanced analytics.

### Implementation Approach
The migration was executed in phases to ensure a seamless transition:
  - Assessment & Planning:
    - Conducted an audit of all source systems and dependencies.
    - Defined data governance policies and best practices.
  - Incremental Migration Strategy:
    - Used Fivetran for automated data replication from PostgreSQL to AWS S3.
    - Implemented CDC via Kafka for real-time tracking.
  - Validation & Testing:
    - Ensured data accuracy and consistency across PostgreSQL and Snowflake.
    - Created validation scripts to compare migrated and legacy datasets.
  - Final Cut-Over & Go-Live:
    - Switched to Snowflake as the primary data warehouse.
    - Established monitoring and alerting mechanisms for proactive issue resolution.

### Benefits
The transition to Snowflake’s cloud data warehouse resulted in significant improvements across performance, cost, and scalability, including:
  - Near Real-Time Data Availability:
    - Latency reduced significantly, with data now available in near real-time.
    - Business users no longer wait hours for updated reports.
  - 90% Reduction in Data Outages:
    - Fully automated ingestion and monitoring reduced data failures by 90%.
    - Built-in redundancy ensured high availability.
  - Seamless Change Data Capture (CDC):
    - The new CDC pipeline detects and processes changes instantly.
    - Eliminated costly full-table refreshes.
  - Simplified and Fully-Managed Pipelines:
    - Snowflake + Fivetran + Kafka created a low-maintenance, cost-effective data pipeline.
    - No need for constant manual intervention.
  - Faster Third-Party Data Onboarding:
    - Standardized APIs and custom connectors enabled rapid integration of new data sources.
    - Reduced onboarding time from weeks to days.
  - Cost Savings & Scalability:
    - Auto-scaling compute and storage eliminated on-prem infrastructure costs.
    - Optimized query performance ensured efficient data processing.

## Understanding Data Mesh Through a Real-World Analogy
Data is the backbone of modern enterprises, but traditional data management approaches often struggle with scalability, governance, and agility. This case study explores the concept of Data Mesh by drawing an analogy to a food court, illustrating its real-world applicability.

### Food Court Analogy
A food court closely resembles data mesh architecture in the following ways:
  - Multiple Data Domains: Just like different food outlets specialize in specific cuisines, various business functions own and manage their respective data domains (e.g., Marketing, Sales, Support).
  - Self-Serve Infrastructure: Each food outlet operates independently but follows common guidelines set by the mall management, much like a self-serve data platform that allows teams to manage their data independently while adhering to governance rules.
  - Federated Governance: The mall management enforces food safety regulations while allowing individual food outlets to create their own unique offerings. Similarly, Data Mesh governance provides a centralized framework while allowing domain teams to operate autonomously.

### Evolution of Enterprise Data Strategies
Understanding the evolution of enterprise data strategies helps contextualize why Data Mesh has emerged as a game-changer.
- Traditional Approaches:
  - 1980s-1990s: Data Warehousing centralized data storage with restricted access.
  - 2000s-2010s: Business Intelligence (BI) and Master Data Management (MDM) emerge, along with cloud computing.
  - 2010s-Present: Shift toward data democratization, real-time data processing, AI-driven analytics, and privacy compliance.
- Data Mesh & the Future:
  - Decentralized Data Ownership: Each team manages its own data.
  - Data as a Product: Domains produce high-quality, reusable data products.
  - Self-Serve Data Infrastructure: Common tools and frameworks facilitate ease of access.
  - Federated Governance: A balance between centralized policies and domain autonomy.

### Architecture Breakdown
- Domains as Individual Business Units:
  - Each business domain, like a food outlet, is responsible for managing its own data. For example:
    - Marketing: Handles campaign performance and customer engagement data.
    - Sales: Manages transactions, product orders, and revenue insights.
    - Support: Processes customer complaints, chatbot interactions, and call logs.
- Self-Serve Data Platform:
  - The data platform ensures smooth data access, governance, and collaboration, akin to the mall’s infrastructure enabling seamless food court operations.
- Federated Governance & Security:
  - A governance team enforces compliance rules across all data domains while allowing flexibility, just as a mall mandates hygiene standards but lets individual outlets decide their menu.

### Key Takeaways
- Data Mesh decentralizes data ownership, making teams accountable for their own data products.
- It enhances data accessibility, governance, and agility, allowing businesses to scale seamlessly.
- Drawing real-world analogies, like a food court, makes technical concepts easier to grasp.

## Fortune 500 Health Service Provider Achieves 30% Productivity Boost with Azure Databricks
A Fortune 500 health service provider faced immense challenges managing its complex data ecosystem. An expert stepped in to design and implement a modern data platform leveraging Azure Databricks, enabling the client to achieve a 30% increase in productivity while reducing IT costs and improving data quality and scalability.

### Challenges
The client’s data ecosystem was characterized by:
- Unstructured Data Overload
- Complex Data Configuration
- High Fixed Costs
- Lengthy Development Cycles
- Persistent Data Issues

### Solution
Designed and implemented a cutting-edge data platform powered by Azure Databricks, addressing the client’s challenges and modernizing their data ecosystem. Key components of this platform include:
- Leveraging Azure Data Factory (ADF)
- Implementing Databricks Delta Tables using medallion architecture.
- Optimized Storage
- Metadata-Driven Ingestion Framework
- Enhanced Transformation with PySpark and PolyBase

Post-modernization, the client’s data platform was transformed into a robust, agile, and scalable system. Key benefits included:
- Increased Productivity
- Improved Data Quality and Integrity
- Cost Reductions
- Enhanced Scalability and Agility
- Faster Time-to-Insights
