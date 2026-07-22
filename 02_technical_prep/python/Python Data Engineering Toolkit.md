# 🐍 Python Data Engineering Toolkit

## 1. Memory Efficiency & Streaming Large Datasets

When dealing with large production datasets, loading an entire file into RAM can crash your pipeline with an Out of Memory (OOM) error. Data engineers use generators and iterative streaming to process infinite data streams in a fixed memory footprint.

### Ⅰ. Generators and the `yield` Keyword
Instead of computing an entire array of results at once and keeping it in memory, a generator yields one item at a time on demand.

```python
import sys

# ❌ BAD: Memory Intensive (Loads everything into RAM)
def load_large_log_list(filepath):
    output = []
    with open(filepath, 'r') as f:
        for line in f:
            if "ERROR" in line:
                output.append(line.strip())
    return output  # Returns a massive list

#  GOOD: Memory Efficient (Streams line-by-line)
def stream_large_log_generator(filepath):
    with open(filepath, 'r') as f:
        for line in f:
            if "ERROR" in line:
                yield line.strip()  # Suspends execution, returns next item on demand

# Usage
log_stream = stream_large_log_generator("massive_production_log.txt")
print(next(log_stream))  # Outputs the first error line
```

### Ⅱ. Generator Expressions vs. List Comprehensions
* **List Comprehension (Memory Intensive):** `[line.upper() for line in lines]` (Creates a whole list in memory).
* **Generator Expression (Lazy Evaluation):** `(line.upper() for line in lines)` (Creates an iterator object, uses almost zero memory upfront).

---

## 2. Resource Management (Context Managers)

Pipelines must close database connections, file handles, and network sockets cleanly, even if a runtime processing error occurs. Failing to do so causes resource leaks.

### Custom Context Manager Design
Interviewers may ask you to build a custom tool to handle connection states or track execution time.

```python
from contextlib import contextmanager
import time

@contextmanager
def pipeline_timer(step_name):
    """Measures pipeline execution time and guarantees logs even if code fails."""
    start_time = time.time()
    print(f"🎬 Starting Pipeline Step: {step_name}")
    try:
        yield  # Code inside the 'with' block executes here
    finally:
        end_time = time.time()
        duration = end_time - start_time
        print(f"⏱️ Finished {step_name} in {duration:.2f} seconds.")

# Usage
with pipeline_timer("ETL Customer Ingestion"):
    # Simulate processing work
    time.sleep(2)
```

---

## 3. The Dataframe Divide: Pandas vs. PySpark vs. Polars

Data engineering candidates must know which library to use based on data volume, network overhead, and computational limits.

| Feature | Pandas | Polars | PySpark |
| :--- | :--- | :--- | :--- |
| **Execution** | Single-core, In-Memory. | Multi-threaded, In-Memory (Rust). | Distributed Cluster (JVM). |
| **Evaluation** | Eager (Computes instantly). | Lazy & Eager (Optimizes query plan). | Lazy (Builds DAG, runs on action). |
| **Ideal Scale** | Small datasets (< 2GB). | Medium datasets (2GB – 50GB). | Massive Scale (> 100GB / Terabytes). |

### 🚨 PySpark Lazy Evaluation & Shuffling Gotchas
PySpark does not compute operations immediately. It builds a **Directed Acyclic Graph (DAG)** of transformations.

```python
# 1. Transformations: Lazy (No data moves yet)
df_filtered = df.filter(df.country == "USA")
df_grouped = df_filtered.groupBy("state").count()  # Triggers an expensive network shuffle!

# 2. Action: Eager (Triggers the execution plan across the cluster)
df_grouped.show() 
```
* **Interview Alert:** Transformations that change the row shape or shuffle rows across machines (like `groupBy`, `join`, or `distinct`) are called **Wide Transformations**. They trigger network **Data Shuffling**, which is the primary driver of performance degradation in distributed Python pipelines.

---

## 4. Interview "Gotchas" & Pythonic Antipatterns

### 🚨 Mutating Mutable Default Arguments
* **The Pitfall:** Providing a mutable object (like a list or a dictionary) as a default function argument initializes that object only **once** when the module loads, causing shared state data leaks across pipeline loops.
* **The Antipattern:**
  ```python
  def append_to_batch(record, batch=[]):  # ❌ The batch list persists across calls!
      batch.append(record)
      return batch
  ```
* **The Data Engineering Fix:** Always default to `None` and instantiate the mutable structure dynamically inside the runtime loop:
  ```python
  def append_to_batch(record, batch=None):
      if batch is None:
          batch = []
      batch.append(record)
      return batch
  ```

### 🚨 Row-by-Row Iteration (`.iterrows()` in Pandas)
* **The Pitfall:** Using `for index, row in df.iterrows():` to transform data scales poorly. It loops through data point by data point in pure Python, discarding vectorized performance optimizations.
* **The Fix:** Use built-in vectorized column expressions or the `.apply()` method to run batch computations down columns natively in C/Rust.
