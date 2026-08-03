## Data Warehousing & Lakehouses

- **The Problem**:
	- Suppose you're running an online store.
	- Every purchase generates:
		- Order ID
		- Customer ID
		- Items
		- Payment
		- Shipping
	- Your application stores everything in **PostgreSQL**. Life is good.
	- Now, your CEO asks: "Show me total revenue by state, product category, and month for the last five years."
	- Can PostgreSQL answer all of these questions? Technically, yes. But practically, it shouldn't.
### OLTP vs. OLAP
- **Online Transactional Processing (OLTP)**:
	- Used to support operational applications.
	- Examples:
		- Banking
		- E-commerse
		- Hospital systems
		- Airline reservations
	- Typical Queries:
		```sql
		SELECT *
		FROM Orders
		WHERE order_id = 12345;
		```
		- Very small. Very fast. Thousands per second.
	- Characteristics:
		- Many writes
		- Small reads
		- **Row-oriented**
		- **Highly normalized**
		- Low latency
- **Online Analytical Processing (OLAP)**:
	- Used to analyze historical data.
	- Typical Queries:
		```sql
		SELECT
		    state,
		    category,
		    SUM(revenue)
		FROM sales
		GROUP BY state, category;
		```
		- Huge scans. Millions or billions of rows. Fewer queries. **Much more expensive**.
	- Characteristics.
		- Many reads
		- **Large scans**
		- **Heavy aggregation**
		- **Column-oriented**
- **Why Not Use PostgreSQL?**
	- Imagine your website receives 10,000 purchases/sec.
	- Meanwhile, an analyst launches the following query across 5 billion rows:
		```sql
		SELECT *
		FROM orders
		```
	- The database suddenly starts reading enormous amounts of data.
	- Application performance suffers.
	- Customers experience slow checkouts.
	- The workloads interfere with one another.
- **The Solution**:
	- Separate them:
		```
		Application
		
		↓
		
		OLTP Database
		
		↓
		
		ETL / ELT
		
		↓
		
		Data Warehouse
		
		↓
		
		Analytics
		```
	- The warehouse exists so analytical workloads don't impact operational systems.

### Data Warehouse Characteristics
- A warehouse is optimized for:
	- Large scans
	- Aggregations
	- Historical analysis
	- Concurrent analysts
	- Reporting
	- Business Intelligence
- Not:
	- Updating one customer record
	- Processing a credit card
	- Reserving an airline seat

### Columnar Storage
- Columnar storage is one of the biggest innovations in modern data warehouses.
- Traditional databases store data by grouping it into individual rows.
- A columnar warehouse stores data by grouping it into columns.
- **Why Is This Faster?**
	- This is much faster because queries that target specific columns don't need to scan entire rows to find the required information. Only the required columns need to be scanned.
	- Columnar storage significantly reduces disk I/O.
- **Compression**:
	- Columns typically contain similar values. This attribute makes them much easier to compress.
	- This leads to smaller storage, less disk I/O, and faster queries.
- **Massively Parallel Processing (MPP)**:
	- Modern warehouses don't rely on one server. Instead, each node scans part of the data and the results are combined.
	- This is very similar to Spark executors.
- **Real Data Engineering Pipeline**:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet (S3)
	
	↓
	
	Snowflake
	
	↓
	
	Power BI
	```

### Tradeoffs
- OLTP
	- Pros:
		- Fast transactions
		- Low latency
		- Strong consistency
		- Frequent updates
	- Cons:
		- Poor for analytics
- Data Warehouse:
	- Pros:
		- Massive analytical queries
		- High concurrency
		- Columnar Storage
		- Compression
		- Parallel Execution
	- Cons:
		- Not designed for transactional workloads
		- Higher latency for small writes

### Mental Model
- Connection to Previous Topics:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Airflow
	
	↓
	
	Snowflake
	
	↓
	
	Dashboard
	```
	- OLTP and OLAP aren't mutually exclusive. They work together to solve different problems.
	- OLTP: A cashier at a grocery store processing individual customer transactions
	- OLAP: A warehouse inventory office analyzing things like top-performing products for a given quarter, regional sales, and top-performing suppliers.

### Common Interview Questions
1. Why not run analytics directly against PostgreSQL?
> 	Operational databases are optimized for many small transactional reads and writes, while analytical queries scan large portions of historical data and perform aggregations. Running those workloads on the same system can negatively impact application performance. Data warehouses separate analytical processing from transactional processing and use techniques like columnar storage and massively parallel processing to optimize large-scale analytics.
	- PostgreSQL acts primarily as a database management system meant for transactional reads and writes. It is not meant to perform heavy analytical workloads.
2. Your company currently stores all application data in PostgreSQL. Business analysts frequently run complex reports that aggregate several years of sales data. The engineering team has noticed that application performance slows dramatically whenever these reports run. How would you explain why this is happening? What architectural change would you recommend? Why would it help?
> 	Application performance degrades when these reports are run because the PostgreSQL database is optimized for fast transactions and frequent updates. It is not designed for analytical queries that scan large amounts of data. I would recommend moving the data to a data warehouse and performing analytical queries against that. Data warehouses are optimized for analytical workloads mainly because they implement columnar storage and compress data very well. Separating operational and analytical workloads prevents long-running reporting queries from competing with customer-facing transactions for database resources.
	- A database is optimized for heavy writes and small reads.
	- A data warehouse is optimized for large scans and heavy aggregation.
3. A teammate says: "Columnar storage just rearranges the data. Why would that make queries any faster?" How would you explain the performance benefits of columnar storage? When is it most effective? Can you think of workloads where row-oriented storage is still the better choice?
> 	Columnar storage allows for faster queries because columns are more easily compressed than rows, saving disk I/O. Additionally, queries that target specific columns are optimized by ignoring irrelevant columns when scanning data. Row-oriented storage is better suited transactional workflows, such as querying specific records by a primary key, not for heavy aggregation.
4. If columnar storage is so much faster, why don't we store everything in columns?
	- Because OLTP systems are typically used to perform queries such as:
		```sql
		SELECT *
		FROM customers
		WHERE customer_id = 123;
		```
		- This query needs the **entire row**.
		- A row-oriented database can retrieve it with a single read.
		- A column store may need to assemble data from multiple column files.
	- Column stores excel at reading a few columns from many rows.
	- Row stores excel at reading many columns from a single row.
	- Neither is universally better.

## Partitioning vs. Clustering

### Partitioning
- **Partitioning in a data warehouse is NOT the same thing as partitioning in Kafka or Spark**.
- The idea of "splitting data" is similar, but the **goal** is different.
	- **Kafka:** Increase parallelism and preserve ordering.
	- **Spark:** Distribute computation across executors.
	- **Data Warehouse:** Reduce how much data needs to be read from storage.
- Suppose you have a Sales table with 10 billion rows. An analyst runs:
	```sql
	SELECT *
	FROM sales
	WHERE sale_date = '2026-08-01';
	```
	- Should the warehouse read all 10 billion rows to answer this question?
	- Probably not.
- **Partitioning** physically divides a table into smaller pieces based on a column.
	- For example, when a table is partitioned by `sale_date`, the storage might look like:
		```
		2026-07-30/
		
		2026-07-31/
		
		2026-08-01/
		
		2026-08-02/
		```
		- This allows the **warehouse** (not Spark) to eliminate most of the data.
- **Partition Pruning**:
	- This is the real benefit of partitioning.
	- Because the query specified `WHERE sale_date = '2026-08-01'`, the warehouse only reads the `2026-08-01` directory. This is called **partition pruning**. This is the exact same idea behind Parquet format.
- **Good Partition Keys**:
	- A good partition key:
		- Appears frequently in filters.
		- Eliminates large portions of the table.
		- **Doesn't create too many tiny partitions**.
	- Examples:
		- Date
		- Year
		- Month
		- Region (sometimes)
	- Bad partition keys:
		- Keys, such as `customer_id`, that create an excessive amount of tiny partitions. This is called **high cardinality**.
		- High cardinality creates a lot of metadata overhead and tiny files.
		- This leads to poor query planning and performance.

### Clustering
- Now, suppose an analyst runs: `WHERE customer_id = 12345`.
	- You can't reasonably partition by `customer_id` because there are too many unique values.
	- Instead, you **cluster** by `customer_id`.
	- Unlike partitioning, clustering doesn't create separate directories. Instead, it organizes data **within partitions**.
	- Within a partition, rows with similar `customer_id` values are grouped together.
	- This reduces how much data must be scanned.

### Snowflake Micro-Partitions
- Snowflake works a little bit differently. You don't manually create partitions. Instead, Snowflake automatically creates **micro-partitions**.
- Each micro-partition contains metadata such as:
	- Minimum value
	- Maximum value
	- Distinct values
- Similar to partitioning in traditional warehouses, micro-partitioning in Snowflake allows more for more efficient scans because queries that filter based partition key allow sections of data to be skipped during scanning.

### Clustering in Snowflake
- Snowflake can also maintain a clustering key. This improves pruning for columns that:
	- Are frequently filtered.
	- Have large tables.
	- Aren't naturally well organized.
- Unlike partitioning, clustering is often an optimization rather than a necessity.
- **Data Engineering Example**:
	- Suppose your table receives 100 million rows/day.
	- Almost every report filters by `sale_date`.
	- Partitioning (or Snowflake's automatic micro-partitioning) is ideal.
	- Later, analysts start frequently investigating customers by `customer_id`.
	- Adding a `customer_id` clustering key helps keep data organized within each `sale_date` partition, improving query performance **without creating millions of tiny partitions**.

### Tradeoffs
- Too Few Partitions:
	- Cons: Every query scans large amounts of data.
- Too Many Partitions:
	- Cons:
		- Metadata overhead
		- Tiny files
		- Slower planning
		- Poor performance
- Over-Clustering:
	- Cons:
		- Consumes compute
		- Requires data reorganization
		- Only worthwhile if query performance improves enough to justify the cost
- **Partitioning vs. Clustering**:

| Partitioning                            | Clustering                                              |
| --------------------------------------- | ------------------------------------------------------- |
| Physically divides data                 | Organizes data within partitions                        |
| Best for low-cardinality filter columns | Best for frequently filtered higher-cardinality columns |
| Enables partition pruning               | Improves pruning within partitions                      |
| Too many partitions hurt performance    | Too many clustering columns increase maintenance        |

### Mental Model
- Connection to Previous Topics:
	- Kafka partitioning mainly **benefits parallelism**.
	- Spark partitioning mainly **distributes computation**.
	- Data Warehouse partitioning mainly **reduces storage scans**.
- Same word. Different purpose.

### Common Interview Questions
1. Why not partition by customer_id?
> 	Partitioning works best with relatively low-cardinality columns that are commonly used for filtering, such as dates. Partitioning by customer_id could create millions of partitions, increasing metadata overhead and hurting performance. A better approach is often to partition by date and cluster by customer_id if customer-based filtering is common.
2. Your sales table contains 15 billion rows. Nearly every dashboard filters by: `WHERE sale_date BETWEEN...`. Occasionally, analysts also filter by: `WHERE customer_id = ...`. How would you organize this table? Would you partition, cluster, both, or neither? Why?
> 	I would organize the table by partitioning by sale_date, then clustering by customer_id. Partitioning by sale_date creates a reasonable number of partitions, while clustering by customer_id allows for better organization within a partition. Partitioning and Clustering combined lead to improved query performance. Clustering increases the likelihood that rows for a given customer are stored close together, allowing the warehouse to prune more data within a partition.
3. A teammate proposes partitioning a table by `transaction_id` because every transaction ID is unique. Would you agree? Why or why not? What problems could this create?
> 	I would mention that partitioning by a high cardinality key, such as transaction_id, severely degrades query performance by creating a lot of tiny partitions. This creates an excessive amount of metadata. It also makes query planning and execution more difficult because the warehouse spends more effort managing metadata and planning queries across millions of partitions, while also suffering from many tiny files and inefficient storage organization.
4. Suppose almost every query filters by both `sale_date` and `region`. Would you partition by both?
	- It depends. I'd ask:
		- How many distinct regions are there?
		- How often is region filtered independently?
		- What does the resulting partition count look like?

## Star Schema vs. Snowflake Schema

- Data warehouses intentionally optimize for reading, even if that means duplicating data. This is the opposite of OLTP design.
- Suppose an analyst asks: "Show total revenue by product category and customer state." Where does the data come from? Not from one table. It comes from multiple business entities:
	- Sales
	- Customers
	- Products
- How should these tables be organized?
- **Fact Tables**:
	- The center of a warehouse is usually a fact table. Facts record **measurable business events**.
	- Examples include:
		- Sales
		- Orders
		- Payments
		- Shipments
		- Clicks
	- Fact tables contain:
		- Foreign keys
		- Numerical measures
	- Fact tables don't contain a lot **descriptive information**.
- **Dimension Tables**:
	- Dimensions describe facts.
- **Star Schema**:
	```
	               Customer
	                   │
	                   │
	Product ─── Fact Sales ─── Date
	                   │
	                   │
	               Store
	```
	- Dimension tables surround a central fact table.
	- Benefits:
		- Simple joins: The fact table contains foreign keys, which are the primary keys of the surrounding dimension tables. For example, `sale_id` might be the primary key of the Sales table, but `customer_id`, `product_id`, and `date` will be foreign keys.
- **Denormalization**:
	- Denormalization is the practice of repeating redundant information in a table.
	- For example, the `state` column in the Customers table will have a lot repeated values for customers that live in the same state.
	- This is done intentionally in data warehouses to reduce the number of joins that need to be performed for analysis.
	- Reading is more important than reducing storage.
- **Snowflake Schema**:
	- Unlike a denormalized star schema, where information is repeated to reduce joins, a snowflake schema **normalizes** data by separating it into different tables.
	- For example, instead of having `customer_id` and `state` in the same table, they are separated into different tables.
	- This causes the schema to grow into an increasing number of branches, like a snowflake.

### Surrogate & Natural Keys
- Suppose customer IDs come from an external operational system. Today, a particular customer has an ID of `100`.
	- Tomorrow, the company migrates systems and that same customer has their ID changed to `5000100`.
	- Warehouse joins would break.
	- Instead, the warehouse creates a **surrogate key** for the customer.
	- The customer ID from the source system is referred to as the **natural key**.
	- This is much simpler because it decouples the data warehouse from the source system.
	- Many warehouses prefer surrogate keys in dimensions.

### Tradeoffs
- Star Schema:
	- Pros:
		- Fewer joins
		- Faster queries
		- Simpler SQL
		- Easier for analysts
	- Cons:
		- More duplicated data
		- Slightly larger storage
- Snowflake Schema:
	- Pros:
		- Less redundancy
		- Easier to maintain shared reference data
		- Better normalization
	- Cons:
		- More joins
		- More complex queries
		- Often slower analytical performance

### Mental Model
- Connection to Previous Topics:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	```
	- In this pipeline, Spark isn't just cleaning data.
	- It's often transforming operational tables into fact and dim tables before loading them into the warehouse.

### Common Interview Questions
1. Why do data warehouses often use a star schema instead of a fully normalized schema?
> 	Analytical workloads prioritize fast reads over minimizing storage. A star schema denormalizes dimension tables, reducing the number of joins required for common analytical queries. This simplifies SQL and often improves query performance, while the additional storage cost is usually acceptable in a warehouse.
2. You're designing a warehouse for an e-commerce company. Analysts frequently ask questions like:
	- Revenue by product category
	- Revenue by customer state
	- Revenue by month
	- How would you model this data? What would be fact tables? What would be dimension tables? Would you choose a star schema or a snowflake schema? Why?
> 	I would model the data by using a star schema and creating a fact table for sales, then supporting dim tables for customers, products, dates, and stores. While this denormalized approach would increase data duplication, it would allow for much faster analytical queries by reducing the amount of joins that need to be performed. For many modern data warehousing platforms, storage is cheaper than compute, so this is a fair tradeoff.
3. A teammate says: "We should normalize every dimension table because duplicate data is always bad database design." How would you respond? What tradeoffs would you discuss? Why are data warehouse design principles different from OLTP database design?
> 	While normalized dimension tables significantly reduce data duplication, it's not always a bad design choice. The choice ultimately comes down to business requirements. If minimizing redundancy, maintaining reference data, or enforcing normalization is the higher priority, a snowflake schema may be appropriate. If analysis was more important, a denormalized star schema would be more efficient.
	- In modern data warehouses, snowflake schemas are often chosen not because they optimize storage costs, but because:
		- Shared reference data is easier to maintain.
		- Updates to dimensions are centralized.
		- Data consistency is easier to enforce.
4. If star schemas duplicate data, doesn't that create update problems?
	- Much less than in OLTP systems.
	- Warehouse dimensions change relatively infrequently.
	- In this case, optimizing reads makes a lot more sense than optimizing updates.

## Slowly Changing Dimensions

- Slowly Changing Dimensions isn't about **whether** a table can be update when information changes, it's about **how** a table should be updated.
	- Should history be preserved or overwritten?
- A Slowly Changing Dimension (SCD) is a strategy for handling changes to dimension data over time.
- Examples of Changing Attributes:
	- Customer:
		- Address
		- State
		- Marital status
	- Product:
		- Category
		- Brand
	- Employee:
		- Department
		- Manager
- SCD determines how these changes are stored.

### SCD Type 1
- SCD type 1 simply overwrites the old value. No historical context is preserved.
- Advantages:
	- Simple
	- Small storage
	- Easy queries
- Disadvantages:
	- History is lost
- Appropriate Use Cases:
	- A customer updating their email address.
	- In most practical applications, no historical context related to a customer's old email address is ever needed.

### SCD Type 2
- Historical context is preserved by creating a new row when an entity's attributes are updated.
- Before:

| customer_sk | customer_id | state    | current |
| ----------- | ----------- | -------- | ------- |
| 1           | 100         | Virginia | Y       |
- After:

|customer_sk|customer_id|state|current|
|---|---|---|---|
|1|100|Virginia|N|
|2|100|Texas|Y|
- The surrogate key changed, but the customer ID remained the same. The surrogate key changes because it refers to the *version* of the entity, not the entity itself.
- In the table above, the `current` column is used to track the current version. When a customer's info is updated, the old row's `current` attribute is updated, then the new row is added with `current` set to `Y`.
- Most SCD Type 2 tables also contain dates specifying when a version was created and deprecated:

|customer_sk|state|effective_from|effective_to|
|---|---|---|---|
|1|Virginia|2021-01-01|2024-03-15|
|2|Texas|2024-03-15|NULL|
- When fact tables are properly joined with dimension tables using columns such as `customer_sk`, historical reports remain correct. Each historical sale refers to the version of the customer that was active at the time.
- In the table above, the `effective_to` column is used to track the current version. When a customer's info is updated, the old row's `effective_to` is updated, then a new row is added with `effective_to` set to `NULL`.

### Type 1 vs. Type 2
- **Type 1**:
	- Old data is lost.
	- Tables are smaller and easier to maintain.
- **Type 2**:
	- Old data is preserved.
	- Tables are larger and more difficult to maintain.
	- More complex ETL.
	- Much richer analytics.

### ETL Process
- Suppose the following Airflow pipeline runs nightly:
	```
	Source Database
	
	↓
	
	Spark
	
	↓
	
	Compare Dimension
	
	↓
	
	Insert New Version
	
	↓
	
	Snowflake
	```
	- Spark detects a customer's state has changed.
- Instead of updating the row directly, it:
	1. Marks the old row as no longer current.
	2. Inserts a new row.
	3. Generates a new surrogate key.

### Tradeoffs
- Type 1:
	- Pros:
		- Simple
		- Smaller Storage
		- Easier maintenance
	- Cons:
		- No historical tracking
- Type 2:
	- Pros:
		- Complete history
		- Historical reporting
		- Time-travel analytics
	- Cons:
		- More storage
		- More ETL complexity
		- Larger dimension tables

### Mental Model
- Connection to Previous Topics:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Detect Customer Changes
	
	↓
	
	Update DimCustomer
	
	↓
	
	Load Snowflake
	```
	- Spark performs the SCD logic.
	- Airflow schedules it.
	- Snowflake stores the results.

### Common Interview Questions
1. When would you use SCD Type 2?
> 	I'd use Type 2 when historical attribute values affect reporting or business analysis. Instead of overwriting existing records, Type 2 creates a new dimension row with a new surrogate key while preserving previous versions using effective dates or current flags. This allows fact tables to remain associated with the correct historical dimension values.
2. A customer changes their **shipping address**. Your business wants historical reports to show sales using the address that was valid **at the time of the sale**, not the customer's current address. Would you implement SCD Type 1 or Type 2? Walk me through how the dimension table would change. Why is a surrogate key important here?
> 	I would use SCD Type 2 for this specific implementation because the business requires historical reports to use the historical address, not the currently active address. When a customer's shipping address is updated in the dimension table, the new record is inserted with the updated address and given a unique surrogate key. The 'current' column is set to Y. The old record's 'current' column is changed from Y to N to indicate the record is no longer active.
	- It's also worth mentioning effective dates, since a lot of modern SCD Type 2 implementations use both 'current' flags and effective dates.
3. A teammate says: "Type 2 dimensions are always better because they preserve more information." Would you agree? Can you think of attributes where **Type 1** is actually the better choice? What tradeoffs would you discuss?
> 	Type 2 is only better when business requirements mandate historical information be preserved. SCD Type 2 introduces more complex ETL logic and longer dimension tables because each version of a record is given its own row. SCD Type 1 uses simpler ETL logic and keeps dimension tables relatively short. A scenario in which SCD Type 1 would be an appropriate implementation is updating a customer's email address or phone number, or correcting a typo.
4. If you're using Type 2, why can't the fact table just reference `customer_id`?
	- Because `customer_id` identifies the **business entity**, not the **historical version**.

## Data Lakes vs. Data Warehouses vs. Lakehouses

- Suppose a company stores all of its analytical data in a warehouse. Life is good.
	- All of a sudden, the company starts collecting:
		- JSON logs
		- Clickstream events
		- Images
		- PDFs
		- IoT sensor data
		- Application logs
	- All of this is semi-structured or unstructured data.
	- Can a traditional warehouse efficiently store all of this? Not really.
- **Data Warehouse**:
	- Structured data
	- Schema defined before loading
	- Optimized for SQL analytics
	- High performance
	- Strong governance
- Traditional data warehouses aren't designed to efficiently store unstructured data.
- **Data Lake**:
	- Instead of loading only cleaned, transformed data, a data lake stores all types of data.
	- Data lakes are optimized for:
		- Cheap storage
		- Massive scale
		- Any file format
		- Raw data retention
	- Data lakes are **not** optimized to perform efficient SQL queries.
	- A data warehouse needs to know the schema of a dataset before loading it.
	- A data lake can store **raw** data and interpret the schema at a later point. This is called **schema-on-read**. This improves flexibility.
- **Problems With Data Lakes**:
	- Since a data lake can store massive amounts of unstructured data, it can quickly become a **data swamp** if the data isn't managed properly.
	- The following problems can lead to a messy data lake:
		- No governance.
		- No quality rules.
		- No reliable metadata.
		- Too many versions.
		- Everyone wrote files differently.
### Lakehouse Architecture
```
Object Storage

↓

Parquet

↓

Delta / Iceberg / Hudi

↓

SQL

↓

Spark

↓

Analytics
```
- A lakehouse offers the flexibility of a data lake and the reliability of a data warehouse.
- A lakehouse adds features that data lakes lacked:
	- ACID transactions
	- Schema enforcement
	- Schema evolution
	- Time travel
	- Versioning
	- Metadata management

### ACID Transactions
- ACID transactions are a set of four key database properties—**atomicity**, **consistency**, **isolation**, and **durability**—that guarantee safe and reliable data processing. They ensure that all parts of a data operation complete successfully or none of them do, protecting system integrity from crashes and errors.
	- **Atomicity:** Treats the transaction as a single, all-or-nothing unit; if one step fails, the entire process is undone.
	- **Consistency:** Ensures the database moves from one valid, rule-abiding state to another, preserving all data constraints.
	- **Isolation:** Keeps concurrent transactions separate so they do not interfere with each other or read half-finished data.
	- **Durability:** Guarantees that once a transaction is finished and saved, the changes are permanent even if the system loses power
- Suppose a Spark job writes 1 million rows but crashes halfway through:
	- A traditional data lake would perform partial writes, resulting in half-written files and a corrupted table.
	- A lakehouse prevents this by treating the entire operation as a single pass-fail event. The **entire** transaction rolls back and readers never see partial data.
	- This is exactly what you'd expect from a data base.


### Lakehouse Properties
- **Schema Enforcement**:
	- Suppose today's data consists of:
		```
		customer_id
		revenue
		```
	- Tomorrow, someone accidentally sends:
		```
		customerId
		revenue
		```
		- Without schema enforcement, tables become inconsistent.
		- With schema enforcement, **the write fails**.
- **Schema Evolution**:
	- Now suppose the business adds a new `discount_amount` to the existing schema.
		- We don't want to go back and rewrite 500 TB of data to add the column to historical data (just for the field to be null or a default value anyways).
		- Lakehouse formats support **controlled** schema evolution.
- **Time Travel**:
	- One of the coolest features of lakehouse architecture.
	- Suppose yesterday's pipeline corrupted the table.
	- Instead of restoring backups:
		- Query yesterday's version.
		- Alternatively, rollback.
- **Delta Lake**:
	- Created by Databricks.
	- Adds:
		- ACID transactions
		- Time travel
		- Schema enforcement
		- Efficient MERGE operations
	- Works very well with Spark.
- **Apache Iceberg**:
	- Another modern table format.
	- Focuses heavily on:
		- Huge datasets
		- Engine independence
	- Compatible with:
		- Spark
		- Trino
		- Flink
		- Snowflake
- **Apache Hudi**:
	- Optimized for:
		- Incremental ingestion
		- Frequent updates
		- Streaming workloads
	- Very popular for CDC (batch processing) pipelines.
- **Real Pipeline Example**:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Delta Lake
	
	↓
	
	Airflow
	
	↓
	
	Snowflake
	
	↓
	
	Power BI
	```

### Tradeoffs
- Warehouse:
	- Pros:
		- Excellent SQL
		- Strong governance
		- Fast analytics
	- Cons:
		- Less flexible
		- Typically supports **structured** data only.
- Data Lake:
	- Pros:
		- Cheap
		- Flexible
		- Any data
	- Cons:
		- Weak governance
		- Can become a 'data swamp' without careful maintenance
- Lakehouse:
	- Pros:
		- Flexible
		- ACID transaction
		- Time travel
		- Schema evolution
		- SQL support
	- Cons:
		- More operational complexity
		- Newer ecosystem
		- Tool compatibility varies

### Mental Model
- Connection to Previous Topics:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Delta Lake
	
	↓
	
	Airflow
	
	↓
	
	Snowflake
	
	↓
	
	Dashboard
	```
- Imagine a warehouse building:
	- A **data lake** is like a huge storage warehouse where boxes are stacked wherever there's room. It's inexpensive and flexible, but finding the right box can become difficult without organization.
	- A **data warehouse** is like a retail store. Everything is organized, labeled, and ready for customers, but only selected products are displayed.
	- A **lakehouse** combines both ideas: the inexpensive storage of the warehouse building with the organization and inventory management of the retail store.

### Common Interview Questions
1. Why did lakehouse architecture become popular?
> 	Traditional data lakes provided inexpensive, flexible storage for raw data but lacked features such as ACID transactions, schema enforcement, and reliable metadata management. Lakehouse architecture combines the scalability and flexibility of data lakes with many of the reliability and governance features of data warehouses, allowing organizations to use a single storage layer for both data engineering and analytics.
2. Your company currently stores raw JSON logs, CSV files, images, and Parquet datasets in Amazon S3. Analysts complain that:
	- Different teams define schemas differently.
	- Tables occasionally become corrupted after failed writes.
	- Recovering from bad ETL jobs is difficult.
	- Would you recommend continuing with a traditional data lake or moving to a lakehouse? Why? Which lakehouse features specifically address these problems?
> 	I would recommend switching to a lakehouse. A lakehouse combines the scalability and flexibility of data lakes with the reliability and governance of data warehouses. A lakehouse also allows raw data to remain flexible while enforcing consistent schemas for curated datasets, preventing different teams from accidentally introducing incompatible table definitions. ACID transactions would ensure writes either completely succeed or completely fail. Time travel would make recovering from bad ETL jobs easier.
	- A lakehouse doesn't encourage every team to define completely independent schemas for the same curated dataset. Instead, it provides:
		- **Schema enforcement** to prevent accidental writes with incompatible schemas.
		- **Schema evolution** to intentionally add or modify columns in a controlled way.
		- Each team contributes their own columns to a unified schema and only uses parts that are relevant to them. Data governance ensures teams don't create incompatibilities within this unified schema. 
3. A teammate says: "We already have Parquet files in S3. Why do we need Delta Lake or Iceberg? Aren't they just different file formats?" How would you respond? What additional capabilities do Delta Lake and Iceberg provide beyond storing data in Parquet?
> 	I would respond by saying that Parquet is a storage format that mainly allows for efficient compression and querying, while Delta Lake and Iceberg are table formats built on top of Parquet (or similar columnar files) that provide additional capabilities. Delta lake offers features such as ACID transactions, schema enforcement, and time travel, which making writes, data governance, and failure recovery easier. Iceberg is efficient for storing huge datasets.
	- Delta Lake and Iceberg also add:
		- Transaction logs
		- Metadata
		- `MERGE` support
		- Version management
		- Snapshot management
		- Schema tracking
	- They're not replacing Parquet. They're **managing Parquet files**.
	- Delta Lake efficiently supports update and merge semantics while maintaining ACID guarantees. This is a huge advantage for CDC pipelines.
4. If Delta Lake already provides ACID transactions and SQL capabilities, why would a company still use Snowflake?
> 	They solve related but different problems. Delta Lake provides reliable, transactional storage on object storage and integrates well with processing engines like Spark. Snowflake is a fully managed analytical database that provides compute management, workload isolation, security, governance, and highly optimized SQL execution. Many organizations actually use both—Delta Lake as the storage layer for engineering pipelines and Snowflake as the platform analysts use for interactive analytics.

## Medallion Architecture (Bronze, Silver, Gold)

- Suppose your company receives data from multiple sources:
	- Website
	- Mobile app
	- Third-party vendors
	- CRM system
- Each source has their own set of problems:
	- Missing fields
	- Duplicate records
	- Different schemas
	- Invalid values
- Should analytics query the **raw** data directly? Probably not.
- The core idea behind the medallion architecture is to refine data through multiple layers. Typically, this consist of a Bronze, Silver, and Gold layer.

### Bronze Layer
- The Bronze layer contains raw data **exactly as it arrived**.
- Examples:
	- Kafka events
	- JSON logs
	- CSV files
	- API responses
- **Very little** transformation occurs.
- The idea is simply **store** the raw data, not **fix** it.
- Raw data is kept because it makes data pipelines more resilient. Imagine a silent bug corrupting data in your ETL pipeline for two months.
	- If you keep the raw data, you can **replay** it after fixing the bug.
	- If you discard the raw data, you're out of luck. You need to re-ingest it, if possible.
- Bronze acts as the **source of truth** for a data warehouse.
- Bronze data usually contains:
	- Raw
	- Incomplete
	- Duplicate
	- Unvalidated
	- Immutable

### Silver Layer
- This is where Spark begins cleaning data.
- Typical operations include:
	- Remove duplicates
	- Standardize schemas
	- Validate records
	- Parse timestamps
	- Handle null values
	- **Join related datasets**
- Silver data is usually:
	- Validated
	- Cleaned
	- Standardized
	- Consistent
	- Trusted
- Most machine learning and downstream ETL begins here.

### Gold Layer
- This is where data is **optimized for business users**:
- Instead of raw events, the Gold layer creates:
	- Sales summaries
	- Customer KPIs
	- Daily revenue
	- Star schemas
	- Fact tables
	- Dimension tables
- This is the layer dashboards query.
- Instead of millions of transactions (rows), the Gold Layer typically contains **pre-aggregated** data. This makes queries very fast.
- **Visual Pipeline**:
	```
	Raw Kafka
	
	↓
	
	Bronze
	
	↓
	
	Spark Cleaning
	
	↓
	
	Silver
	
	↓
	
	Spark Aggregation
	
	↓
	
	Gold
	
	↓
	
	Power BI
	```
	- Spark performs different work at each stage.
- **Real Example**:
	- Suppose a customer's data arrives like this:
		```json
		{
		 "cust":"15",
		 "state":"va",
		 "purchase":"120"
		}
		```
	- Bronze: Store exactly as received.
	- Silver: Standardize:
		```json
		{
		 "customer_id":15,
		 "state":"VA",
		 "purchase":120
		}
		```
	- Gold: Aggregate for reporting:
		```
		Virginia
		
		↓
		
		Revenue
		
		↓
		
		$2.4M
		```

### Why Use Three Layers?
- Gold and Silver *could* be combined, but then Gold would need to perform:
	- Validation
	- Deduplication
	- Aggregation
- Silver is meant to separate data quality from business analytics. This is much cleaner and fault tolerant.
- Examples of data quality checks in Silver include:
	- Remove duplicates
	- Reject invalid dates
	- Validate IDs
	- Standardize formats
- Gold assumes the data is already trusted.
- **Typical Technologies**:
	- Bronze:
		- Kafka
		- S3
		- Delta Lake
	- Silver:
		- Spark
		- Delta Lake
	- Gold:
		- Spark
		- Snowflake
		- Power BI
		- Tableau
- **Change Data Capture**:
	- Bronze stores the raw event.
	- Silver updates the record using SCD Type 1 or 2.
	- Gold updates business metrics.

### Tradeoffs
- Bronze:
	- Pros:
		- Complete history
		- Easy replay
		- Maximum flexibility
	- Cons:
		- Poor quality
		- Not analyst-friendly
- Silver:
	- Pros:
		- Trusted
		- Standardized
		- Excellent foundation
	- Cons:
		- More storage
		- More ETL
- Gold:
	- Pros:
		- Fast dashboards
		- Business-friendly
		- Optimized queries
	- Cons:
		- Most derived
		- Least flexible
		- Requires maintenance

### Mental Model
- Connection to Previous Topics:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	Bronze (Delta Lake)
	
	↓
	
	Spark
	
	↓
	
	Silver
	
	↓
	
	Spark
	
	↓
	
	Gold
	
	↓
	
	Snowflake
	
	↓
	
	Power BI
	```

### Common Interview Questions
1. Why not let analysts query Bronze directly?
> 	The Bronze layer intentionally preserves raw source data, including duplicates, inconsistent schemas, and invalid records. Analysts generally need trusted, standardized data. By introducing a Silver layer for cleansing and validation and a Gold layer for business-specific modeling and aggregation, the architecture separates ingestion, data quality, and analytics concerns.
2. Your company receives clickstream events from Kafka. The events contain:
	- Duplicate records
	- Missing user IDs
	- Inconsistent timestamp formats
	- Extra debugging fields that analysts don't need
	- Walk me through how these events would flow through the Bronze, Silver, and Gold layers. What transformations would occur at each stage? Which layer would business analysts query?
> 	The raw data would be ingested and stored in the Bronze layer in its current format. The data would be cleaned, standardized, and validated in the Silver layer. The Silver layer would also deduplicate data and remove unnecessary debugging fields. Possible transformations include standardizing schemas, parsing timestamps, and handling null values. The data would be aggregated and summarized in the Gold layer. Analysts would typically query the Gold layer.
3. A teammate says: "Maintaining three copies of the data wastes storage. Let's eliminate the Silver layer and transform Bronze directly into Gold." Would you agree? What responsibilities would be lost? What operational or architectural problems could this create?
> 	Combining the Gold and Silver layers would increase complexity and maintenance overhead. The Silver layer is primarily meant to clean, standardize, and validate raw data, while the Gold layer is primarily meant to summarize data. Removing the Silver layer forces the Gold layer to handle both data quality and business modeling. That mixes two separate responsibilities into one pipeline, making it more complex, harder to test, and less reusable.
4. Could a company have more than three layers?
	- Yes. The Bronze/Silver/Gold terminology is a **logical model**, not a strict rule.
	- Some companies have Landing, Bronze, Silver, Gold, and Semantics layers.
	- Other companies have Raw, Validated, Curated, and Serving lays.
	- The important part isn't the names. It's the **separation of responsibilities**.

## Change Data Capture (CDC)

- CDC explains how a data warehouse learns about changes to the details of an entity in a database.
- **Option 1: Full Refresh**:
	- Every night, regardless of which rows were updated or added, the entire table is loaded into the data warehouse.
	- A better solution would be to simply capture the rows that have been added or changed. This is change data capture.
- CDC is a technique that captures:
	- Inserts
	- Updates
	- Deletes
- These changes are captured as they occur, instead of periodically copying entire tables. Much more efficient.
- **Types of Changes**:
	- `INSERT`: Add a new row.
	- `UPDATE`: Update a row.
	- `DELETE`: Remove a row.

### Capturing Changes
- Timstamp-based:
	- Suppose every table has an `updated_at` column.
	- Each ETL task runs: `WHERE updated_at > last_run`
	- Advantages:
		- Easy
		- Minimal infrastructure
	- Disadvantages
		- Missed timestamps
		- Clock synchronization
		- Deletes are difficult
- Log-based:
	- Modern systems often read a **database transaction log** instead of querying tables directly.
	- Examples:
		- MySQL binlog
		- PostgreSQL WAL
		- SQL Server transaction log
	- These logs contain every change.
	- No table scans required. Much more efficient.
- Debezium:
	- One of the most common CDC tools.
	- Architecture:
		```
		PostgreSQL
		
		↓
		
		Transaction Log
		
		↓
		
		Debezium
		
		↓
		
		Kafka
		
		↓
		
		Spark
		
		↓
		
		Warehouse
		```
		- Reads the WAL
		- Produces Kafka events
		- Spark processes these events
- CDC Event Example:
	```json
	{
	  "operation": "UPDATE",
	  "customer_id": 100,
	  "old_state": "Virginia",
	  "new_state": "Texas"
	}
	```
	- Applying CDC depends on the change logic being applied (SCD Type 1 or 2).
	- Type 1: Overwrite
	- Type 2: 
		- Mark old row as inactive.
		- Insert new row.
		- Generate surrogate key.
		- CDC and SCD work together. CDC **captures**. SCD **records**.
- Deletes:
	- Suppose an operational system deletes a customer. Should the warehouse delete the customer as well?
	- The answer depends on the business requirements.
	- Many warehouses implement a soft delete strategy by using an `is_deleted` column. This allows history to remain visible.
- Idempotency:
	- If a CDC event happens twice, it's not a bad thing if the **processing logic is idempotent**.
	- Spark `MERGE` operations are commonly used for this purpose.
- Ordering:
	- Ordering of changes is also crucial to maintaining an accurate history.
	- This is where Kafka partitioning strategies are especially importent. Partitioning needs to be designed in a way that forces related events to arrive in order.
- Data Pipeline Example:
	```
	Application
	
	↓
	
	PostgreSQL
	
	↓
	
	WAL
	
	↓
	
	Debezium
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Delta Lake
	
	↓
	
	Snowflake
	```

### Tradeoffs
- Full Refresh:
	- Pros:
		- Simple
		- Easy to reason about
	- Cons:
		- Slow
		- Expensive
		- Scans everything
- Change Data Capture:
	- Pros:
		- Incremental
		- Efficient
		- Near real-time
		- Less compute
	- Cons:
		- More infrastructure
		- More operational complexity
		- Ordering matters
		- Idempotency matters

### Mental Model
- Connection to Previous Topics:
	```
	Application
	
	↓
	
	PostgreSQL
	
	↓
	
	CDC
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Delta Lake
	
	↓
	
	Airflow
	
	↓
	
	Snowflake
	
	↓
	
	Gold Tables
	```

### Common Interview Questions
1. Why use CDC instead of nightly full refreshes?
> 	CDC captures only inserts, updates, and deletes rather than repeatedly scanning entire tables. This significantly reduces compute and network costs while enabling near real-time synchronization between operational systems and analytical platforms. Although CDC introduces additional operational complexity, it's generally much more efficient for large datasets where only a small percentage of records change.
2. Your operational database contains **800 million customer records**. Each day:
	- About **40,000 records are updated**.
	- About **5,000 new customers are added**.
	- A few hundred customers are deleted.
	- Would you recommend nightly full refreshes or CDC? Why? Walk me through how those changes would flow into your warehouse.
> 	I would recommend CDC because the amount of inserts, updates, and deletes is small compared to size of the database. Implementing CDC would involve either using an updated_at column in each table to capture changes, or using the database transaction log to capture changes. CDC events are streamed to the processing layer (for example, Kafka → Spark), where the appropriate business logic is applied before updating the warehouse. Changes that are captured are recorded using either SCD Type 1 or Type 2 logic, depending on the business requirements.
	- CDC doesn't save changes directly to the warehouse. Events are typically published to Kafka, consumed and processed by Spark, then loaded to the data warehouse.
3. A teammate says: "CDC is just a faster way to copy data." Would you agree? What additional engineering concerns does CDC introduce that don't exist with simple nightly batch refreshes?
> 	I would agree that CDC is a much faster way of recording changes, not necessarily copying data. They're not the same thing. CDC allows incremental changes in a database to be captured and loaded into a data warehouse, instead of performing a full table refresh. While CDC is faster, it introduces additional complexity and requires more infrastructure. For example, updates must be applied in the correct order to maintain accurate historical data. Additionally, the logic that processes changes must be idempotent just in case a CDC event is delivered more than once.
4. If only 10 rows change each day, should we always use CDC?
	- Not necessarily. It depends on the size of the table relative to those 10 rows. If the table only has 200 rows, a full refresh would be simpler and more cost-effective.
	- CDC introduces:
		- More infrastructure
		- More monitoring
		- More operational complexity
	- For very small tables, a full refresh can be the better engineering choice.

## Data Quality

- Suppose your pipeline finishes successfully. Spark didn't fail and Airflow shows green. The data was successfully loaded into Snowflake.
	- However, the dashboard shows: `Revenue: -$4,200,000`.
	- The pipeline worked, but the data was bad.
	- This is the key distinction between **pipeline reliability** and **data quality**. A successful pipeline doesn't necessarily produce trustworthy data.
- Data quality is about ensuring that data is:
	- Accurate
	- Complete
	- Consistent
	- Valid
	- Timely
	- Unique
- **Accuracy**:
	- Does the data correctly represent reality?
- **Completeness**:
	- Does all of the data arrive, or are certain components `NULL` or missing?
- **Consistency**:
	- Is the same data recorded in a consistent format?
	- For example, does the `state` column use the full name, two-letter abbreviation, some other variation, or does it vary with each entry?
	- The Silver layer of the Medallion architecture is meant to enforce consistency.
- **Validity**:
	- Is the data valid for a given context?
	- For example, is `Age` a negative value? Is `Year` a year that hasn't occurred yet, like `2098`?
	- Validation rules catch these errors.
- **Timeliness**:
	- Is the data stale or current?
	- For example, does a sales dashboard show the current day's sales or yesterday's sales?
- **Uniqueness**:
	- Does the pipeline allow duplicate values to exist?
	- Is ingestion logic idempotent?
	- Deduplication becomes useful.
- Most data quality checks occur in the **Silver Layer** of the Medallion architecture.

### Common Quality Checks
- Required columns: `customer_id != NULL`
- Range checks: `price > 0`
- Accepted values:
	```
	status
	
	IN
	(
	Completed,
	Pending,
	Cancelled
	)
	```
- Referential integrity: Every `customer_id` exists in the `DimCustomer` table.
- Duplicate detection: Every unique `order_id` appears once.

### Fail Fast vs. Quarantine
- Suppose 10 million records arrive, but only 500 are invalid. Should the entire pipeline fail?
	- Sometimes. Most of the time, no.
- **Fail Fast**:
	- Pros:
		- No bad data reaches analysts.
	- Cons:
		- One bad record can stop everything.
- **Quarantine**:
	- Valid records continue.
	- Invalid records are moved to a quarantine table.

### Data Quality Metrics
- Good pipelines **measure** in addition to validating.
- Examples:
	- Null rate
	- Duplicate rate
	- Freshness
	- Row count
	- Invalid record count
- **Great Expectations**:
	- Great Expectations is one framework used to enforce data quality rules.
	- Examples:
		- Expect `customer_id` never null.
		- Expect `price` between 0 and 100000
		- Expect `country` in `[US, CA, UK]`
- **Real Pipeline Example**:
	```
	Kafka
	
	↓
	
	Bronze
	
	↓
	
	Spark
	
	↓
	
	Silver
	
	↓
	Quality Checks
	
	↓
	
	Gold
	
	↓
	
	Dashboard
	```
	- Quality checks are built into the pipeline (Silver Layer). They're not added afterward.
	- Similar to the Beta and Gamma stages of a production code pipeline.

### Tradeoffs
- **Strict Validation**:
	- Pros: Very reliable
	- Cons: Pipelines fail more often
- **Lenient Validation**:
	- Pros: High availability
	- Cons: Bad data reaches analysts
- Many organizations use a hybrid approach:
	- Critical violations will fail the pipeline.
	- Minor violations quarentine data.

### Mental Model
- Connection to Previous Topics:
	```
	Kafka:
	
	Provides the events.
	
	↓
	
	Spark:
	
	Cleans them.
	
	↓
	
	Silver:
	
	Validates them.
	
	↓
	
	Gold:
	
	Publishes trusted data.
	
	↓
	
	Dashboards:
	
	Consume it.
	```
	- Data quality is what turns raw data into trusted data.

### Common Interview Questions
1. What data quality checks would you add to a pipeline?
> 	I'd start with schema validation, required field checks, range validation, duplicate detection, referential integrity, and row-count monitoring. I'd also define which violations should fail the pipeline versus which should quarantine records for later investigation, based on business impact.
2. Your daily pipeline processes **5 million orders**. This morning you discover:
	- 800 records have a `NULL customer_id`.
	- 25 records have a negative order amount.
	- 3 duplicate order IDs exist.
	- The total row count matches expectations.
	- Would you fail the pipeline or quarantine the bad records? Walk me through your reasoning. Which issues are critical? Which could safely continue?
> 	I would most like quarantine the bad records. In total, only about 850 records out of the 5 million daily records have failed quality checks. This is a minuscule amount of data and the symptoms don't appear to indicate a systemic issue with the pipeline. I would quarantine all records for further investigation and only configure the pipeline to fail when error counts rise above an established SLA. I'd evaluate each rule based on business impact. Some violations, such as duplicate financial transactions, might justify failing the pipeline even if only a handful of records are affected.
	- Each quality check should have its own SLA based on severity and potential business impact.
3. A teammate says: "If the Spark job succeeds, we know the data is good." Would you agree? What additional monitoring or validation would you implement to ensure the pipeline is producing trustworthy data rather than simply completing successfully?
> 	A Spark job succeeding does not unequivocally validate data. The Spark job could have absolutely no data quality checks and simply apply necessary transformations to the data. Data quality checks need to be implemented into the Spark job to ensure the quality of the data meets business requirements. In addition to transformation logic, I'd implement schema validation, null checks, duplicate detection, range validation, referential integrity checks, and freshness or row-count monitoring.
4. Yesterday the pipeline processed 5 million records. Today it processed 500 million records. Everything else looks valid.
	- Investigate before deciding. Check things such as:
		- A marketing campaign generated huge traffic.
		- A Kafka backlog was replayed.
		- A duplicate ingestion occurred.
		- A sudden 100× increase isn't automatically wrong, but it's unusual enough to warrant investigation.
		- Whether you fail the pipeline depends on the business impact and whether you can determine the cause quickly.

## Data Governance & Data Catalogs

- Data Quality answers "Can I trust this data?" Data Governance answers "Can I find it? Who owns it? Who can use it?"
- Imagine your company has:
	- 2,000 tables
	- 50 data engineers
	- 300 analysts
- An analyst asks, "Which customer table should I use?" They find:
	```
	customer
	
	customer_new
	
	customer_v2
	
	customer_backup
	
	customer_final
	
	customer_final_v2
	
	customer_gold
	
	customer_latest
	```
	- Which one is correct?
	- This is an example of a **governance** problem.
- **Data governance** is the collection of policies, processes, and responsibilities that ensure data is:
	- Discoverable
	- Trusted
	- Secure
	- Well documented
	- Properly owned
	- Used consistently
- Governance is a process, not a technology.
- A **data catalog** is like a search engine for your company's data. Instead of searching the web, you search:
	- Tables
	- Columns
	- Dashboards
	- Pipelines
- For example, searching `customer` in the data catalog could yield results such as:
	```
	DimCustomer
	
	Owner:
	Marketing
	
	Description:
	Master customer dimension
	
	Quality:
	99.98%
	
	Last Updated:
	Today
	```
	- This is much more useful than simply seeing a table name.

### Governance and Catalog Concepts
- **Metadata**:
	- A catalog only stores metadata, not the data itself.
	- Examples of metadata include:
		- Table descriptions
		- Column descriptions
		- Owners
		- Update frequency
		- Tags
		- Lineage
		- Quality metrics
- **Data Ownership**:
	- Every important dataset should have clear ownership.
	- When something breaks, this tells everyone who is responsible for investigating the issue.
- **Business Glossary**:
	- Metrics such as revenue may have different definitions for different teams in the same organization.
	- A business glossary clearly defines exactly what each metric means.
- **Data Classification**:
	- Not all data is equally sensitive.
	- Levels of classification include:
		- Public: `Product Name`
		- Internal: `Inventory Count`
		- Confidential: `Employee Salary`
		- Restricted: `Social Security Number`
	- Classifications determine:
		- Who can access the data
		- How the data can be used
		- Encryption requirements
		- Retention policies
- **Access Controls**:
	- Suppose an intern runs: `SELECT * FROM payroll;`
		- This query should fail to execute because an intern should not have access to broad payroll data.
		- Data governance decides who can access a dataset.
		- The **Principle of Least Privilege** is typically applied to data governance policies.
- **Data Stewardship**:
	- A **data steward** is responsible for data quality and documentation.
	- Sometimes the data owner is also the data steward.
	- Sometimes the steward is a different person.
- **Documentation**:
	- Good governance includes documentation such as column descriptions.
	- For example:
		- `FactSales`: Contains one row per completed customer order.
		- `order_date`: Date order completed, UTC.
	- This saves countless hours for data analysts.
- **Popular Catalog Tools**:
	- Microsoft Purview
	- AWS Glue Data Catalog
	- Google Dataplex
	- DataHub
	- Apache Atlas
	- Collibra
	- Alation
- **Real Pipeline Example**:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Delta Lake
	
	↓
	
	Snowflake
	
	↓
	
	Data Catalog
	
	↓
	
	Analysts
	```
	- The catalog doesn't move data. It helps people **discover and understand** it.

### Tradeoffs
- Strong Governance:
	- Pros:
		- Trusted data
		- Easy discovery
		- Better security
		- Clear ownership
	- Cons:
		- More documentation
		- Additional operational overhead
		- Requires organizational discipline
- Weak Governance:
	- Pros:
		- Less upfront effort
	- Cons:
		- Duplicate datasets
		- Conflicting metrics
		- Confusion
		- Security risks

### Mental Model
- Connection to Previous Topics:
	```
	Kafka
	
	↓
	
	Bronze
	
	↓
	
	Silver
	
	↓
	
	Gold
	
	↓
	
	Snowflake
	```
	- Imagine 100s of tables in the gold layer.
	- Without governance, analysts spend more time **finding** the correct data than **analyzing** it.
	- Governance turns a collection of tables into a useable data platform.

### Common Interview Questions
1. Why is a data catalog valuable?
> 	A data catalog centralizes metadata about an organization's datasets, making them easier to discover, understand, and trust. It documents ownership, schema, update frequency, lineage, and quality information so analysts and engineers can confidently identify the correct datasets and understand how they should be used.
	- **discoverability and trust** are two of the most important features that a data catalog provides.
2. Your company has:
	- 3,500 tables
	- Multiple data engineering teams
	- Analysts frequently build duplicate datasets because they can't find existing ones.
	- Two dashboards regularly report different values for the same KPI.
	- What governance improvements would you recommend? How would a data catalog help? What metadata would you prioritize documenting?
> 	I would recommend building a robust data catalog that clearly defines what tables exist and what each column represents. The data catalog should also document the owner of the table, it's most recent quality status, and when it was last updated. This metadata would allow analysts to collaborate more efficiently so they don't build duplicate datasets and dashboards use appropriate metrics for their business context. I'd also establish a business glossary that defines key metrics such as Revenue, Active Customer, and Conversion Rate so every dashboard uses consistent definitions.
3. A teammate says: "Documentation is a waste of time. Engineers can just look at the table schema." Would you agree? What important information is **not** captured by a schema? Can you think of situations where good documentation would prevent costly mistakes?
> 	A table schema, while useful, only tells you what columns appear in a table, their associated data types, and which columns are primary or foreign keys. The schema does not list a table's owner, data quality, recency, or column descriptions. Good documentation helps prevent incorrect data from being used to create business-critical dashboards.
4. If a data catalog already contains table descriptions, why do we need data owners?
> 	Documentation doesn't maintain itself. Data owners are accountable for keeping datasets accurate, documenting changes, responding to quality issues, and approving schema modifications. Without clear ownership, problems can go unresolved because nobody is responsible for the dataset.
	- Ownership is about **accountability**, not just information.

## Data Lineage

- Governance answers:
	- Who owns this dataset?
	- What does it mean?
- Lineage answers:
	- Where did come from?
	- Who depends on it?
- Suppose an executive notices the following:
	```
	Revenue
	
	Yesterday
	
	↓
	
	$12.4M
	
	↓
	
	Today
	
	↓
	
	$9.8M
	```
	- No one knows why there was a significant drop in revenue.
	- Where do you start investigating?
	- Without **data lineage**, you need to manually inspect:
		- Airflow DAGS
		- Spark jobs
		- Kafka topics
		- SQL transformations
		- Gold tables
	- This could take hours and is inefficient.
- Data lineage is the ability to **trace** data:
	- Where it originated
	- How it was transformed
	- Where it was stored
	- Who consumes it
- Suppose an analyst queries `Gold.RevenueSummary`. The lineage might show:
	```
	Gold.RevenueSummary
	
	↑
	
	Silver.Sales
	
	↑
	
	Bronze.KafkaOrders
	
	↑
	
	Kafka Topic
	
	↑
	
	Order Service
	```
	- Now you immediately know where the data came from.
	- This allows you to identify the source of data quality issues more quickly.
- Suppose someone wants to modify the `Silver.Sales`:
	```
	Silver.Sales
	
	↓
	
	Gold.Revenue
	
	↓
	
	Executive Dashboard
	
	↓
	
	Finance Dashboard
	
	↓
	
	Forecast Model
	```
	- With lineage, they understand:
		- Which dashboards or tables will be affected.
		- Other potential downstream impacts.
		- This is called **impact analysys**.
- Column-level lineage allows you to track individual columns, not just tables.
- **End-to-End Example**:
	```
	Orders API
	
	↓
	
	Kafka
	
	↓
	
	Bronze
	
	↓
	
	Spark Cleaning
	
	↓
	
	Silver
	
	↓
	
	Spark Aggregation
	
	↓
	
	Gold
	
	↓
	
	Snowflake
	
	↓
	
	Power BI Dashboard
	```
	- Lineage can trace any value all the way back to the original source.

### Common Use Cases
- Debugging:
	- Revenue looks incorrect.
	- Trace backwards to the source.
- Change Management:
	- Schema Change Planned.
	- Trace forward to assess potential impact.
- Compliance:
	- A regulator asks, "Where does this report get its data?"
- Data Quality:
	- A quality check fails.
	- Which downstream datasets are affected?
	- Data lineage tells you.

### Popular Lineage Tools
- Examples include:
	- OpenLineage
	- DataHub
	- Apache Atlas
	- Microsoft Purview
	- Collibra

### Tradeoffs
- Good Lineage:
	- Pros:
		- Faster debugging
		- Easier impact analysis
		- Better compliance
		- Improved trust
	- Cons:
		- Requires metadata collection
		- Additional operational effort
		- Needs to stay current
- No Lineage:
	- Pros:
		- Less setup
	- Cons:
		- Slow investigations
		- Risky schema changes
		- Poor visibility
		- Difficult audits

### Mental Model
- Connection to Previous Topics:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	Bronze
	
	↓
	
	Silver
	
	↓
	
	Gold
	
	↓
	
	Snowflake
	
	↓
	
	Dashboard
	```
	- Lineage connects all of those components into a traceable graph.
	- When something goes wrong, you're no longer guessing where the problem started.

### Common Interview Questions
1. Why is data lineage important?
> 	Data lineage provides visibility into how data flows through a platform, from its original source through every transformation to the final datasets and dashboards. It helps engineers perform root cause analysis, assess the impact of schema changes, support compliance requirements, and build trust in analytical data.
	- Since lineage shows how data travels from source to dashboard, it can be used for root cause analysis and impact (downstream) analysis that is required before implementing changes.
2. An executive reports that yesterday's revenue dashboard is incorrect. Walk me through how you would use **data lineage** to investigate the issue. What stages would you examine? How does lineage reduce the time required for root cause analysis?
> 	I would use the data lineage graph to trace backwards from the dashboard, through the Gold, Silver, and Bronze layers of transformation, to assess where the data was corrupted. Data lineage helps make this task more efficient because it provides a clear roadmap of how data moves from source to target. This eliminates the need for manual investigation of where corrupt data was sourced before actually identifying the root cause.
3. A teammate wants to rename a column in your **Silver Sales** table. Would you make the change immediately? How would lineage help you evaluate the impact of that change? What downstream systems might be affected?
> 	Before renaming the column in the Silver Sales table, I would use the lineage graph to assess which downstream dependencies could potentially be affected by the change, such as the Gold Sales table, machine learning models, and analytical dashboards used for reporting. I would work with the owners of these dependencies to identify and mitigate any potential risks befor beginning my implementation.
	- The Gold table in a medallion architecture can be used to feed machine learning models, not just executive dashboards.
4. If lineage shows that a Gold table depends on ten upstream tables, does that mean all ten are equally important?
	- Not necessarily. Some may be critical inputs, such as the `Orders` table, while others may provide optional enrichment, such as the `Customer Marketing Segment` table.
	- Lineage only shows **dependencies**. You still need to understand the **business logic** to assess its impact.

## Security & Privacy

- Suppose your company stores:
	- Customer names
	- Email addresses
	- Credit card tokens
	- Social Security Numbers
	- Purchase history
- Should every engineer have access to every table? Hopefully not.
- The **Principle of Least Privilege** states that users should have only the minimum permissions necessary to perform their job.
- Example:
	- The Marketing team may only need access to the following information:
		- Customer State
		- Age Group
		- Purchase History
	- They probably don't need access to:
		- Social Security Number
		- Payroll Data
		- Credit Card Tokens
- **Role-Based Access Control (RBAC)**:
	- Instead of granting permissions to **individuals**, permissions are granted to roles.
	- Example Data Engineer Role Permissions:
		```
		Data Engineer
		
		↓
		
		Read Bronze
		
		Read Silver
		
		Write Gold
		```
	- Example Business Analyst Role Permissions:
		```
		Business Analyst
		
		↓
		
		Read Gold Only
		```
	- Now, adding a new analyst simply means assigning them to the Business Analyst Role. No custom or special configuration required.
- **Identity and Access Management (IAM)**:
	- IAM Answers:
		- Who are you?
		- What are you allowed to do?
	- Examples:
		- AWS IAM
		- Azure ID
		- Google Cloud IAM
	- IAM controls access to:
		- S3 Bucks
		- Snowflake
		- Kafka
		- Airflow
		- Databases
	- IAM acts as the gatekeeper for your platform.

### Encryption
- **Encryption At Rest**:
	- Data stored on a disk is encrypted.
	- Examples:
		- S3 objects
		- Snowflake tables
		- Delta lake files
	- If someone steals the storage media, they still can't read the data without the encryption keys.
- **Encryption in Transit**:
	- Data moving across the network is encrypted.
	- Examples:
		```
		Application
		
		↓
		
		HTTPS / TLS
		
		↓
		
		Kafka
		
		↓
		
		TLS
		
		↓
		
		Spark
		
		↓
		
		TLS
		
		↓
		
		Snowflake
		```
		- Every time the data moves from one source to another, it is encrypted using Transport-Layer Security (TLS).
		- This protects against network eavesdropping.
- **Personally Identifiable Information (PII)**:
	- PII is information that can identify an individual.
	- Examples:
		- Social Security Number
		- Driver's License Number
		- Passport Number
		- Email Address
		- Phone Number
		- Home Address
	- Not every dataset should contain PII.
- **Data Masking**:
	- Suppose an analyst needs to differentiate between two customers.
	- Instead of being given access to each customer's full SSN to make the distinction, they're given access to a version that redacts the sensitive portion of the SSN: `XXX-XX-1234`.
	- Combined with other non-sensitive information, the masked SSN should be enough to uniquely identify the customer.
	- This is called data masking.
- **Tokenization**:
	- Instead of storing a credit card number as `4111-1111-1111-1111`, it is stored as `A91B72XZ`.
	- The token has no meaning by itself.
	- Unlike encryption, tokenization **replaces** the sensitive value entirely.
- **Secrets Management**:
	- Suppose your Spark job needs Snowflake credentials. Should these credentials be hard-coded? Absolutely not.
	- Instead a secrets manager is used. This acts as a secure storage layer that applications and programs can use to access sensitive information.
	- Examples:
		- AWS Secrets Manager
		- Azure Key Vault
		- HashiCorp Vault
- **Auditing**:
	- Audit logs describe who accessed sensitive data, when they accessed it, and why they accessed it.
	- Audit logs are essential for:
		- Compliance
		- Incident investigation
		- Security monitoring
- **Data Retention**:
	- Data shouldn't necessarily be stored forever.
	- Governance often defines retention policies.
	- For example:
		- Application logs can be deleted after 90 days.
		- Financial transactions can be deleted after 7 years.
	- Retention depends on legal and business requirements.
- Security is applied throughout all components of a data pipeline. It's not a single feature, it's a cross-cutting concern.

### Tradeoffs
- Very Restrictive Security:
	- Pros:
		- Lower risk
		- Better compliance
	- Cons:
		- Slower development
		- More access requests
		- Potential productivity impact
- Very Open Security:
	- Pros:
		- Easy collaboration
	- Cons:
		- Data leaks
		- Compliance violations
		- Greater insider risk
- The goal is to strike the right balance based on **business and regulatory** requirements.

### Mental Model
- Connection to Previous Topics:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	Bronze
	
	↓
	
	Silver
	
	↓
	
	Gold
	
	↓
	
	Snowflake
	
	↓
	
	Dashboard
	```
	- Security exists at **every stage**, not just the database.

### Common Interview Questions
1. How would you secure a data pipeline?
> 	I'd apply the principle of least privilege using role-based access control, encrypt data both in transit and at rest, store secrets in a dedicated secrets manager rather than in code, protect sensitive fields through masking or tokenization where appropriate, and enable audit logging so access to sensitive data can be monitored and investigated.
2. Your company stores customer purchase history in Snowflake. Analysts need to analyze purchasing behavior, but they do **not** need access to customers' Social Security Numbers or full credit card information. How would you design access to this data? Which security techniques would you use? How would you balance usability with privacy?
> 	I would design access using the Principle of Least Privilege, ensuring analysts are only permitted access to the exact data they need to perform their duties. I'd implement this using Role-Based Access Control. I would also ensure data is encrpyted at rest and in transit, ensure secrets are accessed using a secure secrets manager instead of being hard coded, mask or tokenize sensitive fields, and enable audit logging to monitor and investigate access to secure information.
3. A teammate says: "Our S3 bucket is encrypted, so we don't need to worry about IAM permissions or secrets management." Would you agree? Why or why not? How do encryption, IAM, RBAC, and secrets management complement one another rather than replace each other?
> 	I would not agree. Even if an S3 bucket is encrypted, an attacker with excessive permissions could still read the data. Encryption protects confidentiality, while IAM and RBAC control access, and secrets management protects the credentials used to obtain that access. Encryption, IAM, RBAC, and secrets management complement one another by ensuring data is secured at and in between every phase of transformation in a data pipeline. These features also ensure analysts only have access to the data they need to make meaningful business insights.
	- S3 supports encryption at rest using SSE-KMS, while secure access to S3 over HTTPS supports encryption in transit using TLS.
	- Encryption is only one part of the battle. RBAC and secrets management are still needed to keep data secure.
4. If analysts only need aggregated sales data, should they even have access to the Bronze layer?
	- Usually, no. Bronze contains raw data, duplicates, invalid record, and potential PII.
	- Analysts typically only need access to Gold tables.