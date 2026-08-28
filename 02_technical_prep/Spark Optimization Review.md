# Spark Optimization Interview Prep

> **Core mental model:** Spark performance problems usually come down to **data distribution, data movement, memory pressure, storage layout, or execution overhead**. The interview skill is to explain **why** a job is slow, **what evidence** you would inspect, and **why** your fix addresses the root cause. The source PDF emphasizes systematic diagnosis rather than blindly adding executors.

---

# 1. The Five Major Performance Problems

| Problem | Core issue | Typical symptom |
|---|---|---|
| **Skew** | Uneven data distribution across partitions | One/few tasks run far longer |
| **Spill** | Intermediate data doesn't fit in memory | High spill metrics, slower stages |
| **Shuffle** | Data moves between partitions/executors | Expensive wide transformations |
| **Storage** | Inefficient file/directory layout | Slow reads, tiny files |
| **Serialization** | Data crosses execution boundaries | UDF overhead |

These can compound:

```text
Skew → oversized partition → memory pressure → spill → disk I/O → slower job
```

**Interview principle:** Don't immediately say "add more executors." Diagnose first.

---

# 2. Narrow vs. Wide Transformations

## Narrow

Each output partition depends on one input partition.

Examples:

- `map`
- `filter`
- `select`

These can generally be pipelined without data movement.

## Wide

An output partition may depend on multiple input partitions.

Examples:

- `groupBy`
- `join`
- `repartition`
- `sort`

Wide transformations require a **shuffle** and create a stage boundary. The PDF emphasizes that shuffles involve network transfer, disk I/O, and prevent pipelining across the boundary.

### Interview answer

> "Narrow transformations operate within existing partitions, so Spark can pipeline them without moving data. Wide transformations redistribute data across partitions, creating a shuffle and stage boundary. They're generally more expensive because of network transfer, disk I/O, and additional scheduling."

---

# 3. Data Skew

**Data skew** means records are distributed unevenly across partitions.

```text
Partition 1 → 1 GB
Partition 2 → 1 GB
Partition 3 → 1 GB
Partition 4 → 90 GB  ← skewed
```

The stage may wait for the oversized task even after all other tasks finish.

### Causes

- Highly imbalanced join keys
- Highly imbalanced aggregation keys
- Hot customers/products/accounts
- Poor partitioning

### Detecting skew

Use the Spark UI:

1. **Task duration distribution** — one/few tasks taking dramatically longer is a strong signal.
2. **Shuffle read distribution** — compare min, quartiles, median, and max.
3. **Spill** — oversized skewed partitions often spill.
4. **Data inspection** — check key frequencies:

```python
df.groupBy("key").count().orderBy("count", ascending=False).show()
```

The source specifically recommends checking Stage task distribution, shuffle-read metrics, spill, and actual key distributions.

## Fixing skew

### 1. AQE

Spark 3's **Adaptive Query Execution** can use runtime statistics to mitigate certain skewed joins and coalesce small shuffle partitions.

### 2. Databricks skew optimization

Databricks provides skew-related optimization features/hints for problematic joins.

### 3. Key salting

Create additional synthetic keys so a hot key is spread across multiple partitions.

```text
Original:
customer_id = 123

Salted:
123_0
123_1
123_2
...
123_N
```

Conceptually:

```text
Large table
   ↓
Salt skewed key
   ↓
Join on (key, salt)
   ↓
Distribute hot key
   ↓
Remove salt
```

The salt range must be chosen experimentally:

- Too small → skew remains
- Too large → too many small partitions

Only salt skewed keys when possible; salting adds complexity and processing cost.

### Skewed aggregations

For aggregations, consider **two-phase aggregation**:

```text
Raw records
    ↓
Partial aggregation
    ↓
Shuffle smaller results
    ↓
Final aggregation
```

This reduces the amount of data shuffled.

---

# 4. Spill

**Spill** occurs when Spark moves intermediate data from memory to disk because it cannot keep it in memory.

```text
Executor memory
      ↓
Not enough space
      ↓
Spill to disk
      ↓
Read back later
```

Spill can prevent OOM, but disk I/O makes jobs slower.

### Common causes

- Data skew
- Oversized partitions
- Large joins
- Large aggregations
- `explode()` causing a large row expansion
- Excessively large input partitions

### Detecting spill

Spark UI exposes:

- **Spill (Memory)**
- **Spill (Disk)**

Look at stage/task/executor details.

### Fixing spill

1. **Check skew first.**
2. Increase executor memory if the workload legitimately requires it.
3. Reduce partition size by increasing partition count.
4. Tune:
   - `spark.sql.shuffle.partitions`
   - `spark.sql.files.maxPartitionBytes`
5. Use `repartition()` or `coalesce()` appropriately.

**Important:** More memory can hide the symptom without fixing the root cause.

---

# 5. Shuffle

**Shuffle redistributes data between partitions**, commonly for:

- `join`
- `groupBy`
- `distinct`
- `sort`
- `orderBy`
- `repartition`

Why it's expensive:

```text
Write intermediate data
        ↓
Network transfer
        ↓
Read on downstream executors
        ↓
Next stage
```

The source notes emphasize that shuffle is often **necessary**, so the goal is to reduce expensive/unnecessary shuffle rather than eliminate all shuffle.

## Reduce shuffle cost

### Filter/project before wide transformations

Prefer:

```text
100 GB
 ↓ filter/select
10 GB
 ↓ shuffle
```

over shuffling unnecessary rows/columns.

### Broadcast small tables

Avoid shuffling the large side when a small side can safely be replicated.

### Avoid unnecessary wide transformations

Review repeated:

- `groupBy`
- `distinct`
- `repartition`
- `sort`

### Data modeling

If an expensive transformation is run constantly, consider precomputing/denormalizing the result when the storage and maintenance cost is justified.

---

# 6. Broadcast Join vs. Sort-Merge Join

## Broadcast join

The smaller relation is copied to executors, so the large relation does not need to shuffle on the join key.

```text
Small table ──→ Executor 1
             ├→ Executor 2
             ├→ Executor 3
             └→ Executor 4

Large table partitions stay local
```

The source notes cite a default `spark.sql.autoBroadcastJoinThreshold` of **10 MB**. You can explicitly request broadcast:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "customer_id"
)
```

### Trade-off

Broadcast eliminates shuffle but requires:

- Enough executor memory
- Network transfer to executors
- A safely small replicated relation

Don't blindly raise the threshold because an oversized broadcast can cause memory pressure/OOM.

## Sort-merge join

Both large sides are shuffled by join key, sorted, then merged.

```text
Large A → shuffle → sort ─┐
                          ├→ merge
Large B → shuffle → sort ─┘
```

Best for large-large joins.

### Interview answer

> "I'd use broadcast when one side is small enough to replicate safely because it eliminates the large-side shuffle. For large-large joins, I'd generally use or expect a sort-merge join. I'd verify the physical plan and statistics rather than relying only on table-size assumptions."

---

# 7. Choosing `spark.sql.shuffle.partitions`

The source notes cite **200** as the common default, but the correct value depends on workload.

A useful starting rule from the notes:

> **Target roughly 100–200 MB of shuffle data per partition.**

Approximation:

```text
partitions ≈ shuffle data size / target partition size
```

Example:

```text
100 GB
÷ 200 MB
≈ 500 partitions
```

### Too few

```text
Large partitions
 → memory pressure
 → spill/OOM
 → poor parallelism
```

### Too many

```text
Many tiny tasks
 → scheduling overhead
 → potentially tiny output files
```

With AQE enabled, Spark can coalesce small shuffle partitions based on runtime statistics.

**Interview answer:**

> "I wouldn't blindly use 200. I'd estimate shuffle volume, choose a reasonable target partition size, consider available cores and memory, and use AQE to adapt where appropriate."

---

# 8. Input Partition Sizing

`spark.sql.files.maxPartitionBytes` controls the approximate amount of file data Spark packs into an input partition.

The source cites **128 MB** as the common default.

```text
Files
  ↓
Input partitioning / file binning
  ↓
Tasks
```

Larger partitions:

- Fewer tasks
- Less scheduling overhead
- More memory per task

Smaller partitions:

- More parallelism
- More scheduling overhead

Tune based on:

- File sizes
- Dataset size
- Available cores
- Executor memory
- Downstream output requirements

---

# 9. Tiny Files

A **small-file problem** occurs when a dataset contains too many small files.

Example:

```text
10,000 files × 5 MB
```

can be much less efficient than a reasonable number of appropriately sized files.

### Why?

Each file introduces:

- Open/close overhead
- Metadata operations
- Scheduling overhead
- Directory/listing overhead

### Detect

Use Spark UI SQL/read information to inspect:

- Number of files read
- Scan time
- File sizes where available

### Fix

1. Compact existing files.
2. Configure ingestion to write larger files.
3. Avoid excessive output partitions.
4. Use `coalesce()` or appropriate `repartition()` before writing.
5. Tune `spark.sql.shuffle.partitions`.

The source notes explicitly connect excessive partitioning to tiny output files and recommend compaction and better partition sizing.

---

# 10. `repartition()` vs. `coalesce()`

## `repartition()`

Redistributes data and normally causes a shuffle.

Use when you need to:

- Increase partitions
- Redistribute data
- Partition by a key
- Correct poor distribution

```python
df = df.repartition(400, "customer_id")
```

## `coalesce()`

Primarily reduces the number of partitions without a full shuffle.

Use when you simply need fewer partitions and can tolerate the existing distribution.

```python
df = df.coalesce(50)
```

### Mental model

```text
repartition → redistribute
coalesce    → reduce
```

---

# 11. Directory Scanning

Too many storage partitions/directories can create scan overhead.

Example:

```text
/year=2026/month=01/day=01/
/year=2026/month=01/day=02/
...
```

Thousands of sparsely populated directories can be costly.

### Detect

Inspect scan time under Spark UI SQL/read operations.

### Fix

- Choose storage partitions based on query patterns.
- Avoid very high-cardinality partition columns.
- Avoid over-partitioning small datasets.
- Use table/catalog metadata where appropriate.

**Principle:**

> Partition stored data according to common access patterns—not every column available.

The source specifically calls out excessive directories and recommends smarter storage partitioning.

---

# 12. Schema Inference and Merging

Schema handling can add read overhead.

### Prefer explicit schemas

When practical, define the schema rather than repeatedly inferring it.

### Parquet

Parquet stores schema information with the file.

### Schema merging

The source references:

```text
spark.sql.parquet.mergeSchema
```

Schema merging can become expensive when hundreds/thousands of part-files each need to be inspected and merged.

Mitigations:

- Provide schemas explicitly.
- Avoid unnecessary schema evolution.
- Use table/catalog metadata where appropriate.
- Use formats/platforms with strong schema-management capabilities.

---

# 13. Serialization and UDFs

Native Spark expressions are preferable because Spark's optimizer can reason about them.

Python UDFs can introduce a JVM ↔ Python boundary:

```text
Spark JVM
   ↓ serialize
Python
   ↓ execute
Python result
   ↓ serialize
Spark JVM
```

This creates overhead, and arbitrary UDF code can limit Catalyst's ability to optimize the surrounding expression.

### Best practice

Prefer:

- Native Spark SQL functions
- Built-in DataFrame functions
- Higher-order functions

before UDFs.

## If Python logic is unavoidable

A **Pandas UDF** can use Apache Arrow to transfer batches rather than individual records, reducing serialization overhead compared with a regular Python UDF.

### Interview answer

> "I'd first replace the UDF with native Spark functions if possible. If Python logic is unavoidable, I'd consider a Pandas UDF because Arrow enables batch transfer and vectorized processing."

---

# 14. AQE — Adaptive Query Execution

AQE allows Spark to adapt execution using runtime statistics.

Important capabilities:

- **Coalesce shuffle partitions**
- **Handle certain skewed joins**
- **Adapt some join strategies at runtime**

### Why it matters

Without AQE, Spark has to make many decisions using information available before execution.

With AQE:

```text
Initial plan
    ↓
Execute / observe runtime statistics
    ↓
Adapt plan
    ↓
Continue more efficiently
```

### Interview answer

> "AQE improves performance by using runtime statistics to adapt execution. It can coalesce small shuffle partitions, mitigate certain skewed joins, and change some join strategies. I would generally keep it enabled unless there's a specific workload or compatibility reason not to."

---

# 15. OOM Debugging

Never begin with:

> "Increase executor memory."

First identify **where** the OOM occurs.

## Driver OOM

Possible causes:

- `collect()` on a large dataset
- `toPandas()` on a large dataset
- Excessive driver-side state
- Oversized broadcast-related state

Dangerous:

```python
df.collect()
```

## Executor OOM

Possible causes:

- Data skew
- Oversized partitions
- Large broadcast relation
- Too few shuffle partitions
- Large intermediate data

### Spark UI

Check:

- Failed stage
- Task distribution
- Spill
- Memory
- Executor behavior
- GC time

The source notes use roughly **10% GC time** as a heuristic for memory pressure; treat that as a signal, not a universal cutoff.

### Mental model

```text
OOM
 ↓
Driver or executor?
 ↓
Which stage?
 ↓
Skew?
Partition size?
Broadcast?
collect/toPandas?
Too few partitions?
 ↓
Targeted fix
```

---

# 16. Caching, Persisting, and Checkpointing

## `cache()`

Convenient persistence using the default storage level.

For DataFrames, the default is commonly:

```text
MEMORY_AND_DISK
```

Use when a dataset is reused and recomputation is expensive.

```python
df.cache()
df.count()  # materializes cache
```

Caching is lazy.

## `persist()`

Lets you choose the storage level.

```python
df.persist(...)
```

Useful when the default cache behavior isn't appropriate.

## `checkpoint()`

Writes to reliable storage and **truncates lineage**.

Use when:

- Lineage becomes very long
- Recomputing lineage is expensive
- Long-running iterative/streaming workloads need lineage truncation

### Key distinction

```text
cache/persist
 → reuse data
 → lineage remains

checkpoint
 → write reliable result
 → lineage truncated
```

The source PDF explicitly tests this distinction as an interview differentiator.

---

# 17. Storage Levels

The source lists these common levels:

| Storage level | Memory | Disk | Serialized | Mental model |
|---|---:|---:|---:|---|
| `MEMORY_ONLY` | ✓ | ✗ | No | Fast when it fits |
| `MEMORY_AND_DISK` | ✓ | ✓ | No | Memory with disk fallback |
| `MEMORY_ONLY_SER` | ✓ | ✗ | Yes | Lower memory footprint, more CPU |
| `MEMORY_AND_DISK_SER` | ✓ | ✓ | Yes | Serialized + disk fallback |
| `DISK_ONLY` | ✗ | ✓ | Yes | Disk persistence |
| `OFF_HEAP` | Off-heap | ✗ | Yes | Specialized/off-heap |

### Practical choice

- Fits comfortably in memory → `MEMORY_ONLY` can be appropriate.
- Memory constrained → consider serialized or memory-and-disk persistence.
- Don't persist to disk unless avoiding recomputation justifies the I/O/storage cost.

---

# 18. Cartesian Products

A Cartesian product produces:

```text
rows(A) × rows(B)
```

Example:

```text
1,000 × 1,000 = 1,000,000 rows
```

### Causes

- Missing join condition
- Accidental `CROSS JOIN`
- Non-equality predicates leading to nested-loop strategies
- Unexpected row multiplication

### Detect

Inspect the physical plan for:

```text
CartesianProduct
BroadcastNestedLoopJoin
```

Also compare input and output cardinalities.

### Fix

- Add the correct join condition.
- Filter unnecessary data first.
- Handle nulls deliberately.
- Use explicit `CROSS JOIN` only when intentional.
- Verify expected cardinality.

The source recommends using the physical plan and row-count explosion as diagnostic signals.

---

# 19. Systematic Performance-Debugging Framework

This is the **highest-value interview skill**.

If asked:

> "The Spark job suddenly became 5× slower. What do you do?"

Use this sequence:

## Step 1 — Input volume

Did:

- Input size increase?
- File count increase?
- Partition sizes increase?

## Step 2 — Data distribution

Did:

- A key become skewed?
- A hot key appear?

## Step 3 — Physical plan

Look for:

- Changed join strategy
- Extra shuffle
- Cartesian/nested-loop join
- Missing filters
- UDFs

## Step 4 — Spark UI

Compare good vs. bad runs:

- Stage durations
- Task-duration distribution
- Shuffle read/write
- Spill
- Task count
- Executor utilization

## Step 5 — Statistics

Could stale statistics be causing a worse query plan?

## Step 6 — Cluster/resource contention

Ask:

- Are other jobs competing for resources?
- Did available resources change?
- Is the cluster shared?

## Step 7 — Storage layout

Check:

- Tiny-file proliferation
- Excessive directories
- Schema-merging overhead
- Storage partitioning

The interview PDF explicitly recommends comparing data volume, distribution, statistics, cluster contention, small files, and Spark UI metrics between good and bad runs.

### Mental model

```text
Slow job
   ↓
Data volume?
   ↓
Distribution?
   ↓
Physical plan?
   ↓
Shuffle / spill?
   ↓
Storage?
   ↓
Resources?
   ↓
Targeted fix
   ↓
Measure again
```

---

# 20. High-Value Interview Questions

## Q1. One task takes 30 minutes while 199 take 10 seconds. What's happening?

> "I'd suspect data skew. I'd inspect task-duration distribution, shuffle-read and spill metrics, then inspect the join/aggregation key distribution. If confirmed, I'd consider AQE skew handling or salting for a skewed join, and two-phase aggregation for aggregation skew."

The source explicitly frames this as a root-cause diagnosis problem rather than an executor-count problem.

---

## Q2. Broadcast vs. sort-merge join?

> "Broadcast is appropriate when one side is small enough to replicate safely because it eliminates the large-side shuffle. For large-large joins, sort-merge is generally more appropriate. I'd inspect the physical plan and statistics and use a broadcast hint when I have evidence the automatic decision is wrong."

---

## Q3. Job suddenly becomes 5× slower with no code changes?

> "I'd compare good and bad runs systematically: input volume, file count, data distribution, physical plan, shuffle, spill, statistics, resource contention, and storage layout. I'd use Spark UI metrics to identify exactly where the regression occurred."

---

## Q4. Narrow vs. wide transformations?

> "Narrow transformations keep output partitions dependent on one input partition and can be pipelined without shuffle. Wide transformations can require data from multiple input partitions, creating shuffle and a stage boundary."

---

## Q5. How do you choose shuffle partitions?

> "I'd estimate shuffle volume and target a reasonable partition size, roughly 100–200 MB as a starting point from these notes. I'd consider available cores and memory, and use AQE to coalesce small partitions."

---

## Q6. Spark OOM — what do you do?

> "First determine driver vs. executor OOM. Then identify the failing stage and inspect memory, spill, task distribution, broadcast size, and partition sizes. Only after identifying the cause would I change memory or partitioning."

---

## Q7. How does AQE help?

> "AQE uses runtime statistics to adapt the plan. It can coalesce shuffle partitions, mitigate certain skewed joins, and adapt some join strategies."

---

## Q8. How do you choose input partition size?

> "I balance task parallelism against per-task memory and scheduling overhead. Larger partitions reduce task count but increase memory requirements; smaller partitions increase parallelism but can create scheduling overhead."

---

## Q9. How do you identify a Cartesian product?

> "I'd inspect the physical plan for `CartesianProduct` or `BroadcastNestedLoopJoin`, compare input/output cardinality, and verify the join predicate. If accidental, I'd add the correct join condition or otherwise fix the logic."

---

## Q10. `cache()` vs. `persist()` vs. `checkpoint()`?

> "`cache()` is convenient default persistence, `persist()` lets me choose the storage level, and both preserve lineage. `checkpoint()` writes to reliable storage and truncates lineage, so I use it when lineage has become too long or recomputation is too expensive."

---

# 21. Scenario Questions

## Scenario 1 — Large skewed join

> A 2 TB fact table joins a 50 GB dimension table. One customer accounts for 40% of the fact records.

Discuss:

1. Inspect physical plan.
2. Determine whether the dimension is safely broadcastable.
3. Inspect task/shuffle distribution.
4. Confirm key skew.
5. Use AQE skew handling where appropriate.
6. Salt the skewed key if necessary.
7. Verify the fix doesn't create excessive small tasks.

---

## Scenario 2 — 100,000 tiny Parquet files

Discuss:

1. Inspect output partition count.
2. Determine why too many partitions were written.
3. Reduce output partitions using `coalesce()` or an appropriate repartition strategy.
4. Tune shuffle partitioning if relevant.
5. Compact existing files.
6. Fix upstream ingestion if it continuously recreates the problem.

---

## Scenario 3 — Executor OOM during join

Investigate:

- Join strategy
- Broadcast size
- Shuffle partition sizes
- Skew
- Spill
- Executor memory
- Input/output cardinality

Potential fixes:

- Remove unsafe broadcast.
- Increase shuffle partitions.
- Handle skew.
- Filter/project before the join.
- Increase memory only if justified.

---

## Scenario 4 — Driver OOM

Look for:

```python
collect()
toPandas()
```

Move work back into distributed Spark processing whenever possible.

---

## Scenario 5 — Python UDF makes pipeline slow

1. Replace it with native Spark functions if possible.
2. If unavoidable, consider a Pandas UDF.
3. Inspect the physical plan.
4. Compare runtime and resource usage.

---

## Scenario 6 — Slow job with no code change

Use:

```text
Input volume
    ↓
File count
    ↓
Data distribution
    ↓
Physical plan
    ↓
Shuffle / spill
    ↓
Statistics
    ↓
Cluster contention
    ↓
Storage layout
```

Do not jump directly to a larger cluster.

---

# 22. Common Interview Traps

### "Add more executors."

Doesn't fix one skewed partition.

### "Shuffle is bad."

Shuffle is often necessary. Minimize expensive/unnecessary shuffle.

### "Increase executor memory."

May hide a skew/partitioning problem.

### "Broadcast every dimension table."

Broadcast only when replication is safe.

### "Use 200 shuffle partitions."

The default isn't universally optimal.

### "UDFs are always bad."

Native functions are preferable, but UDFs can be necessary. Explain the serialization/optimization trade-off.

### "More partitions are always better."

Too many partitions create scheduling overhead and potentially tiny files.

### "Caching always improves performance."

Caching helps only when reuse saves enough recomputation to justify the memory/storage cost.

### "Checkpoint is just a slower cache."

No: the critical distinction is **lineage truncation**.

---

# 23. Spark UI Quick Reference

| Spark UI area | What to investigate |
|---|---|
| **Jobs** | Overall duration, slow jobs |
| **Stages** | Stage duration, shuffle, spill |
| **Tasks** | Outliers, input, shuffle, spill, GC |
| **SQL / Query details** | Physical plan, joins, scans |
| **Executors** | Memory, GC, utilization |

Mental model:

```text
Jobs   → Which job?
Stages → Which stage?
Tasks  → Which partition/task?
SQL    → What execution plan?
```

---

# 24. Configuration Cheat Sheet

| Configuration | Purpose | Interview point |
|---|---|---|
| `spark.sql.shuffle.partitions` | Default shuffle partition count | Tune to workload |
| `spark.sql.files.maxPartitionBytes` | Input partition sizing | Larger isn't always better |
| `spark.sql.autoBroadcastJoinThreshold` | Automatic broadcast threshold | Notes cite 10 MB default |
| `spark.sql.adaptive.coalescePartitions.enabled` | AQE partition coalescing | Reduces tiny shuffle partitions |
| `spark.sql.parquet.mergeSchema` | Parquet schema merging | Can be expensive across many files |

---

# 25. Priority Study Order

## Tier 1 — Must Know

1. Narrow vs. wide transformations
2. Shuffle
3. Data skew
4. Spill
5. Spark UI diagnosis
6. Broadcast vs. sort-merge joins
7. `spark.sql.shuffle.partitions`
8. AQE
9. `repartition()` vs. `coalesce()`
10. Driver vs. executor OOM
11. UDF/serialization overhead
12. Systematic troubleshooting

## Tier 2 — Strongly Recommended

13. Key salting
14. Two-phase aggregation
15. Tiny files
16. Input partition sizing
17. Directory scanning
18. Schema inference/merging
19. Cache vs. persist vs. checkpoint
20. Storage levels
21. Cartesian products

## Tier 3 — Lower Priority

22. Bucketed datasets
23. Spill listeners
24. Fine-grained storage-level details
25. Less-common configuration flags

For interviews, **Tier 1 + scenarios** matter more than memorizing every tuning knob.

---

# 26. Final Interview Checklist

- [ ] Explain narrow vs. wide transformations.
- [ ] Explain why wide transformations are expensive.
- [ ] Define shuffle.
- [ ] Explain why shuffle can't always be eliminated.
- [ ] Define data skew.
- [ ] Detect skew using Spark UI.
- [ ] Explain salting.
- [ ] Explain two-phase aggregation.
- [ ] Explain AQE.
- [ ] Define spill.
- [ ] Detect and mitigate spill.
- [ ] Choose `spark.sql.shuffle.partitions`.
- [ ] Explain too few vs. too many partitions.
- [ ] Explain broadcast joins.
- [ ] Explain sort-merge joins.
- [ ] Explain broadcast memory trade-offs.
- [ ] Explain `repartition()` vs. `coalesce()`.
- [ ] Diagnose tiny files.
- [ ] Diagnose driver vs. executor OOM.
- [ ] Explain UDF serialization overhead.
- [ ] Explain why native Spark functions are preferred.
- [ ] Explain Pandas UDFs.
- [ ] Explain `cache()` vs. `persist()` vs. `checkpoint()`.
- [ ] Explain Cartesian products.
- [ ] Walk through a 5× performance regression systematically.
- [ ] Explain where to look in Spark UI.
- [ ] Explain how you would validate an optimization.

---

# 27. One-Minute Spark Optimization Answer

> "I approach Spark optimization by diagnosing the bottleneck before changing resources. I'd start with the Spark UI and identify the slow job, stage, and task distribution, then inspect shuffle read/write, spill, executor behavior, and the physical plan. I'd determine whether the problem is data skew, excessive shuffle, memory pressure, partition sizing, join strategy, storage layout, or serialization overhead. For example, if one task is much slower than the others, I'd investigate skew and consider AQE or salting. If one join side is safely small, I'd consider broadcasting it. I'd filter and project before wide transformations to reduce shuffle volume, use appropriate partition counts, and prefer native Spark functions over Python UDFs. Finally, I'd compare the optimized run with the original to verify the improvement rather than assuming the change worked."

---

# 28. Final Mental Model

```text
                    SLOW SPARK JOB
                          │
                          ↓
                    Spark UI
                          │
                          ↓
                 Identify bottleneck
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
      Data              Compute           Storage
       │                  │                  │
    Skew?              Spill/OOM?        Tiny files?
    Shuffle?           GC pressure?      Directories?
    Join?              Partitions?       Schema?
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                    Targeted Fix
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
      AQE              Broadcast          Partition
      Salting          Join               tuning
      Filtering        Better plan        Compaction
      Pre-agg          Memory tuning      Schema
      Native funcs
                          │
                          ↓
                     Measure Again
```

> **Senior-level Spark performance thinking:**  
> **Evidence → bottleneck → root cause → targeted fix → validation.**

The source PDF's strongest interview message is exactly this diagnostic mindset: understand *why* the job is slow and use Spark UI/runtime evidence instead of trial-and-error tuning.
