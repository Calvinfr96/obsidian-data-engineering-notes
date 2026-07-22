# 🗄️ Data Modeling Cheat Sheet

## 1. Core Architecture Foundations

### OLTP vs. OLAP

| Feature             | OLTP (Online Transaction Processing)           | OLAP (Online Analytical Processing)                   |
| :------------------ | :--------------------------------------------- | :---------------------------------------------------- |
| **Primary Use**     | Operational, daily application transactions.   | Data analysis, business intelligence, reporting.      |
| **Design Priority** | Fast writes, high concurrency, low latency.    | Fast complex reads, aggregations over large datasets. |
| **Storage Engine**  | Row-oriented (e.g., PostgreSQL, MySQL).        | Columnar (e.g., Snowflake, BigQuery, Redshift).       |
| **Data Format**     | Highly Normalized (3NF) to prevent redundancy. | Denormalized (Star/Snowflake) to minimize joins.      |

### 💡 Columnar Storage Deep Dive (Interview Favorite)
* **How it works:** Rows are split; data for each column is stored sequentially on disk.
* **Why it matters:** 
  1. **I/O Efficiency:** If a query only needs `SELECT SUM(revenue)`, it reads the revenue block on disk and skips all other column blocks entirely.
  2. **High Compression:** Identical data types stored sequentially allow aggressive compression algorithms (Run-Length Encoding, Dictionary Encoding), saving storage and RAM.

---

## 2. Dimensional Modeling

### Fact Tables vs. Dimension Tables
* **Fact Tables:** Contain quantitative metrics or measurements resulting from a business event.
  * *Characteristics:* Highly numeric, millions/billions of rows, long and narrow, contains foreign keys linking to dimensions.
  * *Example:* `fact_orders` (`order_id`, `customer_key`, `date_key`, `order_amount`, `quantity`).
* **Dimension Tables:** Contain descriptive attributes, context, and text about the business entities.
  * *Characteristics:* Highly textual, smaller row count, wide and descriptive.
  * *Example:* `dim_customers` (`customer_key`, `first_name`, `email`, `city`, `tier`).

### Star Schema vs. Snowflake Schema
* **Star Schema:** Dimension tables are completely **denormalized**.
  * *Pros:* Simplest design, optimal query performance (fewer joins), easily understood by BI tools.
  * *Cons:* Redundant data storage in dimensions.
* **Snowflake Schema:** Dimension tables are **normalized** into sub-dimensions (e.g., splitting `dim_customers` into a separate `dim_cities` table).
  * *Pros:* Minimizes data redundancy, easier structural maintenance.
  * *Cons:* Degrades query performance due to complex multi-way joins; column store engines struggle with deep normalization.

---

## 3. Advanced Dimensional Design Patterns

### Slowly Changing Dimensions (SCD)
How do you track dimensional attributes that change over time? (e.g., a customer moves from Chicago to Seattle).

* **SCD Type 0 (Retain Original):** Never change the historical value. The attribute remains static forever.
* **SCD Type 1 (Overwrite):** Overwrite the old value with the new value. History is lost completely.
* **SCD Type 2 (Add New Row):** **(Most Important for Interviews)** Create a new row with a new surrogate key. Use tracking flags to track validity windows.
  * *Schema implementation:* Requires `start_date`, `end_date`, and `is_current` (boolean/flag) columns.
* **SCD Type 3 (Add New Column):** Add a `previous_value` column to track the immediate past state alongside the current state.
* **SCD Type 4 (History Table):** Keep the base table as SCD Type 1, but route all historical changes out to a dedicated log/shadow table.

### Fact Table Types
1. **Transactional Fact:** Records a specific measurement event at a point in time (e.g., a single retail check-out scan).
2. **Periodic Snapshot:** Summarizes activity occurring over a predefined regular interval (e.g., daily bank balance, monthly performance report).
3. **Accumulating Snapshot:** Tracks the progression of a defined process that has a clear start and end with multiple milestone steps (e.g., order fulfillment pipeline: Ordered ➔ Shipped ➔ Delivered).

---

## 4. Interview "Gotchas" & Trade-offs

### 🚨 Star Schema in the Cloud Era
* **Traditional Wisdom:** Minimize joins because CPU and memory were bottlenecked.
* **Modern Cloud Data Warehouses (Snowflake/BigQuery):** They treat Star Schemas differently. Columnar engines handle flat tables or wide structures incredibly well. However, joining very large tables can trigger expensive **Data Shuffling** across clustered storage nodes. 
* **The Trade-Off:** Don't normalize unless you explicitly need to. Modern architecture favors light denormalization over complex deep nesting to minimize network shuffle operations.

### 🚨 Null Values in Dimensions
* **The Pitfall:** Storing absolute `NULL` markers inside a Dimension Foreign Key in a Fact Table breaks referential integrity when executing an `INNER JOIN`.
* **The Data Engineering Fix:** Replace the `NULL` value in the Fact Table key with a dummy key pointing to a fallback row in your Dimension Table (e.g., Key `-1` maps to a row labeled "Unknown" or "Not Applicable").
