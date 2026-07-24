To prepare for data engineering system design rounds, you must practice mapping business requirements to distributed systems components while justifying your architectural trade-offs.
## Prompt 1: The Real-Time Ad Analytics Dashboard
- **The Scenario:** Design an end-to-end data platform that ingests ad impression and click events from global web clients, tracks billing metrics, and populates an internal real-time analytics dashboard for advertisers.
- **Scale Requirements:** 100,000 events per second average, peaking at 500,000 events per second.
- **Core Technical Challenges:**
    - **Idempotency & Exact-Once:** Advertisers cannot be double-billed for duplicate click events caused by network retries.
    - **Late-Arriving Data:** Mobile clients occasionally cache clicks offline and send them hours late. How do you handle windowed aggregations when event time deviates significantly from processing time?

## Prompt 2: The E-Commerce Personalization Pipeline
- **The Scenario:** Design a system that captures user clickstream activity on a retail site and updates a "frequently viewed items" feature store, while simultaneously archiving all historical logs for long-term data science model training.
- **Scale Requirements:** 50 Terabytes of raw data generated per day.
- **Core Technical Challenges:**
    - **Storage Tiering & Layout:** How do you partition the storage layer (e.g., in AWS S3 using Apache Iceberg) to ensure data scientists can efficiently query 6 months of history without triggering massive data shuffling or full-table scans?
    - **Kappa vs. Lambda Architecture:** Would you deploy a unified stream engine or split the track into separate batch and speed layers? Defend your choice based on engineering maintenance costs.

## Prompt 3: The Cross-Region Database Migration Pipeline
- **The Scenario:** Your company is migrating its primary transactional database (OLTP Postgres) from an on-premises data center to a cloud data warehouse (OLAP Snowflake). You need to design the Change Data Capture (CDC) pipeline to replicate data continuously with zero downtime to the production app.
- **Scale Requirements:** 5,000 database writes/updates per second across 200+ tables with evolving schemas.
- **Core Technical Challenges:**
    - **Schema Evolution:** If a software engineer runs a migration that drops or alters a column in the transactional database, how does your pipeline prevent downstream data warehouse breakage?
    - **Data Integrity Verification:** How do you programmatically verify that the data in the analytical warehouse perfectly matches the transactional source without locking the operational database tables?

## Using These Prompts for Interview Preparation
Pick one of the prompts above. To practice effectively, paste the prompt into a new note in your Obsidian vault using your `Company Template.md` framework, and outline your architecture design by addressing:

1. **Ingestion Layer:** (e.g., Push vs. Pull, Event Broker selection)
2. **Processing Layer:** (e.g., Micro-batching vs. True Streaming, Handling state)
3. **Storage Layer:** (e.g., Choosing the right file formats, Partitioning strategies)
4. **Trade-offs:** (e.g., Latency vs. Accuracy, Cost vs. Scale)
