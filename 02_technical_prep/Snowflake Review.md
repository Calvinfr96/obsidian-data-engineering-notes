# Snowflake Interview Prep

> **Purpose:** Interview-focused summary of the uploaded Snowflake study material. This is intentionally more concise and conceptual than the original question banks. Focus on being able to **explain why Snowflake works this way, when to use each feature, and what tradeoffs you would make as a Data Engineer**.

---

## 1. Snowflake: The 30-Second Explanation

**Snowflake is a cloud-native analytical data platform whose key architectural characteristic is the separation of storage and compute.**

- **Storage:** Snowflake manages persistent data storage using compressed, columnar data organized into **micro-partitions**.
- **Compute:** **Virtual warehouses** provide independent compute resources for queries, transformations, loading, and unloading.
- **Cloud Services:** Coordinates authentication, authorization, metadata, query parsing/optimization, transactions, and other platform services.
- Because storage and compute are separated, you can scale compute independently of the amount of data stored.
- Multiple warehouses can work against the same underlying data, which makes **workload isolation** and **concurrency management** much easier than in traditional shared-compute architectures.
- Snowflake is primarily an **OLAP / analytical** platform rather than a traditional OLTP database.

### Strong interview answer

> "Snowflake is a cloud-native analytical platform built around separation of storage and compute. Data is stored in compressed micro-partitions, while queries and data-processing workloads run on independent virtual warehouses. A cloud-services layer handles things like metadata, authentication, query optimization, and transaction coordination. The separation lets organizations scale storage and compute independently and isolate workloads such as ETL, BI, and data science."

**Source:** Architecture and overview notes.

---

# 2. Core Architecture

## Three Layers

### 2.1 Storage Layer

Snowflake stores table data in an optimized, compressed, columnar representation.

The key concept is **micro-partitioning**:

- Tables are automatically divided into **micro-partitions**.
- Micro-partitions are immutable storage units.
- Snowflake maintains metadata about micro-partitions, including value ranges and other statistics.
- Queries can use this metadata for **partition pruning**, avoiding unnecessary data scans.
- Data is automatically clustered based largely on how it is loaded; Snowflake does not require traditional user-managed partitions.

### Why micro-partitions matter

Suppose a table contains years of orders and a query asks:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01';
```

If Snowflake's metadata shows that many micro-partitions contain only older dates, those partitions can be skipped.

**Interview connection:**

> Micro-partitioning + metadata + pruning is one of the reasons Snowflake can scan large analytical tables efficiently without traditional indexes.

---

## 2.2 Compute Layer: Virtual Warehouses

A **virtual warehouse** is a cluster of compute resources used to execute Snowflake workloads.

It provides:

- CPU
- Memory
- Local SSD/cache
- Compute capacity for SQL, DML, data loading, and unloading

Warehouses are independent from one another.

### Why that matters

You can have:

```text
ETL Warehouse  ──┐
                 ├──> Shared Snowflake Storage
BI Warehouse   ──┤
                 │
DS Warehouse   ──┘
```

An expensive ETL workload does not have to compete directly with BI queries for the same compute cluster.

### Scale up vs. scale out

**Scale up = more compute power**

Use this when individual queries are too slow.

Examples:

- Large joins
- Complex transformations
- Large aggregations

**Scale out = more clusters**

Use this when the problem is **concurrency** rather than individual-query performance.

Multi-cluster warehouses can add compute clusters to handle many simultaneous users/queries.

### Interview question: "A dashboard is slow. Should you increase warehouse size?"

Not automatically.

First determine whether:

1. Individual queries need more compute → **scale up**
2. Queries are queued because many users are querying simultaneously → **scale out / multi-cluster**
3. Queries are scanning too much data → optimize SQL, clustering, pruning, or data model
4. The query is repeatedly asking for the same result → caching may help

---

## 2.3 Cloud Services Layer

Think of this as Snowflake's **control plane / coordination layer**.

It handles responsibilities such as:

- Authentication
- Authorization
- Access control
- Metadata management
- Query parsing
- Query optimization
- Transaction coordination
- Infrastructure management
- Concurrency coordination

### Why the layer matters

The separation of services from compute means that not every Snowflake operation requires you to manage infrastructure directly.

---

# 3. Storage vs. Compute: The Big Interview Concept

Traditional data warehouse designs often couple storage and compute.

Snowflake separates them:

```text
              ┌─────────────────────┐
              │   Cloud Services    │
              │ Auth / Metadata /   │
              │ Optimization / Txns │
              └──────────┬──────────┘
                         │
       ┌─────────────────┴─────────────────┐
       │                                   │
┌──────▼──────┐                     ┌──────▼──────┐
│ ETL WH      │                     │ BI WH       │
└──────┬──────┘                     └──────┬──────┘
       │                                   │
       └──────────────┬────────────────────┘
                      ▼
             ┌─────────────────┐
             │ Shared Storage  │
             │ Micro-partitions│
             └─────────────────┘
```

### Main benefits

- Independent scaling
- Workload isolation
- Better concurrency
- Easier resource management
- Pay for compute based on warehouse usage
- Shared data without duplicating storage

### Cost model

Snowflake charges separately for:

- **Storage**
- **Compute / warehouse usage**

Warehouse compute is based on warehouse size and runtime.

**Interview takeaway:** Don't think of a warehouse as where the data lives. A warehouse is **compute**.

---

# 4. Tables

Snowflake's major table types:

| Type | Persists until | Time Travel | Fail-safe | Typical use |
|---|---|---:|---:|---|
| Permanent | Explicitly dropped | Yes | Yes | Production data |
| Transient | Explicitly dropped | Limited | No | Intermediate/persistent-but-recoverability-light data |
| Temporary | Session ends | Limited | No | Session/ETL temporary data |
| External | External data lifecycle | No | No | Query external data without loading it |

## Permanent tables

Default table type.

Use when the data is important and should have Snowflake's normal recovery protections.

## Transient tables

Persist until dropped but are intended for data that does not require the same recovery protection as permanent tables.

Useful for:

- Staging/intermediate data
- Rebuildable datasets
- ETL workflows

## Temporary tables

Exist only within the session that created them.

Useful for:

- Intermediate query results
- Temporary ETL work
- Session-specific transformations

## External tables

Provide a table-like interface over data stored outside Snowflake, such as cloud object storage.

Important distinction:

> **External stage** = where files are located.  
> **External table** = metadata/table interface that lets Snowflake query those external files.

---

# 5. Views

A **view is a saved SQL definition** that behaves like a table to the user but normally does not store the query result itself.

## Standard View

- Stores the SQL definition.
- Query is evaluated when the view is queried.
- Useful for abstraction, reusable transformations, and controlled data access.

```sql
CREATE VIEW it_employees AS
SELECT id, name, salary
FROM employee
WHERE department = 'IT';
```

## Secure View

A secure view is designed to prevent unauthorized users from inspecting the view definition and potentially inferring sensitive information.

Good use case:

> Sharing a derived dataset with another role/account while limiting exposure of the underlying implementation.

## Materialized View

A materialized view stores/precomputes results to improve performance for supported workloads.

Use it when:

- The same expensive computation is repeatedly queried.
- The query pattern is suitable for materialization.
- The performance improvement justifies the additional maintenance/storage cost.

### Standard vs. Materialized

| | Standard View | Materialized View |
|---|---|---|
| Stores query result? | No | Yes/precomputed |
| Query computation | At query time | Maintained by Snowflake |
| Performance | Depends on underlying query | Can be much faster for suitable workloads |
| Storage | Minimal | Additional storage/maintenance |

### Interview trap

Do **not** say:

> "A materialized view is simply a cached query result."

Better:

> "A materialized view is a persisted/precomputed representation maintained by Snowflake, whereas result caching is a query-result cache and is automatically managed by the platform."

---

# 6. Stages

A **stage is a location Snowflake uses to access data files for loading or unloading**.

## Internal stages

Managed by Snowflake.

Types include:

- User stage
- Table stage
- Named internal stage

## External stages

Point to cloud storage such as:

- Amazon S3
- Azure Blob Storage
- Google Cloud Storage

### Key commands

```sql
-- Create a stage
CREATE STAGE my_stage;

-- List files
LIST @my_stage;

-- Upload local file to internal stage
PUT file://path/file.csv @my_stage;

-- Remove a staged file
REMOVE @my_stage/file.csv;
```

### PUT vs. COPY INTO

This is a common interview question.

**PUT**

> Local machine → internal Snowflake stage

**COPY INTO**

> Stage/external location → Snowflake table

or:

> Snowflake table → stage/external location

---

# 7. File Formats

A **file format object** defines how Snowflake should interpret files.

Common formats:

- CSV
- JSON
- Parquet
- Avro
- ORC
- XML

Example:

```sql
CREATE FILE FORMAT csv_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
SKIP_HEADER = 1;
```

Useful options:

- `FIELD_DELIMITER`
- `RECORD_DELIMITER`
- `FIELD_OPTIONALLY_ENCLOSED_BY`
- `SKIP_HEADER`
- `NULL_IF`
- `EMPTY_FIELD_AS_NULL`
- `TRIM_SPACE`
- `ENCODING`
- `COMPRESSION`

### Semi-structured data

Snowflake supports semi-structured data using types such as:

- `VARIANT`
- `OBJECT`
- `ARRAY`

Example architecture:

```text
JSON file
   ↓
Stage
   ↓
COPY INTO
   ↓
VARIANT column
   ↓
SQL querying / transformation
```

---

# 8. COPY INTO

`COPY INTO` is the primary SQL command for bulk loading and unloading files.

## Load

```sql
COPY INTO orders
FROM @my_stage
FILE_FORMAT = (FORMAT_NAME = 'csv_format');
```

## Unload

```sql
COPY INTO @my_stage
FROM orders
FILE_FORMAT = (TYPE = 'CSV');
```

## Important options

### `ON_ERROR`

Controls what happens when bad data is encountered.

Examples include:

- `ABORT_STATEMENT`
- `CONTINUE`
- `SKIP_FILE`

### `VALIDATION_MODE`

Validate files without actually loading them.

```sql
COPY INTO orders
FROM @my_stage
VALIDATION_MODE = RETURN_ERRORS;
```

### `PURGE`

Remove successfully loaded files from the stage.

### `FORCE`

Force files that have already been processed to be loaded again.

### `PATTERN`

Load only files matching a pattern.

```sql
COPY INTO orders
FROM @my_stage
PATTERN = '.*\\.csv';
```

### Loading-performance principle

Prefer an appropriate number of reasonably sized files rather than one enormous file or thousands of tiny files.

Parallelism matters because Snowflake can load multiple files concurrently.

---

# 9. Snowpipe

**Snowpipe is Snowflake's continuous file-ingestion service.**

Use it when data arrives continuously in cloud storage and you want ingestion without repeatedly running manual/batch `COPY INTO` commands.

Typical flow:

```text
Application
    ↓
Cloud Storage (S3 / Azure / GCS)
    ↓
External Stage
    ↓
Snowpipe
    ↓
Snowflake Table
```

## Snowpipe vs. COPY

| | COPY INTO | Snowpipe |
|---|---|---|
| Typical workload | Batch | Continuous / near-real-time |
| Trigger | Explicit execution | Automated/event-driven ingestion |
| Compute | Warehouse-based loading | Snowpipe-managed/serverless ingestion |
| Common source | Stages | External cloud storage |

### Auto-ingest

Cloud-storage event notifications can tell Snowpipe that new files have arrived.

The key idea:

> **Storage event → Snowpipe → load file**

### Important distinction

Snowpipe provides **near-real-time file ingestion**, not row-by-row streaming.

If an application writes events continuously, they generally need to arrive as files or through another ingestion mechanism before Snowpipe processes them.

---

# 10. Streams: Change Data Capture

A **Snowflake Stream tracks changes to a table or supported source**, allowing downstream processing to consume only changes instead of repeatedly scanning the entire table.

Typical changes:

- INSERT
- UPDATE
- DELETE

Example:

```sql
CREATE STREAM employee_stream
ON TABLE employee;
```

Query:

```sql
SELECT *
FROM employee_stream;
```

## Stream metadata

Important metadata columns include:

- `METADATA$ACTION`
- `METADATA$ISUPDATE`

These help identify what happened to a row.

## Standard vs. append-only streams

**Standard stream**

Tracks inserts, updates, and deletes.

**Append-only stream**

Tracks inserts only.

```sql
CREATE STREAM new_employee_stream
ON TABLE employee
APPEND_ONLY = TRUE;
```

Use append-only when only newly arriving records matter, such as append-only event/log pipelines.

---

# 11. Stream Consumption

A major interview concept:

> A stream represents a change position in the source data. Downstream processing can consume the changes so that subsequent processing sees later changes rather than repeatedly processing the same change set.

A common pattern:

```text
Source Table
     ↓
  Stream
     ↓
 Task / SQL
     ↓
Target Table
```

Example:

```sql
INSERT INTO target_table
SELECT id, name
FROM employee_stream;
```

### Why streams are useful

Without a stream:

```text
Every ETL run
    ↓
Scan entire source table
    ↓
Find what changed
```

With a stream:

```text
Source changes
    ↓
Stream tracks changes
    ↓
ETL processes only changes
```

This is the fundamental **incremental-processing / CDC** benefit.

### Important stream caveats

- A stream tracks one source object.
- Streams do not directly monitor an S3 bucket or external stage.
- To track changes from external files, first load them into an appropriate Snowflake object.
- Streams depend on the underlying object's change-data retention window.
- Regularly consume streams so backlogs do not become unnecessarily large.

---

# 12. Snowpipe + Streams + Tasks

This is one of the most useful Snowflake interview architectures.

```text
S3
 │
 │ new file
 ▼
Snowpipe
 │
 ▼
Raw Snowflake Table
 │
 │ changes
 ▼
Stream
 │
 │ pending changes
 ▼
Task
 │
 ▼
Transformed / Curated Table
```

### What each component does

**Snowpipe:** gets new files into Snowflake.

**Stream:** records changes to the Snowflake table.

**Task:** executes SQL on a schedule/condition to process those changes.

### Interview answer

> "I would use Snowpipe for continuous file ingestion, a stream for CDC on the raw table, and a task to periodically consume the stream and apply incremental transformations to the target table. This avoids repeatedly scanning the entire raw dataset."

---

# 13. Tasks

A **Snowflake Task automates execution of SQL or a stored procedure**.

Tasks can be used for:

- ETL/ELT
- Aggregations
- Data quality checks
- Maintenance
- Incremental processing
- Stream consumption

## Basic task

```sql
CREATE TASK daily_task
WAREHOUSE = ETL_WH
SCHEDULE = 'USING CRON 0 0 * * * UTC'
AS
INSERT INTO daily_summary
SELECT ...
;
```

A task is initially **suspended** and must be resumed before it executes.

```sql
ALTER TASK daily_task RESUME;
```

Suspend:

```sql
ALTER TASK daily_task SUSPEND;
```

## User-managed vs. serverless tasks

**User-managed task**

You specify a warehouse.

**Serverless task**

Snowflake manages the compute resources for the task.

### Task dependencies

Tasks can form chains/DAG-like workflows using dependencies such as `AFTER`.

```text
Task A
  ↓
Task B
  ↓
Task C
```

If an upstream task fails, downstream dependent processing should not proceed as though the prerequisite succeeded.

### Monitoring

Use task history to investigate:

- Execution time
- Success/failure
- Errors
- Scheduling behavior
- Long-running tasks

---

# 14. Time Travel

**Time Travel lets you query or recover historical data within the object's retention period.**

Use cases:

- Recover accidental deletes
- Investigate incorrect updates
- Audit historical states
- Recover dropped objects
- Create point-in-time clones

Example:

```sql
SELECT *
FROM my_table
AT (TIMESTAMP => '2026-08-01 12:00:00');
```

You can also use `BEFORE` with a statement identifier.

### Restore a dropped object

```sql
UNDROP TABLE my_table;
```

provided the object is still within the applicable recovery window.

### Retention

The uploaded notes emphasize:

- Default retention is commonly 1 day.
- Retention can be increased depending on edition/configuration.
- Longer retention can increase storage cost.
- The commonly cited maximum in the notes is 90 days.

### Time Travel vs. backup

Time Travel is not the same thing as a conventional backup system.

Think:

> **Time Travel = user-accessible historical recovery within a retention window.**

---

# 15. Fail-safe

**Fail-safe is an additional recovery mechanism after Time Travel is no longer available.**

Key distinction:

| | Time Travel | Fail-safe |
|---|---|---|
| Purpose | Historical access/recovery | Emergency recovery |
| User queryable? | Yes | No |
| Access | User | Snowflake support |
| Duration | Configurable retention | Additional 7 days for supported permanent data |
| Normal operational tool? | Yes | No |

### Interview answer

> "Time Travel is the normal user-accessible recovery mechanism. If data ages beyond the Time Travel window for an eligible object, Fail-safe provides an additional recovery window, but it is not directly queryable and recovery requires Snowflake support."

### Important correction to weak source material

Do not describe Fail-safe as a feature you actively query or use for routine auditing. It is a **last-resort recovery mechanism**.

---

# 16. Zero-Copy Cloning

**Zero-copy cloning creates a new Snowflake object without immediately duplicating the underlying storage.**

Example:

```sql
CREATE TABLE dev_orders
CLONE prod_orders;
```

Conceptually:

```text
Production Data
      ▲
      │ shared underlying storage
      │
Development Clone
```

Initially, both reference the same underlying data.

When changes occur, Snowflake uses a **copy-on-write** model so that modified data requires additional storage.

## Why cloning is useful

Excellent for:

- Development environments
- Testing
- Experimentation
- Reproducing production data
- Point-in-time recovery
- Sandboxes

## Time Travel + cloning

You can create a clone of a historical state:

```sql
CREATE TABLE restored_orders
CLONE orders
AT (TIMESTAMP => '2026-08-01 12:00:00');
```

### Key interview question: "Does cloning duplicate the data?"

No.

> "The clone is initially metadata-based and shares underlying storage. Additional storage is consumed as either the source or clone diverges through changes."

---

# 17. Caching

Snowflake has multiple caching mechanisms. The most important interview distinction is between **result caching** and **warehouse/local data caching**.

## Result cache

Stores the result of a query so Snowflake can potentially return it without re-executing the query.

Useful for:

- Repeated BI queries
- Dashboards
- Repeated identical analytical queries

The uploaded notes commonly describe a 24-hour result-cache window, subject to Snowflake's eligibility/invalidation rules.

### Data changes

When relevant underlying data changes, cached results may no longer be valid.

## Warehouse/local data cache

Compute resources can cache data locally, reducing the need to repeatedly retrieve data from remote storage.

Unlike result caching, the query still executes; it simply benefits from faster access to data already available locally.

### Interview distinction

**Result cache:**

> "I can return the previous result."

**Data cache:**

> "I still execute the query, but reading the required data can be faster."

### Important caveat

Caching is automatic. You generally do not manually tune cache size/eviction policies the way you might with Redis.

---

# 18. Clustering

Snowflake does not use traditional user-managed table partitions in the same way many databases do.

Instead, Snowflake automatically stores data in micro-partitions.

**Clustering keys** can be defined for large tables where natural clustering is insufficient for important query patterns.

Example:

```sql
ALTER TABLE orders
CLUSTER BY (customer_id, order_date);
```

### Why clustering helps

Suppose most queries filter by:

```sql
WHERE customer_id = ?
AND order_date BETWEEN ? AND ?
```

A suitable clustering strategy can improve micro-partition pruning and reduce the amount of data scanned.

### Automatic Clustering

Snowflake can automatically maintain clustering for tables with clustering keys.

### Interview warning

Do not say:

> "Cluster every table."

Better:

> "Clustering is a targeted optimization. I'd first confirm that poor pruning or data organization is actually causing expensive scans before introducing a clustering key, because maintaining clustering has compute/storage cost."

---

# 19. Query Performance Troubleshooting

If a Snowflake query is slow, use a structured process.

### Step 1: Inspect Query History / Query Profile

Look for:

- Total execution time
- Bytes scanned
- Queuing
- Warehouse utilization
- Expensive operators
- Join behavior
- Spilling
- Data skew
- Repeated scans

### Step 2: Ask whether the warehouse is the problem

If many queries are queued:

- Consider larger warehouse capacity or multi-cluster scaling.

If one query is computationally expensive:

- Consider scaling up.
- Optimize the SQL.

### Step 3: Reduce data scanned

Look at:

- Filters
- Projection
- Join conditions
- Micro-partition pruning
- Clustering
- Data model

### Step 4: Check caching

If the same query is repeated frequently, determine whether result caching or local data caching is helping.

### Step 5: Consider materialization

For repeatedly expensive computations, consider:

- Materialized views
- Pre-aggregated tables
- Incremental transformation patterns

---

# 20. Security and RBAC

Snowflake uses **role-based access control (RBAC)**.

Core concepts:

```text
User
  ↓
Role
  ↓
Privileges
  ↓
Objects
```

A **privilege** is permission to perform an action on an object.

Examples:

- `SELECT`
- `INSERT`
- `UPDATE`
- `DELETE`
- `USAGE`
- `CREATE`

## Role hierarchy

Roles can be granted to other roles.

Example:

```text
DATA_ENGINEER
      ↑
ANALYST
```

The hierarchy allows permissions to be inherited.

## Common administrative roles

The notes emphasize:

- `ACCOUNTADMIN`
- `SECURITYADMIN`
- `USERADMIN`
- `SYSADMIN`
- `PUBLIC`

You should understand their broad responsibilities rather than memorizing every administrative detail.

### Typical principle

> Grant privileges to roles, then assign roles to users.

This is preferable to managing permissions individually for every user.

---

# 21. Least Privilege

A strong interview answer should mention **least privilege**.

Example:

A data engineer responsible for loading data does not necessarily need:

```text
ACCOUNTADMIN
```

Instead, give the role only what it needs:

- `USAGE` on warehouse
- `USAGE` on database/schema
- Required stage privileges
- Required table privileges

### Why?

- Limits blast radius
- Improves security
- Supports separation of duties
- Makes auditing easier

---

# 22. Row Access Policies and Masking

## Row Access Policy

Controls **which rows** a user can see.

Example concept:

```text
Analyst A → Region = East
Analyst B → Region = West
Admin     → All regions
```

The underlying table is the same, but the visible rows depend on the user's role/attributes.

## Data Masking Policy

Controls **what value** a user sees.

Example:

```text
Authorized role → 555-12-3456
Unauthorized role → ***-**-3456
```

### Easy distinction

> **Row access = which rows can I see?**  
> **Masking = what value can I see in the row?**

---

# 23. Data Sharing

Snowflake can share data between Snowflake accounts without requiring the producer to copy the entire dataset to the consumer.

Benefits:

- Avoids unnecessary data duplication
- Simplifies collaboration
- Enables near-real-time access to shared data
- Useful for organizations providing datasets to partners/customers

### Interview answer

> "Snowflake's data sharing capability lets a provider expose selected data to another account without the traditional process of exporting and duplicating the dataset. This reduces data movement and can keep shared data current."

---

# 24. Database vs. Schema

Think of the hierarchy as:

```text
Snowflake Account
    └── Database
         └── Schema
              ├── Tables
              ├── Views
              ├── Stages
              ├── Tasks
              └── Other objects
```

A **database** groups schemas.

A **schema** groups database objects.

---

# 25. Data Loading Architecture

A very common DE interview scenario:

> "Files arrive in S3 every few minutes. How would you ingest them into Snowflake?"

A strong answer:

```text
Source System
     ↓
S3
     ↓
External Stage
     ↓
Snowpipe / COPY INTO
     ↓
Raw Table
     ↓
Stream
     ↓
Task / Transformation
     ↓
Curated Tables
     ↓
BI / Analytics
```

Then discuss tradeoffs:

- Batch files arriving periodically → `COPY INTO`
- Continuous file arrival / low latency → Snowpipe
- Incremental downstream processing → Streams
- SQL-based automation → Tasks
- High concurrency → warehouse scaling / multi-cluster
- Data recovery → Time Travel
- Dev/test isolation → Zero-copy cloning

---

# 26. High-Value Comparison Questions

## COPY vs. Snowpipe

**COPY**

- Batch-oriented
- Explicitly executed
- Good for scheduled loads

**Snowpipe**

- Continuous
- Event-driven file ingestion
- Good for lower-latency file pipelines

---

## Stream vs. Task

These are complementary, not competing features.

**Stream = what changed?**

**Task = what should I do about it, and when?**

---

## Snowpipe vs. Stream

**Snowpipe moves files into Snowflake.**

**Stream tracks changes to Snowflake data.**

---

## Time Travel vs. Fail-safe

**Time Travel = user-accessible historical recovery.**

**Fail-safe = last-resort recovery through Snowflake support.**

---

## Clone vs. Copy

**Clone**

- Metadata operation initially
- Shared underlying storage
- Copy-on-write

**Physical copy**

- Duplicates data immediately
- Higher initial storage cost

---

## Standard View vs. Materialized View

**Standard View**

- Stores SQL definition
- Computes at query time

**Materialized View**

- Stores/precomputes results
- Faster for suitable repeated workloads
- Requires maintenance

---

## Scale Up vs. Scale Out

**Scale up**

> More compute for individual queries.

**Scale out**

> More clusters for concurrency.

---

# 27. Common Interview Questions

## Architecture

### Q: Why is separating storage and compute valuable?

**Answer:**

It allows them to scale independently. Storage can grow without requiring more compute, while compute can be increased temporarily for demanding workloads. It also allows separate warehouses to isolate ETL, BI, and other workloads.

**Likely follow-up:**  
"Why not just make one warehouse larger?"

**Answer:**  
Because workload isolation and concurrency may matter more than raw compute. Separate warehouses prevent one workload from monopolizing compute resources.

---

### Q: What are micro-partitions?

**Answer:**

Micro-partitions are Snowflake-managed, immutable storage units containing table data. Snowflake stores metadata about them and can use that metadata for pruning, allowing queries to avoid scanning irrelevant partitions.

**Likely follow-up:**  
"Who decides how the table is partitioned?"

**Answer:**  
Snowflake automatically creates micro-partitions; users do not manually define traditional partitions.

---

### Q: What is a virtual warehouse?

**Answer:**

A virtual warehouse is a cluster of compute resources used to execute Snowflake workloads. It is independent of persistent storage and can be resized or configured for concurrency.

---

## Data Ingestion

### Q: How would you load files from S3?

**Answer:**

I'd create an external stage pointing to S3, define an appropriate file format, and use `COPY INTO` for batch ingestion or Snowpipe if continuous ingestion is required.

---

### Q: How do you handle bad records?

**Answer:**

I'd use validation before production loading where appropriate, then configure `ON_ERROR` based on business requirements. For example, I might abort a critical financial load rather than silently skipping bad data. I'd also monitor load history and capture rejected records for remediation.

---

### Q: Why use Parquet?

**Answer:**

Parquet is columnar and efficient for analytical workloads. It supports compression and allows engines to read only relevant columns, reducing I/O.

---

## Incremental Processing

### Q: Why use a stream instead of scanning the entire table?

**Answer:**

A stream lets the pipeline identify changes since the relevant stream position, enabling incremental processing rather than repeatedly comparing or scanning the full source table.

---

### Q: How would you build an incremental ETL pipeline?

**Answer:**

I'd load raw data into Snowflake, create a stream on the raw table, and use a task to consume the stream and merge changes into the curated target. This reduces repeated full-table scans.

---

## Performance

### Q: A Snowflake query is slow. What do you check?

**Answer:**

I'd start with Query Profile and Query History. I'd determine whether the issue is query complexity, excessive data scanned, poor pruning, warehouse contention, concurrency, skew, or another bottleneck. Then I'd choose the appropriate optimization rather than simply increasing warehouse size.

---

### Q: When would you use a clustering key?

**Answer:**

When a large table has important, repetitive filtering patterns and natural clustering is insufficient, causing excessive scanning. I'd verify that clustering will materially improve pruning before accepting the additional maintenance cost.

---

### Q: When would you use a materialized view?

**Answer:**

When an expensive query or aggregation is executed repeatedly and the workload benefits from precomputed results. I'd compare the query-performance improvement against storage and maintenance overhead.

---

## Recovery

### Q: Someone accidentally deletes a table. What do you do?

**Answer:**

First determine whether the object is within the Time Travel recovery window. If so, use `UNDROP` or create/query a historical version as appropriate. If the data is beyond Time Travel but still eligible for Fail-safe, recovery would require Snowflake support.

---

# 28. Scenario Questions to Practice

These are more important than memorizing dozens of definitions.

### Scenario 1 — Continuous S3 ingestion

> Files arrive in S3 every few minutes. Design a Snowflake ingestion pipeline.

Discuss:

- External stage
- Snowpipe
- File format
- Raw table
- Load monitoring
- Duplicate handling
- Streams
- Tasks
- Data quality

---

### Scenario 2 — ETL is affecting BI

> A nightly ETL job causes dashboards to become slow.

Possible answer:

- Separate ETL and BI warehouses.
- Examine warehouse utilization and query queuing.
- Resize the ETL warehouse if individual ETL queries need more compute.
- Use multi-cluster for high BI concurrency if appropriate.
- Optimize expensive ETL queries.
- Avoid blindly increasing the BI warehouse.

---

### Scenario 3 — Slow analytical query

> A query scans a multi-terabyte table and takes several minutes.

Investigate:

1. Query Profile
2. Bytes scanned
3. Predicate pruning
4. Micro-partition organization
5. Clustering
6. Join strategy
7. Warehouse sizing
8. Materialization/caching opportunities

---

### Scenario 4 — Accidental data modification

> An engineer accidentally updates millions of rows.

Discuss:

- Time Travel
- Historical query
- Compare before/after
- Recover into a new table if appropriate
- Validate recovered data
- Investigate why the mistake happened
- Add safeguards/tests

---

### Scenario 5 — Development environment

> Developers need production-like data without creating an expensive full copy.

Answer:

> Use zero-copy cloning, with appropriate RBAC and data-governance controls.

---

# 29. SQL Cheat Sheet

## Database / Schema

```sql
CREATE DATABASE my_db;

CREATE SCHEMA my_schema;

USE DATABASE my_db;

USE SCHEMA my_schema;
```

## Tables

```sql
CREATE TABLE my_table (
    id INT,
    name STRING
);

CREATE TRANSIENT TABLE staging_table (
    id INT,
    name STRING
);

CREATE TEMPORARY TABLE temp_table (
    id INT,
    name STRING
);
```

## Views

```sql
CREATE VIEW my_view AS
SELECT *
FROM my_table;

CREATE SECURE VIEW my_secure_view AS
SELECT *
FROM my_table;

CREATE MATERIALIZED VIEW my_mv AS
SELECT category, COUNT(*) AS cnt
FROM my_table
GROUP BY category;
```

## Stage

```sql
CREATE STAGE my_stage;

LIST @my_stage;

PUT file://data.csv @my_stage;
```

## File format

```sql
CREATE FILE FORMAT csv_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1;
```

## COPY

```sql
COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (FORMAT_NAME = 'csv_format');

COPY INTO @my_stage
FROM my_table
FILE_FORMAT = (TYPE = 'CSV');
```

## Stream

```sql
CREATE STREAM my_stream
ON TABLE my_table;

SELECT *
FROM my_stream;
```

## Task

```sql
CREATE TASK my_task
WAREHOUSE = ETL_WH
SCHEDULE = '5 MINUTE'
AS
INSERT INTO target_table
SELECT *
FROM my_stream;

ALTER TASK my_task RESUME;

ALTER TASK my_task SUSPEND;
```

## Time Travel

```sql
SELECT *
FROM my_table
AT (TIMESTAMP => '2026-08-01 12:00:00');

SELECT *
FROM my_table
BEFORE (STATEMENT => 'query_id');
```

## Clone

```sql
CREATE TABLE cloned_table
CLONE my_table;

CREATE TABLE historical_clone
CLONE my_table
AT (TIMESTAMP => '2026-08-01 12:00:00');
```

## RBAC

```sql
CREATE ROLE data_engineer;

GRANT USAGE ON DATABASE my_db
TO ROLE data_engineer;

GRANT USAGE ON SCHEMA my_db.my_schema
TO ROLE data_engineer;

GRANT SELECT ON TABLE my_table
TO ROLE data_engineer;
```

---

# 30. Interview "Mental Models"

Memorize these rather than isolated definitions:

| Concept | Mental model |
|---|---|
| Snowflake | Cloud-native analytical platform |
| Storage/compute separation | Data and processing scale independently |
| Micro-partition | Immutable chunk of Snowflake-managed table storage |
| Pruning | Skip irrelevant micro-partitions |
| Warehouse | Compute, not persistent storage |
| Scale up | More compute per workload |
| Scale out | More clusters for concurrency |
| Stage | File landing/access location |
| File format | Rules for interpreting files |
| COPY | Batch file movement |
| Snowpipe | Continuous file ingestion |
| Stream | What changed? |
| Task | What should run automatically? |
| Time Travel | User-accessible historical recovery |
| Fail-safe | Last-resort recovery |
| Clone | Fast copy using shared storage initially |
| Result cache | Reuse a previous query result |
| Data cache | Reuse locally cached data during execution |
| Clustering | Improve data organization/pruning |
| RBAC | Users get roles; roles get privileges |
| Row access policy | Which rows? |
| Masking policy | Which values? |
| Materialized view | Persist/precompute an expensive result |

---

# 31. Important Corrections / Interview Traps

The uploaded question banks contain useful material, but some answers are oversimplified or occasionally inaccurate. For interviews, use the following framing.

### 1. Snowflake is not "an AWS data warehouse"

Snowflake runs on multiple cloud providers. The notes identify AWS, Azure, and GCP.

Say:

> "Snowflake is a cloud-native SaaS data platform available on major cloud providers."

---

### 2. Don't equate all caching with result caching

Result caching and local/warehouse data caching are different mechanisms.

---

### 3. Don't say Fail-safe is queryable

It is not a normal user-queryable historical store.

---

### 4. Don't say Snowpipe is row-level streaming

Snowpipe is primarily continuous **file ingestion**.

---

### 5. Don't say streams are tables that store changed rows

A stream is a **change-tracking object**. It exposes change information from the source rather than functioning as an independent persistent copy of the data.

---

### 6. Don't use clustering as a synonym for partitioning

Snowflake automatically creates micro-partitions. Clustering keys are an additional optimization mechanism.

---

### 7. Don't claim that simply increasing warehouse size fixes every performance problem

First determine whether the bottleneck is:

- Query complexity
- Data scanned
- Poor pruning
- Concurrency
- Warehouse capacity
- Caching
- Data skew
- Inefficient data model

---

### 8. Don't confuse a stage with a table

A stage deals with **files**.

A table stores/query-exposes **data records**.

---

### 9. Don't confuse a stream with a task

Stream:

> captures/exposes changes.

Task:

> automates execution.

---

# 32. What to Prioritize for Data Engineering Interviews

If you are short on study time, prioritize these topics in this order:

## Tier 1 — Must Know

1. Snowflake architecture
2. Storage vs. compute separation
3. Virtual warehouses
4. Micro-partitions and pruning
5. COPY INTO
6. Stages
7. Snowpipe
8. Streams / CDC
9. Tasks
10. Time Travel vs. Fail-safe
11. Zero-copy cloning
12. RBAC / least privilege

## Tier 2 — Strongly Recommended

13. Standard vs. secure vs. materialized views
14. Clustering
15. Caching
16. File formats
17. Semi-structured data / VARIANT
18. Query Profile / performance troubleshooting
19. Data sharing
20. Row access and masking policies

## Tier 3 — Lower Priority

21. Snowflake UI navigation
22. Detailed edition differences
23. Less-common file-format parameters
24. Administrative UI features
25. Minor SQL DDL commands

For a Data Engineer interview, **Tier 1 + the scenario questions** are more valuable than memorizing every question from the source PDFs.

---

# 33. Final Interview Checklist

You should be able to answer these without notes:

- [ ] Explain Snowflake's three-layer architecture.
- [ ] Explain why storage/compute separation matters.
- [ ] Explain micro-partitions and pruning.
- [ ] Explain scale-up vs. scale-out.
- [ ] Explain what a virtual warehouse does.
- [ ] Explain internal vs. external stages.
- [ ] Explain `PUT` vs. `COPY INTO`.
- [ ] Explain file formats and semi-structured data.
- [ ] Explain `ON_ERROR`, `VALIDATION_MODE`, `PURGE`, and `FORCE`.
- [ ] Explain COPY vs. Snowpipe.
- [ ] Design an S3 → Snowflake ingestion pipeline.
- [ ] Explain streams as CDC.
- [ ] Explain standard vs. append-only streams.
- [ ] Explain Streams + Tasks.
- [ ] Explain Snowpipe + Streams + Tasks together.
- [ ] Explain Time Travel.
- [ ] Explain Time Travel vs. Fail-safe.
- [ ] Explain zero-copy cloning.
- [ ] Explain result caching vs. data caching.
- [ ] Explain clustering and when to use it.
- [ ] Troubleshoot a slow Snowflake query.
- [ ] Explain RBAC and least privilege.
- [ ] Explain row access vs. masking policies.
- [ ] Explain standard vs. secure vs. materialized views.
- [ ] Explain Snowflake data sharing.

---

# 34. One-Minute Architecture Summary

If the interviewer asks you to summarize Snowflake, a strong closing answer is:

> "The main thing I associate with Snowflake is separation of storage and compute. Snowflake stores data in compressed micro-partitions and maintains metadata that allows it to prune irrelevant data during queries. Compute is provided through independent virtual warehouses, so I can isolate ETL, BI, and other workloads and scale compute separately from storage. On top of that architecture, Snowflake provides native capabilities for ingestion with stages, COPY and Snowpipe; incremental processing with Streams and Tasks; recovery with Time Travel and Fail-safe; development isolation with zero-copy cloning; and security through RBAC and policies such as row access and masking. As a data engineer, I'd choose among those features based on latency, workload, concurrency, recoverability, security, and cost requirements."
