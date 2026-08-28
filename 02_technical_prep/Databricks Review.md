# Databricks Interview Prep

> **Purpose:** Interview-focused Databricks notes distilled from the uploaded Q&A PDF. The original material is largely certification/interview-style multiple-choice questions, so this document reorganizes the material around the concepts you should be able to **explain, compare, and apply** rather than memorizing answer choices.
>
> **Core mental model:** Databricks combines **Spark compute + lakehouse storage + governance + orchestration/development tooling**. For interviews, be able to explain what each component does, why it exists, and when you would choose it.

---

# 1. Databricks Lakehouse

A **data lakehouse** combines characteristics of data lakes and data warehouses.

```text
                  Databricks Lakehouse
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   Open storage       Reliable data     Analytics/
   + flexibility      management        ML workloads
        │                │                │
     Parquet         Delta Lake       SQL / Spark
                     + ACID
```

### Why a lakehouse?

A traditional data lake provides flexible, inexpensive storage but historically lacked some of the transactional and governance capabilities associated with warehouses.

A lakehouse adds capabilities such as:

- ACID transactions
- Reliable updates/deletes
- Schema management
- Data governance
- SQL analytics
- Batch + streaming processing
- ML/data science workloads

### Interview answer

> "A lakehouse combines the flexibility and open storage of a data lake with warehouse-like reliability and governance. In Databricks, Delta Lake provides transactional capabilities on top of cloud object storage, allowing the same underlying data to support engineering, analytics, and other workloads."

---

# 2. Data Silos and Unity Catalog

One of the PDF's central lakehouse questions asks how a lakehouse can prevent data engineering and analytics teams from producing inconsistent reports.

The answer is:

> **Provide a common source of truth.**

Unity Catalog helps establish centralized governance and discovery across data assets.

### Interview concept

Without centralized governance:

```text
Data Engineering → Dataset A
Data Analytics    → Dataset B
                    ↓
              Different results
```

With centralized governance:

```text
              Shared governed data
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
     Engineers     Analysts      Data Science
```

### Remember

> **Lakehouse benefit:** shared data foundation.

> **Unity Catalog:** centralized governance, discovery, and access control for data/AI assets.

---

# 3. Databricks Architecture: Control Plane vs. Data Plane

A useful interview distinction is **where management happens vs. where computation happens**.

## Control plane

Responsible for managing the Databricks environment.

The source specifically identifies the **Databricks web application** as a control-plane component in the classic architecture.

Typical responsibilities include:

- Workspace management
- Cluster provisioning/management
- Notebook management
- Access control
- Job scheduling
- REST/API management

## Data plane

Where workload execution occurs.

Includes resources such as:

- Driver
- Worker nodes
- Spark execution
- Connections to data sources

### Mental model

```text
Control Plane
    │
    ├── Web application
    ├── Workspace management
    ├── APIs
    └── Orchestration/control
            │
            ↓
Data Plane
    │
    ├── Driver
    ├── Workers
    └── Spark computation
```

### Interview answer

> "The control plane manages and coordinates the Databricks environment, while the data plane is where Spark workloads actually execute."

---

# 4. Clusters

A Databricks cluster provides compute resources for Spark workloads.

Conceptually:

```text
             Cluster
                │
        ┌───────┴───────┐
        ↓               ↓
     Driver          Workers
        │               │
        └────── Spark ──┘
```

## Driver

The driver coordinates the Spark application.

Responsibilities include:

- Creating the Spark session/context
- Building the execution plan
- Scheduling work
- Coordinating executors
- Maintaining application-level state

## Workers

Workers execute tasks assigned by the driver.

### Interview distinction

> **Driver = coordinator**

> **Workers/executors = distributed computation**

---

# 5. Cluster Pools

A **cluster pool** keeps a set of idle, ready-to-use cloud instances available so that new clusters can start faster.

### Why use a pool?

Cluster startup can be slow because new cloud instances need to be provisioned.

Pools reduce startup latency:

```text
Without pool:
Job
 ↓
Provision instances
 ↓
Start cluster
 ↓
Run job

With pool:
Job
 ↓
Use pre-warmed instances
 ↓
Start cluster faster
 ↓
Run job
```

### Interview question

> When would a data team use a cluster pool?

When **startup latency matters**, such as an automated report that needs to refresh as quickly as possible.

### Important

Pools improve **startup time**; they don't inherently make the Spark computation itself faster.

---

# 6. Delta Lake

**Delta Lake** is the storage layer that adds reliability and transactional capabilities to data stored in a data lake.

Key capabilities emphasized by the notes:

- ACID transactions
- Batch + streaming workloads
- Updates/deletes
- MERGE/upserts
- Time travel
- Transaction history
- Schema management/evolution

### Why Delta instead of plain Parquet?

Parquet provides efficient columnar storage, but Delta adds a transaction/logging layer that allows reliable table operations.

### Mental model

```text
Cloud Object Storage
        │
        ├── Parquet data files
        │
        └── Delta transaction log
                 ↓
        Reliable table semantics
```

---

# 7. Delta Table Storage

A Delta table is **not a single file**.

It is a collection of files, including:

- Parquet data files
- Delta transaction-log information
- Metadata/history information

Conceptually:

```text
my_table/
├── part-00000.parquet
├── part-00001.parquet
├── ...
└── _delta_log/
    ├── ...
    └── transaction history
```

### Interview answer

> "A Delta table is fundamentally a collection of data files plus Delta transaction-log metadata. The transaction log tracks table versions and operations, while the Parquet files hold the actual data."

---

# 8. ACID Transactions

One of the PDF's questions asks which lakehouse feature most directly improves data quality compared with a traditional data lake.

Answer:

> **ACID-compliant transactions.**

ACID provides:

- **Atomicity** — operations succeed completely or don't apply.
- **Consistency** — transactions preserve valid table state.
- **Isolation** — concurrent operations are handled safely.
- **Durability** — committed changes persist.

### Why this matters

Without transactional guarantees, concurrent writes or partial failures can leave a data lake in an inconsistent state.

Delta Lake provides transactional semantics on top of the data lake.

---

# 9. Delta CRUD Operations

Delta tables support SQL operations such as:

```sql
DELETE FROM my_table
WHERE age > 25;
```

and:

```sql
UPDATE my_table
SET status = 'inactive'
WHERE age > 25;
```

This is a major difference from treating a data lake as a collection of immutable files that must be manually rewritten.

---

# 10. `MERGE` and Upserts

Use `MERGE` when you need to synchronize incoming records with an existing Delta table.

Typical use case:

```text
Incoming CDC/data
       ↓
     MERGE
   ↙       ↘
match      no match
  ↓           ↓
UPDATE       INSERT
```

Example:

```sql
MERGE INTO target t
USING source s
ON t.id = s.id

WHEN MATCHED THEN
  UPDATE SET *

WHEN NOT MATCHED THEN
  INSERT *;
```

### Why use `MERGE`?

It supports **upsert** behavior:

- Existing records → update
- New records → insert

This is particularly useful for:

- CDC pipelines
- Incremental ingestion
- Deduplication/synchronization

### Interview answer

> "I'd use MERGE when incoming data can represent both new and existing records. It lets me express update-and-insert logic atomically rather than separately implementing UPDATE and INSERT operations."

---

# 11. Delta Time Travel

**Time travel** allows you to query or restore historical versions of a Delta table.

Conceptually:

```text
Version 0
   ↓
Version 1
   ↓
Version 2
   ↓
Version 3  ← current
```

You can access an earlier version when its required data files still exist.

### Important interview trap

Time travel metadata alone is not enough.

If old data files have been physically removed, the historical version cannot be reconstructed from the transaction log alone.

---

# 12. `VACUUM` and Time Travel

`VACUUM` removes old, unneeded data files.

This is useful for storage cleanup, but it can reduce the historical versions available for time travel.

The PDF's scenario:

> An engineer wants to restore a table to a version three days old, but the underlying files have been deleted.

Likely cause:

> **`VACUUM` was run with retention settings that removed those files.**

### Mental model

```text
Delta history
     ↓
Historical data files
     ↓
VACUUM
     ↓
Old files removed
     ↓
Some historical versions no longer recoverable
```

### Interview answer

> "`VACUUM` cleans up obsolete Delta data files. If it removes files required by an older table version, that version can no longer be reconstructed through time travel."

---

# 13. Databricks Repos / Git Integration

Databricks provides Git-backed development workflows through Repos.

The source question distinguishes Git operations that can be performed from those that should be handled through the external Git provider.

### Important workflow

```text
Databricks
   ↓
Create branch
   ↓
Develop
   ↓
Commit
   ↓
Push
   ↓
Git provider
   ↓
Pull Request / Merge
   ↓
Main branch
```

The source identifies **merge** as the operation performed outside the Databricks Repos UI in its tested workflow.

### Interview principle

> Databricks Repos integrates Git into development, but organizational code review and branch merging commonly remain part of the external Git workflow.

---

# 14. Development vs. Production

A good Databricks development workflow separates:

```text
Development
    ↓
Git branch
    ↓
Testing
    ↓
Code review
    ↓
Merge
    ↓
Production
```

Don't treat a shared production notebook as your development environment.

The goal is:

- Reproducibility
- Version control
- Code review
- Isolation
- Safer deployment

---

# 15. PySpark Access to SQL Tables

Spark SQL tables can be accessed through PySpark.

The PDF specifically tests:

```python
spark.table("sales")
```

This returns a DataFrame representing the table.

You can then use normal PySpark transformations:

```python
df = spark.table("sales")

df.filter(df.amount > 100)
```

### Interview point

> SQL and PySpark are different interfaces to the same Spark data-processing environment.

---

# 16. Databases and Schemas

A database organizes schemas and data objects.

Useful command:

```sql
DESCRIBE DATABASE customer360;
```

This provides information about the database, including its location.

### Mental model

```text
Database
   └── Schema
        ├── Tables
        ├── Views
        ├── Functions
        └── Other objects
```

---

# 17. Table Comments and Metadata

Databricks SQL allows descriptive metadata to be attached to tables.

Example:

```sql
CREATE TABLE CustomersInFrance
COMMENT "Contains PII"
AS
SELECT
    id,
    firstName,
    lastName
FROM customerLocation
WHERE country = 'FRANCE';
```

### Why metadata matters

Metadata can communicate:

- PII classification
- Business purpose
- Ownership
- Usage guidance
- Documentation

### Interview principle

> Governance isn't only access control; clear metadata helps users understand how data should be used.

---

# 18. Complex / Nested Data

Spark SQL array functions are useful for manipulating **complex and nested data**, particularly data originating from formats such as JSON.

Examples of relevant operations include:

- Extracting array elements
- Exploding arrays
- Transforming array contents
- Working with nested structs

### Interview answer

> "Spark SQL's complex-type functions are useful when semi-structured data contains arrays and nested objects. They let me transform nested data without first flattening everything externally."

---

# 19. SQL UDFs

Databricks SQL supports user-defined functions.

The source emphasizes that the function is created using:

```sql
CREATE FUNCTION
```

rather than:

```sql
CREATE UDF
```

Conceptually:

```sql
CREATE FUNCTION combine_nyc(city STRING)
RETURNS STRING
RETURN CASE
    WHEN city = 'brooklyn' THEN 'new york'
    ELSE city
END;
```

### When to use a SQL UDF

Use one when custom logic is:

- Reusable
- Parameterized
- Appropriate for SQL consumers

### Interview caveat

Don't create a UDF just because you can. Prefer built-in SQL/Spark functionality when it already solves the problem.

---

# 20. Conditional Execution with Python

Databricks notebooks can combine SQL with Python control flow.

For example:

```python
if day_of_week == 1 and review_period:
    # run final operation
```

The source tests two Python concepts:

- `==` means comparison.
- `=` means assignment.
- `and` is the logical operator.
- `True` is a Boolean, not the string `"True"`.

### Interview principle

Databricks notebooks support multi-language workflows, so orchestration/control logic can be implemented around SQL statements when necessary.

---

# 21. `COPY INTO`

`COPY INTO` loads files into a table.

Example:

```sql
COPY INTO transactions
FROM '/transactions/raw'
FILEFORMAT = PARQUET;
```

### Important behavior from the source

`COPY INTO` keeps track of files that have already been loaded.

Therefore, running the same command again does **not necessarily reload the same file**.

This provides useful protection against repeatedly ingesting identical files.

### Interview answer

> "`COPY INTO` is designed for incremental file ingestion and tracks previously loaded files, so rerunning the command doesn't normally reload files that have already been processed."

---

# 22. JDBC Data Sources

Databricks/Spark can read from relational databases through JDBC.

Conceptually:

```text
SQLite / Postgres / MySQL / etc.
             ↓
            JDBC
             ↓
        Spark DataFrame
             ↓
        Delta / other sink
```

Example pattern:

```sql
CREATE TABLE jdbc_customer360
USING org.apache.spark.sql.jdbc
OPTIONS (
    url 'jdbc:sqlite:/customers.db',
    dbtable 'customer360'
);
```

### Interview point

> JDBC is a connectivity mechanism for integrating Spark with external relational databases.

---

# 23. SQL Set Operations

The PDF tests the difference between joins and set operations.

If you want all rows from two compatible datasets and don't want duplicates:

```sql
SELECT * FROM march_transactions
UNION
SELECT * FROM april_transactions;
```

### `UNION`

Combines result sets and removes duplicate rows.

### `UNION ALL`

Combines result sets while preserving duplicates.

### `INTERSECT`

Returns rows present in both result sets.

### Interview distinction

```text
UNION     → combine
INTERSECT → common rows
JOIN      → combine columns based on relationships
```

---

# 24. Managed vs. External Tables

This is an important storage-management concept.

## Managed table

Databricks/Spark manages both:

- Table metadata
- Underlying data lifecycle

Dropping the table can remove its underlying data.

## External table

The table metadata points to data stored at an externally managed location.

Dropping the table removes the table definition, but the underlying files remain.

### Source scenario

```sql
DROP TABLE IF EXISTS my_table;
```

The table disappears from `SHOW TABLES`, but the data files remain.

Answer:

> **The table was external.**

### Mental model

```text
Managed:
DROP TABLE
   ↓
metadata + data lifecycle managed together

External:
DROP TABLE
   ↓
metadata removed
   ↓
external files remain
```

---

# 25. Tables vs. Views vs. Temporary Views

## Table

Use when the data entity:

- Must persist
- Must be physically stored
- Must be accessible across sessions
- Should be shared with other users

## View

Use when you want:

- A reusable logical query
- No separate stored result
- A virtual representation of data

## Temporary view

Use for:

- Session-specific logic
- Intermediate transformations
- Temporary SQL access

### Interview question

> "The data entity must be shared across sessions and physically stored. What should you create?"

Answer:

> **A table.**

---

# 26. Delta Live Tables / Pipeline Data Quality

The source identifies **Delta Live Tables (DLT)** as the Databricks capability used to build managed pipelines and automate data-quality checks.

A pipeline can define expectations/quality rules and monitor whether incoming data satisfies them.

Conceptually:

```text
Source
  ↓
Ingestion
  ↓
Pipeline
  ↓
Data quality expectations
  ↓
Curated tables
```

### Why use it?

- Declarative pipeline definitions
- Automated pipeline execution
- Data-quality monitoring
- Dependency management
- Operational visibility

### Interview answer

> "For managed Databricks pipelines with built-in data-quality expectations and monitoring, I'd use the DLT-style pipeline framework rather than implementing all of that orchestration and validation manually."

> **Terminology note:** The source uses the older "Delta Live Tables" terminology; Databricks has evolved this product into its newer Lakeflow pipeline tooling. For an interview, recognize both terms.

---

# 27. Continuous vs. Triggered Pipeline Execution

The source includes a scenario with:

- Streaming datasets
- Delta sources
- Production mode
- Continuous pipeline mode

The expected behavior is:

> The pipeline keeps updating datasets at intervals until it is stopped.

Compute resources:

> Are deployed for the pipeline and terminated when the pipeline is stopped.

### Mental model

```text
Continuous pipeline
       ↓
Deploy compute
       ↓
Process available data
       ↓
Wait / monitor
       ↓
Process new data
       ↓
Continue until stopped
```

### Contrast with triggered execution

```text
Triggered:
Start → process → finish

Continuous:
Start → process → continue → process → ...
```

---

# 28. Ownership and Administration

The source includes a scenario where an employee who owns Delta tables leaves the organization.

If the original owner no longer has access, the source identifies the **workspace administrator** as the person who can transfer ownership in the tested workflow.

### Interview principle

Ownership is an administrative/governance concern, not something a new lead engineer automatically inherits merely because they have a senior title.

Think:

```text
Data ownership
      ↓
Administrative permissions
      ↓
Authorized administrator
```

---

# 29. Cluster Pools vs. Other Development Features

Be able to distinguish common Databricks capabilities.

| Feature | Primary purpose |
|---|---|
| **Cluster pool** | Reduce cluster startup time |
| **Repos / Git** | Version control and collaboration |
| **Delta Lake** | Reliable transactional data storage |
| **Unity Catalog** | Governance/catalog/access control |
| **DLT/Lakeflow pipelines** | Managed data pipelines + quality |
| **Spark** | Distributed data processing |
| **Data Explorer** | Explore/manage data assets |

This is a useful interview framework because many Databricks questions are really asking:

> **"Which tool solves this specific problem?"**

---

# 30. High-Value Comparison Questions

## Delta Lake vs. Parquet

**Parquet**

> Efficient columnar file format.

**Delta Lake**

> Storage/table layer built around Parquet plus transaction-log metadata and table-management capabilities.

---

## Managed vs. External Table

**Managed**

> Databricks/Spark manages the data lifecycle.

**External**

> Metadata is managed by Databricks, but the underlying files remain externally managed.

---

## Table vs. View

**Table**

> Physically persists data.

**View**

> Persists a query definition rather than a separate result dataset.

---

## Table vs. Temporary View

**Table**

> Persistent and accessible across sessions.

**Temporary view**

> Session-scoped.

---

## `UNION` vs. `JOIN`

**UNION**

> Stacks rows from compatible result sets.

**JOIN**

> Combines columns from related rows based on a condition.

---

## `MERGE` vs. `INSERT`

**INSERT**

> Adds rows.

**MERGE**

> Handles matching and non-matching records, enabling upsert logic.

---

## `COPY INTO` vs. continuous pipelines

**COPY INTO**

> File-based ingestion command.

**Continuous pipeline**

> Continuously processes available/new data according to pipeline configuration.

---

# 31. Scenario Questions

## Scenario 1 — Two teams produce different reports

> Data engineering and analytics report different numbers for the same metric.

Strong answer:

> Establish a governed shared source of truth, with centralized cataloging/governance such as Unity Catalog, and standardize the data model used by both teams.

---

## Scenario 2 — Daily report must refresh immediately

> A scheduled report needs to start as quickly as possible.

Answer:

> Consider a cluster pool to reduce cluster startup latency.

---

## Scenario 3 — Bad data quality appears upstream

> Incoming data quality has deteriorated and you want automated monitoring.

Answer:

> Use DLT/Lakeflow pipeline data-quality expectations and monitoring.

---

## Scenario 4 — Need historical Delta state

> An engineer accidentally modified a Delta table.

Approach:

1. Check Delta history.
2. Identify the desired historical version.
3. Use time travel to inspect/recover it if the required files still exist.
4. Check whether `VACUUM` has removed those files.
5. Validate recovered data before replacing production data.

---

## Scenario 5 — Need to synchronize incoming CDC

> An upstream system sends inserts and updates.

Answer:

> Use `MERGE` into the target Delta table, using the business/record key to distinguish matches from new records.

---

## Scenario 6 — Need a persistent shared dataset

> Multiple engineers need the data across sessions and it must be physically stored.

Answer:

> Create a table, not a temporary view.

---

## Scenario 7 — Need to query an external database

> Data currently resides in SQLite or another relational database.

Answer:

> Use Spark's JDBC connector to access the source.

---

## Scenario 8 — Drop table but keep underlying files

> You want to remove the table definition but retain externally managed files.

Answer:

> Use an external table.

---

# 32. High-Value Interview Questions

### Q: What is the Databricks Lakehouse?

> "It's an architecture that combines the flexibility of data-lake storage with warehouse-like reliability, transactions, governance, and analytics capabilities."

### Q: What does Delta Lake provide?

> "Delta Lake adds a transaction and metadata layer on top of data-lake storage, providing ACID transactions, reliable updates/deletes, MERGE, time travel, and support for both batch and streaming workloads."

### Q: What is Unity Catalog?

> "Unity Catalog is Databricks' centralized governance and cataloging layer for managing access, discovery, and organization of data and other assets."

### Q: What is the difference between the control plane and data plane?

> "The control plane manages the Databricks environment, while the data plane contains the compute resources that execute workloads."

### Q: What is a cluster pool?

> "A pool keeps idle instances available so new clusters can start faster. It's primarily a startup-latency optimization."

### Q: What happens when you `VACUUM` a Delta table?

> "VACUUM removes obsolete data files. That can reduce storage usage, but if historical files are removed, older Delta versions may no longer be available for time travel."

### Q: Why use `MERGE`?

> "MERGE lets me implement upsert logic: update matching records and insert new records in one operation."

### Q: How does `COPY INTO` avoid duplicate file ingestion?

> "It tracks files that have already been loaded, so rerunning the command normally doesn't reload the same files."

### Q: Managed vs. external table?

> "With a managed table, the platform manages the table and underlying data lifecycle. With an external table, the table points to externally managed files, so dropping the table definition doesn't remove those files."

### Q: How do you access a SQL table from PySpark?

```python
df = spark.table("sales")
```

### Q: What is DLT/Lakeflow used for?

> "It's used to define and operate managed data pipelines, including dependency management and data-quality expectations."

---

# 33. Configuration / Command Cheat Sheet

## Delta

```sql
DELETE FROM my_table
WHERE age > 25;
```

```sql
MERGE INTO target t
USING source s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

## Time Travel

Conceptually:

```sql
SELECT *
FROM my_table VERSION AS OF 10;
```

or:

```sql
SELECT *
FROM my_table
TIMESTAMP AS OF '...';
```

## Database

```sql
DESCRIBE DATABASE customer360;
```

## PySpark

```python
df = spark.table("sales")
```

## File ingestion

```sql
COPY INTO transactions
FROM '/transactions/raw'
FILEFORMAT = PARQUET;
```

## SQL UDF

```sql
CREATE FUNCTION combine_nyc(city STRING)
RETURNS STRING
RETURN CASE
    WHEN city = 'brooklyn' THEN 'new york'
    ELSE city
END;
```

## Set operation

```sql
SELECT * FROM march_transactions
UNION
SELECT * FROM april_transactions;
```

## Conditional Python

```python
if day_of_week == 1 and review_period:
    ...
```

---

# 34. Common Interview Traps

### "A Delta table is one file."

No.

> It's a collection of data files plus Delta transaction-log metadata.

### "VACUUM deletes the transaction history."

Not exactly.

> VACUUM removes obsolete data files. The transaction log/history and physical data files have different roles.

### "More executors fix slow Databricks jobs."

Not necessarily.

> First identify the actual bottleneck.

### "Cluster pools make Spark jobs faster."

Not directly.

> They reduce cluster startup latency.

### "A view stores the data."

Normally no.

> A view stores a query definition.

### "A temporary view can be shared across sessions."

No.

> Temporary views are session-scoped.

### "MERGE is just INSERT."

No.

> MERGE supports matching and non-matching logic.

### "UNION and JOIN are interchangeable."

No.

> UNION combines rows; JOIN combines columns based on relationships.

### "External tables delete their files when dropped."

No.

> Dropping the external table removes the table metadata; the externally managed files remain.

### "A UDF is always better than built-in functions."

No.

> Prefer native functions when they already solve the problem.

### "COPY INTO reloads every file every time."

No.

> Previously loaded files are tracked and normally skipped.

---

# 35. Priority Study Order

## Tier 1 — Must Know

1. Databricks Lakehouse
2. Delta Lake
3. Delta vs. Parquet
4. ACID transactions
5. Delta tables and transaction logs
6. Time travel
7. `VACUUM`
8. `MERGE`
9. Managed vs. external tables
10. Unity Catalog
11. Control plane vs. data plane
12. Driver vs. workers
13. Cluster pools
14. `COPY INTO`
15. DLT/Lakeflow pipelines
16. Data-quality expectations
17. Tables vs. views vs. temporary views
18. Git/Repos workflow

## Tier 2 — Strongly Recommended

19. PySpark access to SQL tables
20. JDBC
21. SQL UDFs
22. Complex/nested data
23. `UNION` vs. `UNION ALL`
24. `INTERSECT`
25. Database/schema metadata
26. Table comments/metadata
27. Continuous vs. triggered pipelines
28. Ownership/admin concepts

## Tier 3 — Lower Priority

29. Detailed cluster configuration
30. Less-common SQL syntax
31. Fine-grained administrative edge cases

For your Data Engineering interviews, **Tier 1 + scenario questions** should be the focus.

---

# 36. Final Interview Checklist

- [ ] Explain what Databricks is.
- [ ] Explain the Lakehouse architecture.
- [ ] Explain why a lakehouse helps eliminate data silos.
- [ ] Explain Unity Catalog.
- [ ] Explain control plane vs. data plane.
- [ ] Explain driver vs. worker nodes.
- [ ] Explain cluster pools and their purpose.
- [ ] Explain Delta Lake.
- [ ] Explain Delta vs. Parquet.
- [ ] Explain Delta table storage.
- [ ] Explain ACID transactions.
- [ ] Explain `MERGE`.
- [ ] Explain Delta time travel.
- [ ] Explain `VACUUM`.
- [ ] Explain why VACUUM can affect historical recovery.
- [ ] Explain managed vs. external tables.
- [ ] Explain tables vs. views vs. temporary views.
- [ ] Explain `COPY INTO`.
- [ ] Explain how COPY INTO avoids reloading processed files.
- [ ] Explain DLT/Lakeflow pipelines.
- [ ] Explain data-quality expectations.
- [ ] Explain continuous vs. triggered execution.
- [ ] Explain Git/Repos workflows.
- [ ] Explain how to access tables from PySpark.
- [ ] Explain JDBC.
- [ ] Explain SQL UDFs.
- [ ] Explain nested/complex data.
- [ ] Explain `UNION` vs. `JOIN`.
- [ ] Explain ownership/admin concepts.
- [ ] Walk through a Delta recovery scenario.
- [ ] Walk through a CDC/upsert scenario.
- [ ] Walk through a data-quality scenario.
- [ ] Walk through a slow/failed Databricks pipeline scenario.

---

# 37. One-Minute Databricks Interview Answer

> "I think of Databricks as a unified data and analytics platform built around Spark and the lakehouse architecture. The lakehouse gives us flexible cloud-object storage while Delta Lake adds transactional reliability, schema management, time travel, and operations such as MERGE. Unity Catalog provides centralized governance and a shared source of truth across engineering, analytics, and data science. On the compute side, Databricks manages Spark clusters with a driver coordinating worker execution, while features such as cluster pools can reduce startup latency. For production pipelines, I can use file ingestion with COPY INTO or managed pipeline tooling, and implement data-quality expectations and incremental processing. From a development perspective, I would use Git-backed workflows and separate development from production. The important thing in an interview is not just knowing the features, but explaining which Databricks capability solves a particular engineering problem."

---

# 38. Final Mental Model

```text
                         DATABRICKS
                             │
             ┌───────────────┼────────────────┐
             ↓               ↓                ↓
          Storage          Compute         Governance
             │               │                │
        Delta Lake         Spark           Unity Catalog
             │               │                │
       ┌─────┼─────┐     ┌───┴────┐       Access
       ↓     ↓     ↓     ↓        ↓       Catalog
      ACID  Time   MERGE Driver  Workers   Lineage
            Travel
             │
             ↓
       Reliable Lakehouse
             │
       ┌─────┼───────────────┐
       ↓     ↓               ↓
    Batch  Streaming       Analytics
       │     │               │
       └─────┼───────────────┘
             ↓
      Production Pipelines
             │
       DLT / Lakeflow
       COPY INTO
       JDBC
       Git / Repos
```

> **Interview mindset:**  
> **Problem → Databricks capability → why it fits → trade-offs → operational considerations.**
