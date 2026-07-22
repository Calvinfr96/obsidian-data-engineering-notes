# 💾 SQL Optimization Principles

## 1. Core Evaluation Order (How the Engine Thinks)
Writing optimized SQL requires understanding that the database engine does not execute code from top to bottom. It evaluates clauses in a specific logical order, meaning data filtering should happen as early as possible.

```text
1. FROM / JOIN  ➔ Identifies the target datasets and executes cross-products/joins
2. WHERE        ➔ Filters out rows at the individual row level before aggregation
3. GROUP BY     ➔ Groups the filtered data into buckets
4. HAVING       ➔ Filters aggregated buckets (Never use HAVING for row-level filters)
5. SELECT       ➔ Extracts and evaluates the exact columns and expressions
6. WINDOW       ➔ Executes window calculations (DENSE_RANK, SUM OVER, etc.)
7. DISTINCT     ➔ Deduplicates the final output rows
8. ORDER BY     ➔ Sorts the final result dataset
9. LIMIT / FETCH➔ Truncates the payload returned to the client
```

---

## 2. Advanced Window Functions & "Gotchas"

Window functions compute values over a specified partition of rows without collapsing the underlying dataset into a single row.

### Core Syntax Blueprint
```sql
SELECT 
    user_id,
    login_date,
    -- Tracks running total of logins per user over time
    COUNT(login_date) OVER(
        PARTITION BY user_id 
        ORDER BY login_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_login_count
FROM user_logins;
```

### 🚨 Crucial Interview Distinction: Rank vs. Dense Rank vs. Row Number
Interviewers frequently ask you to find the "top N items per category" (e.g., top 3 highest-paid employees per department). Choosing the wrong function leads to logic bugs.

* **`ROW_NUMBER()`**: Assigns a sequential integer starting at 1. Never repeats values. If there is a tie, it arbitrarily picks an order.
* **`RANK()`**: Assigns duplicate ranks for tied values. However, it **skips** subsequent ranks. (e.g., 1, 2, 2, 4).
* **`DENSE_RANK()`**: Assigns duplicate ranks for tied values, but **never skips** numbers. (e.g., 1, 2, 2, 3). Always use this for financial or strict tiering questions.

---

## 3. High-Performance Optimization Strategies

When working with large data warehouses (Snowflake, BigQuery, Redshift), performance issues are rarely caused by disk storage limitations; they are caused by memory pressure and network bottlenecks.

### Ⅰ. Minimize Network Data Shuffling
* **The Problem:** Joining two massive tables on un-indexed or un-partitioned keys forces the cluster nodes to copy data across the network to align corresponding keys on the same physical server. This is called **Shuffling** and it destroys query speed.
* **The Fix:** 
  * Always join a large `Fact Table` to a small `Dimension Table` (Broadcast Join).
  * Filter down your tables inside subqueries or Common Table Expressions (CTEs) *before* passing them to a complex multi-way `JOIN`.

### Ⅱ. Avoid Predicate Sifting (Non-Sargable Queries)
* **The Problem:** Wrapping database columns inside scalar functions prevents the query optimizer from leveraging table indexes or partition boundaries.
* **The Bad (Non-Sargable):**
  ```sql
  SELECT user_id FROM sales WHERE YEAR(transaction_date) = 2026;
  ```
* **The Good (Sargable):**
  ```sql
  SELECT user_id FROM sales WHERE transaction_date >= '2026-01-01' AND transaction_date <= '2026-12-31';
  ```

### Ⅲ. Optimize Joins and Aggregations
* **Join Order:** Place the most restrictive table filter early or join the largest table first depending on the engine's query optimizer layout.
* **Avoid `SELECT *`:** Columnar storage databases only read the blocks of columns requested. Using `SELECT *` forces the engine to pull every single column asset from remote storage, completely negating the benefit of a columnar database.
* **Replace `LIKE '%string%'`:** Leading wildcards disable standard index usage. If you must search strings at massive scale, leverage native cloud search optimization indexes or pre-tokenized fields.

---

## 4. Common Technical Interview Pitfalls

### 🚨 The `NULL` Value Aggregate Trap
* **The Behavior:** Standard aggregate functions like `COUNT(column)`, `AVG(column)`, and `SUM(column)` completely **ignore** `NULL` entries.
* **The Trap:** `COUNT(*)` counts the physical rows in a table regardless of data content. `COUNT(customer_id)` counts rows where `customer_id` is explicitly not null. If an interviewer asks you to calculate an average metric, verify whether `NULL` values should be evaluated as `0` or discarded.
* **The Fix:** Use `COALESCE(column, 0)` to handle null values safely:
  ```sql
  SELECT AVG(COALESCE(revenue, 0)) FROM store_sales;
  ```

### 🚨 `NOT IN` vs. `NOT EXISTS` with Nulls
* **The Trap:** If a subquery returns even a single `NULL` value, a `WHERE column NOT IN (subquery)` expression evaluates to completely empty and returns 0 rows due to three-valued logic (`TRUE`, `FALSE`, `UNKNOWN`).
* **The Fix:** Use `NOT EXISTS` or a `LEFT JOIN ... WHERE right_table.key IS NULL`, which handle null values reliably without breaking your filter logic.
