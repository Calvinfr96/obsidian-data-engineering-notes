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
