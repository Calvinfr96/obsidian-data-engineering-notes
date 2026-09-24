---
aliases:
  - "databricks_essentials"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Spark and DataBricks/databricks_essentials.md"
imported: 2026-09-24
---

# Delta Lake Labs

Review: [[02_technical_prep/Databricks Review|Databricks Review]] · [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]]

## Contents

- [[#Creating Clusters and Notebooks]]
- [[#Schema Evolution in Delta Lake]]
- [[#Deletion Vectors in Delta Lake]]
- [[#Delta Cloning]]
- [[#Delta Change Data Feed (CDF)]]
- [[#Delta Optimization – Partitioning]]
- [[#Delta Optimizations – Bloom Filter]]
- [[#Delta Optimization – Z-Ordering]]
- [[#Delta Lake Liquid Clustering]]
- [[#Delta Lake Hands-On Demo]]
- [[#Time Travel in Delta Lake]]
- [[#Delta Optimization – Default Small Files]]

## Creating Clusters and Notebooks
- Creating a Cluster:
  - Clusters are created in the data bricks UI under 'Compute'. When creating a cluster, you specify the name, policy (access control), whether the cluster is for machine learning, data bricks run time (best to select a long-term support (LTS) version of Scala and Spark), and worker type (GB of RAM and Number of cores).
  - When selecting a worker type, you can also specify the minimum and maximum number of worker nodes. This is used by data bricks to autoscale the cluster for workloads of varying intensity. If you select 'single node' instead, no autoscaling will take place.
  - The 'Terminate After' option specifies the period of inactivity after which data bricks will automatically terminate the cluster. This is important for cost-saving purposes.
  - Once a compute is created, you are given access to the Spark UI, which can give you insights into the performance of jobs and queries that are run on that compute cluster.
  - In the free edition / free trial, you are given access to a basic, serverless compute cluster, but cannot create your own cluster.
- Creating a Notebook:
  - Notebooks are created in the data bricks UI under 'Workspace'. These notebooks use the same file structure as a Jupyter notebook.
  - Once a notebook is created, you can connect it to a cluster of your choosing to run the code that write in the notebook.
  - When using a notebook, the `%` is used in the first line of code to either specify the language (i.e. `%sql`) or which notebook to run (i.e. `%run ./test_notebook`).
## Schema Evolution in Delta Lake
```python
# Databricks notebook source
spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.optimizeWrite", False)
spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.autoCompact", False)

# COMMAND ----------

# MAGIC %sql
# MAGIC use s3catalog.default;

# COMMAND ----------

dbutils.fs.rm('s3://testdatabricks1992/delta_schema_enforcement', True)

# COMMAND ----------

# MAGIC %sql
# MAGIC DROP TABLE IF EXISTS delta_schema_enforcement;
# MAGIC
# MAGIC CREATE OR REPLACE TABLE delta_schema_enforcement (id INT, valid BOOLEAN NOT NULL, some_text STRING)
# MAGIC using delta 
# MAGIC LOCATION 's3://testdatabricks1992/delta_schema_enforcement';
# MAGIC
# MAGIC insert into delta_schema_enforcement values (1, true, "ok")

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement;

# COMMAND ----------

# MAGIC %md
# MAGIC check if the schma is enforced

# COMMAND ----------

# MAGIC %sql
# MAGIC desc delta_schema_enforcement;

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into delta_schema_enforcement values (1, true, "ok", "this is a test") # Tries to insert four values into a row even though there are only three columns. Doesn't work due to schema mismatch.

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into delta_schema_enforcement values ("two", true, "ok") # The schema mismatch here is caused by mismatched data types. String cannot be cast to an int.

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into delta_schema_enforcement values ("2", true, "ok") # This insert works because "2" can be converted to an int.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement;

# COMMAND ----------

# MAGIC %md
# MAGIC Enable schem evaluation
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC desc delta_schema_enforcement

# COMMAND ----------

data = [(3, True, "ok", "not ok")]
df = spark.createDataFrame(data, "id int, valid boolean, some_text string, others string")
df.display()

# COMMAND ----------

df.write.format("delta").option("mergeSchema","true").mode("append").saveAsTable("delta_schema_enforcement") # Overrides the schema in the existing table and appends the data in the data frame to the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc delta_schema_enforcement # Shows that the schema of the table was updated from the above write command.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history delta_schema_enforcement # Describes history of changes to the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement;

# COMMAND ----------

data = [("1", True, "not ok")]
df = spark.createDataFrame(data, "id string, valid boolean, others string")
df.display()

# COMMAND ----------

df.write.format("delta").option("mergeSchema","true").mode("append").saveAsTable("delta_schema_enforcement") # The update to the schema cannot be performed because string cannot be converted to int.

# COMMAND ----------

data = [(4, True, "not ok")]
df = spark.createDataFrame(data, "id int, valid boolean, others string")
df.display()

# COMMAND ----------

df.write.format("delta").option("mergeSchema","true").mode("append").saveAsTable("delta_schema_enforcement") # The update to the schema succeeds. The fourth column isn't removed, it's just populated with null.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

# MAGIC %md
# MAGIC SQL schema evaluation
# MAGIC

# COMMAND ----------

spark.conf.set("spark.databricks.delta.schema.autoMerge.enabled", "true") # Allows the schema to be automatically updated, without needing to explicitly set the option as in the examples above.

# COMMAND ----------

# MAGIC %sql
# MAGIC Insert into delta_schema_enforcement values (11, true, "ok", "others", "extra");
# MAGIC select * from delta_schema_enforcement; # Schema is successfully updated automatically.

# COMMAND ----------

# MAGIC %md
# MAGIC Overwrite - Careful when you use them

# COMMAND ----------

data = [(1, True, "ok", "not ok")]
df = spark.createDataFrame(data, "id int, valid boolean, name string, others string")
df.display()

df.write.format("delta").option("overwriteSchema","true").mode("overwrite").saveAsTable("delta_schema_enforcement") # Overwrites the entire schema instead of updating it.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------
```
## Deletion Vectors in Delta Lake
```python
# Databricks notebook source
spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.optimizeWrite", False)
spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.autoCompact", False)

# COMMAND ----------

spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.optimizeWrite", False)
spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.autoCompact", False)

# COMMAND ----------

dbutils.fs.rm('s3://testdatabricks1992/delta_schema_enforcement', True)

# COMMAND ----------

# MAGIC %sql
# MAGIC use s3catalog.default;

# COMMAND ----------

# MAGIC %sql
# MAGIC DROP TABLE IF EXISTS delta_schema_enforcement;
# MAGIC
# MAGIC CREATE OR REPLACE TABLE delta_schema_enforcement (id int, name STRING)
# MAGIC using delta 
# MAGIC LOCATION 's3://testdatabricks1992/delta_schema_enforcement'; # Creates a parquet file for the new table.
# MAGIC
# MAGIC insert into delta_schema_enforcement values (1, "alis"),(2, "bob"),(3, "clare"); # Creates a parquet file for the newly updated table.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC delete from delta_schema_enforcement
# MAGIC where name = 'alis'; # Creates a delete_vector bin file.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history delta_schema_enforcement # Shows that there was a delete operation, as well as an optimize operation.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

#show the delta table 
# explain the data file and show the delta log files

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

#show the delta table 
# explain the data file and show the delta log files
# expand and show the delta logs with the deletion vectors 

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement # Removes stale data files (older than the default 7 days).

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement dry run # Shows files that will be removed (older than 7 days).

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours dry run # Shows all files that will be removed.

# COMMAND ----------

spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled","false") # The above command fails without updating this setting, even if it is just a dry run.

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into delta_schema_enforcement values (11, "Daisy");
# MAGIC delete from delta_schema_enforcement where name = 'Daisy' # No optimization needed to be performed after this query.

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours dry run

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement version as of 4
# MAGIC

# COMMAND ----------
```
- When a delete operation is run, it creates a `bin` file with the prefix `delete_vector`, instead of a `parquet` file.
- A JSON file is created in the transaction log for the table creation, update, delete, and query optimization.
- Vacuuming affects your ability to time travel, especially when you use a retention period of 0 days. **When working in production environments, it's best to stick with at least the default retention period of 7 days.**
## Delta Cloning
```python
# Databricks notebook source
# MAGIC %md
# MAGIC #### Shallow clone

# COMMAND ----------

spark.conf.set("spark.databricks.delta.autoCompact.enabled", "false")
spark.conf.set("spark.databricks.delta.optimizeWrite.enabled", "false")

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC drop table workspace.my_new_schema.customer_src;
# MAGIC -- drop table workspace.my_new_schema.customer_shallow_clone;
# MAGIC -- drop table workspace.my_new_schema.customer_deep_clone;

# COMMAND ----------

# MAGIC %sql
# MAGIC use workspace.my_new_schema # Uses a brand new schema.

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC create or replace table customer_src as (select * from samples.tpch.customer where c_custkey between 1 and 5) # Inserts 5 records into the newly created table.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC desc detail customer_src

# COMMAND ----------

# MAGIC %md
# MAGIC second update

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into customer_src select * from samples.tpch.customer where c_custkey between 6 and 10

# COMMAND ----------

# MAGIC %md
# MAGIC third update

# COMMAND ----------

# MAGIC %sql
# MAGIC delete from customer_src 
# MAGIC where c_custkey = 2

# COMMAND ----------

# MAGIC %sql
# MAGIC update customer_src
# MAGIC set c_address = 'new address'
# MAGIC where c_custkey = 5

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history customer_src; # Shows the creation of the table and all of the updates made.

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC Create or replace table customer_shallow_clone shallow clone customer_src # Creates a shallow clone of the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC describe detail customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC describe detail customer_shallow_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC desc history customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC desc history customer_shallow_clone # Shows the history of the cloned table. Will only show one entry, which is associated with the clone operation.

# COMMAND ----------

# MAGIC %sql
# MAGIC SET spark.databricks.delta.retentionDurationCheck.enabled = false;
# MAGIC VACUUM customer_src RETAIN 0 HOURS; # Removes stale files after performing clone.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_shallow_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC insert into customer_src 
# MAGIC select * from samples.tpch.customer where c_custkey = 100

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src # The source table is updated with the new record.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from  customer_shallow_clone # The cloned table is not updated with the new record.

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC delete from  customer_src where c_custkey between 1 and 5

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from  customer_shallow_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC SET spark.databricks.delta.retentionDurationCheck.enabled = false;
# MAGIC VACUUM customer_src RETAIN 0 HOURS; # The vacuum won't remove the parquet files associated with the records deleted in the source table because they're still referenced by the cloned table.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from  customer_shallow_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC delete from customer_src where c_custkey = 1

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history customer_src

# COMMAND ----------

# MAGIC %md
# MAGIC change in cloned table 

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into customer_shallow_clone values (9999,	'Customer#000000010',	'6LrEaV6KR6PLVcgl2ArL  Q3rqzLzcT1 v2',	5	,'15-741-346-9870',	'2753.54',	'HOUSEHOLD',	'es regular deposits haggle. fur')

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_shallow_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------


# COMMAND ----------

# MAGIC %md
# MAGIC clone of a clone

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC select * from customer_src timestamp as of '2025-09-04T19:43:12.000+00:00'

# COMMAND ----------

# MAGIC %sql
# MAGIC create or replace table customer_shallow_clone_v8 shallow clone customer_src timestamp as of '2025-09-04T19:50:31.000+00:00'

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_shallow_clone_v8

# COMMAND ----------

# MAGIC %md
# MAGIC #### Deep clone

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC CREATE or replace table customer_deep_clone deep clone customer_src # Clones metadata and table data.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_deep_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail customer_src

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail customer_deep_clone

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC insert into customer_deep_clone select * from samples.tpch.customer where c_custkey = 1000; # Has no effect on source table, just like a shallow clone.

# COMMAND ----------
```
- Shallow clones only copy the metadata associated with a table. The parquet (data) files are shared between the cloned table and original table.
- A deep clone copies the entire table (metadata and actual data) into a new location. Both tables become independent.
- Performing a shallow clone affects vacuuming behavior. If you delete records from the source table, but not the cloned table, the records associated with those deleted records will be retained because they're still referenced by the cloned table.
- You cannot create a shallow clone of a table that is itself a shallow clone.
## Delta Change Data Feed (CDF)
```python
# Databricks notebook source
# MAGIC %md
# MAGIC Change Data Feed
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC set spark.databricks.delta.properties.default.enablechangedatafeed = true # All tables created after updating this setting will have CDF enabled.

# COMMAND ----------

# MAGIC %sql
# MAGIC use workspace.my_new_schema # Sets schema that will be used.

# COMMAND ----------

from pyspark.sql.functions import *

# COMMAND ----------

source_df = spark.table('samples.tpch.customer').filter(col('c_custkey').between(100, 105) )

source_df.write.mode('overwrite').saveAsTable('cdf_customer_table') # Creates a table using the data frame.

# COMMAND ----------


source_df.write.mode('overwrite').saveAsTable('goldTable') # Creates a table using the data frame.

# COMMAND ----------

# MAGIC %sql
# MAGIC DESCRIBE HISTORY cdf_customer_table

# COMMAND ----------

# MAGIC %sql
# MAGIC ALTER TABLE cdf_customer_table SET TBLPROPERTIES (delta.enableChangeDataFeed = true) # Enables CDF for the current version of specified table.
# MAGIC     

# COMMAND ----------

# MAGIC %sql
# MAGIC UPDATE cdf_customer_table SET c_address = 'newly updated address' WHERE c_custkey = '102'

# COMMAND ----------

# MAGIC %sql
# MAGIC UPDATE cdf_customer_table SET c_name = 'new name' WHERE c_custkey = '103'

# COMMAND ----------

# MAGIC %sql
# MAGIC DELETE from cdf_customer_table WHERE c_custkey = '105'
# MAGIC

# COMMAND ----------

(spark.table('samples.tpch.customer').filter(col('c_custkey').between(1010, 1015) )
.write.mode('append').saveAsTable('cdf_customer_table')) # Adds records from the customer table to the cdf_customer_table.

# COMMAND ----------

# MAGIC %md
# MAGIC Reading the CDF in sql
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC describe history cdf_customer_table

# COMMAND ----------

# MAGIC %sql
# MAGIC SELECT c_custkey, c_address, _change_type, _commit_timestamp, _commit_version FROM table_changes('cdf_customer_table', 2) # Selects the specified columns from version 2 of the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC SELECT c_custkey, c_address, _change_type, _commit_timestamp, _commit_version FROM table_changes('cdf_customer_table', 2, 4) # Selects the specified columns from versions 2 through 4 of the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC SELECT * FROM table_changes('cdf_customer_table', '2025-09-06T15:56:36') # Reading the same table by specifying a timestamp instead of version number(s). 

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC SELECT * FROM table_changes('cdf_customer_table', '2025-09-06T15:56:36', '2025-09-06T15:57:04') # Reads table changes between two timestamps.

# COMMAND ----------

# MAGIC %md
# MAGIC Reading in python api
# MAGIC

# COMMAND ----------

# version as ints or longs
(spark.read \
  .option("readChangeFeed", "true") \
  .option("startingVersion", 1) \
  .option("endingVersion", 4) \
  .table("cdf_customer_table")).display()

# COMMAND ----------

(spark.read \
  .option("readChangeFeed", "true") \
  .option("startingVersion", 2) \
  .table("cdf_customer_table")).display()

# COMMAND ----------

# timestamps as formatted timestamp
(spark.read \
  .option("readChangeFeed", "true") \
  .option("startingTimestamp", '2025-09-06T15:56:36') \
  .option("endingTimestamp",  '2025-09-06T15:57:04') \
  .table("cdf_customer_table")).display()

# COMMAND ----------

# MAGIC %md
# MAGIC ETL example

# COMMAND ----------

# MAGIC %md
# MAGIC # moving the changes in silver table to gold table

# COMMAND ----------

# MAGIC
# MAGIC %sql
# MAGIC CREATE OR REPLACE TEMPORARY VIEW silverTable_latest_version as
# MAGIC SELECT * 
# MAGIC     FROM 
# MAGIC          (SELECT *, rank() over (partition by c_custkey order by _commit_version desc) as rank
# MAGIC           FROM table_changes('cdf_customer_table', 2)
# MAGIC           WHERE _change_type !='update_preimage')
# MAGIC     WHERE rank=1 # Selects the latest version of each customer.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from silverTable_latest_version

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from goldTable

# COMMAND ----------

# MAGIC %sql
# MAGIC -- Merge the changes to gold
# MAGIC MERGE INTO goldTable t USING silverTable_latest_version s 
# MAGIC     ON s.c_custkey = t.c_custkey
# MAGIC     WHEN MATCHED AND s._change_type='update_postimage' THEN 
# MAGIC         UPDATE SET 
# MAGIC             t.c_name = s.c_name, t.c_address = s.c_address
# MAGIC     WHEN NOT MATCHED THEN 
# MAGIC         INSERT (t.c_custkey,t.c_name,t.c_address,t.c_nationkey,t.c_phone,t.c_acctbal,t.c_mktsegment,t.c_comment) VALUES (s.c_custkey,s.c_name,s.c_address,s.c_nationkey,s.c_phone,s.c_acctbal,s.c_mktsegment,s.c_comment)
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from goldTable

# COMMAND ----------
```
- A CDF is a feature that allows data bricks to track row-level changes on a table.
## Delta Optimization – Partitioning
```python
# Databricks notebook source
# MAGIC %md
# MAGIC - Benefits of partitioning over data skipping are not always so obvious
# MAGIC - if usage pattern changes, partitioning needs to be updated
# MAGIC - to change the partitions, table must ne rewritten
# MAGIC - High chance of data skewness
# MAGIC - possible problem with large amount of files
# MAGIC
# MAGIC
# MAGIC - Databricks recommends not to partition the table that are less that 1 TB in size and when done that table should have at least 1 gb of data per partitions
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC use my_new_schema

# COMMAND ----------

spark.conf.set('spark.databricks.io.cache.enable', False)

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from csv.`dbfs:/databricks-datasets/asa/airlines/2007.csv`

# COMMAND ----------

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/{2007, 2008}.csv', header=True, inferSchema=True) \
    .write \
    .format('delta') \
    .mode('overwrite') \
    .saveAsTable('airlines_no_partition_2') # Table created with no partitions.

# COMMAND ----------

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/{2007, 2008}.csv', header=True, inferSchema=True) \
    .write \
    .format('delta') \
    .mode('overwrite') \
    .partitionBy('Year') \
    .saveAsTable('airlines_year_2') # Creates a table partitioned by year.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail airlines_no_partition_2

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail airlines_year_2

# COMMAND ----------

# MAGIC %md
# MAGIC multipartitions

# COMMAND ----------

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/{2007, 2008}.csv', header=True, inferSchema=True) \
    .write \
    .format('delta') \
    .mode('overwrite') \
    .partitionBy('Year','Origin') \
    .saveAsTable('airlines_year_origin') # Creates a table partitioned by year and origin, with origin acting as a "tie breaker". This means files will be stored in a year-origin subdirectory.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail airlines_year_origin

# COMMAND ----------
```
- Partitioning involves separating the data within a table into different directories. This can help optimize query performance by reducing the amount of data that needs to be scanned for targeted queries.
- Partitioning should be used with caution for the following reasons:
  - If usage patterns change, partitioning needs to be updated. For example, if data analysis requires the partitioning of sales data to be changed from city to year, the partitioning for the sales data needs to be redone.
  - Changing partitioning of a table involves re-writing the entire table.
  - There's a higher chance of data being skewed. For example, if there are a lot sales in a particular city or year.
  - Partitioning can cause issues when a table comprises of a large number of files.
  - DataBricks recommends not partitioning a table that is less than 1TB in size. When partitioning is performed, each partition should hold at least 1GB of data,
- When a table is partitioned, the parquet files stored in S3 are saved into separate folders. This reduces the number of files that need to be scanned when performing queries.
## Delta Optimizations – Bloom Filter
```python
# Databricks notebook source
# MAGIC %md
# MAGIC - choose the highest cardinal column in the table - values that are unique to each row
# MAGIC - can be used to index other columns that are not in the first 32 columns where there is no stats calculated
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC use my_new_schema

# COMMAND ----------

spark.conf.set('spark.databricks.io.cache.enabled', 'false')

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC select * from csv.`dbfs:/databricks-datasets/asa/airlines/2007.csv`

# COMMAND ----------

from pyspark.sql.functions import *
spark.read.csv('dbfs:/databricks-datasets/asa/airlines/2007.csv', header=True, inferSchema=True) \
        .withColumn('flight_id', concat_ws('-',col('Year'), col('Month'), col('DayOfMonth'), col('UniqueCarrier'), col('FlightNum'))).write.mode('overwrite').saveAsTable('airlines_indexed') # Creates a highly-cardinal flight_id column in the airlines_indexed table.

# COMMAND ----------

spark.table('airlines_indexed').limit(100).display()

# COMMAND ----------


df = spark.read.csv('dbfs:/databricks-datasets/asa/airlines/2007.csv', header=True, inferSchema=True)
df.withColumn('flight_id', concat_ws('-', col('Year'), col('Month'), col('DayOfMonth'), col('UniqueCarrier'), col('FlightNum'))) \
  .write.mode('overwrite').saveAsTable('airlines_indexed_no_bloom')

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from airlines_indexed_no_bloom where flight_id in ('2007-1-1-WN-2755')

# COMMAND ----------

# MAGIC %sql
# MAGIC CREATE BLOOMFILTER INDEX ON
# MAGIC TABLE airlines_indexed
# MAGIC FOR COLUMNS(flight_id) # Creates a bloom filter using the flight_id column.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail airlines_indexed

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from airlines_indexed where flight_id in ('2007-1-1-WN-2755')

# COMMAND ----------

# MAGIC %sql
# MAGIC explain extended select * from airlines_indexed where flight_id in ('2007-1-1-WN-2755') # Shows that the file scan that was executed during the query used the previously created bloom filter.

# COMMAND ----------

# MAGIC %

# COMMAND ----------
```
- A bloom filter is similar to an index in RDBMS. You can index any column with high cardinality (a lot of unique values). Bloom filters are typically added to the highest-cardinality column in a dataset.
- The bloom filter helps skip data when doing file scans during a query. The bloom filter adds a separate index that will tell the optimizer if data is present in a file (can produce false positives, but not false negatives). DataBricks will scan this index first before scanning the table, allowing it to skip data and scan efficiently.
- Bloom filters are best used for optimizing queries that look for "a needle in a haystack" (i.e. a very small subset of data in a large dataset.)
## Delta Optimization – Z-Ordering
```python
# Databricks notebook source
# MAGIC %md
# MAGIC - Usually more efficient than partition as avoids over partitioning 
# MAGIC - more efficient in data skipping 
# MAGIC - tables with more columns we use z-ordering as it's more efficient 
# MAGIC - Typically used for higher cardinality columns
# MAGIC - columns should be in the first 32 of the tables
# MAGIC - as usage patterns changes we need to change the partition and z ordering 
# MAGIC - z-order can be used in partitions also
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC https://medium.com/@srinidhichundru/deep-dive-into-liquid-clustering-z-ordering-and-partitioning-part-2-4c11eb18ba8f

# COMMAND ----------

# MAGIC %sql
# MAGIC use my_new_schema

# COMMAND ----------

spark.conf.set('spark.databricks.io.cache.enable', False)

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from csv.`dbfs:/databricks-datasets/asa/airlines/2007.csv`

# COMMAND ----------

# MAGIC %md
# MAGIC multipartitions

# COMMAND ----------

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/{2007, 2008}.csv', header=True, inferSchema=True) \
    .write \
    .format('delta') \
    .mode('overwrite') \
    .saveAsTable('airlines_year')

# COMMAND ----------

# MAGIC %sql
# MAGIC optimize airlines_year # Performs file compaction.
# MAGIC zorder by (year, origin) # Specifies z-ordering based on year and origin.

# COMMAND ----------

spark.conf.set('spark.databricks.delta.retentionDurationCheck.enabled', 'false')

# COMMAND ----------

# MAGIC %sql
# MAGIC -- set spark.databricks.delta.systemDefault.vacuum.retentionDurationCheck.enabled = false
# MAGIC vacuum airlines_year retain 0 hours

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail airlines_year

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC desc history airlines_year

# COMMAND ----------

# MAGIC %md
# MAGIC

# COMMAND ----------

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/{2007, 2008}.csv', header=True, inferSchema=True) \
    .write \
    .format('delta') \
    .mode('overwrite') \
    .partionBy('Year') \
    .saveAsTable('airlines_year_2') # Creates a table that is partitioned by year

# COMMAND ----------

# MAGIC %sql
# MAGIC optimize airlines_year_2
# MAGIC where year = 2007
# MAGIC zorder by (Origin) # Creates a z-order specifically for the 2007 partition of the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC set spark.databricks.delta.retentionDurationCheck.enabled = false
# MAGIC vacuum airlines_year_2 retain 0 hours

# COMMAND ----------

# MAGIC %sql
# MAGIC optimize airlines_year_2
# MAGIC where year >=2000 and year <=2007
# MAGIC zorder by (Origin)

# COMMAND ----------

# MAGIC %sql
# MAGIC set spark.databricks.delta.retentionDurationCheck.enabled = false
# MAGIC vacuum airlines_year_2 retain 0 hours

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC desc history airlines_year_2
```
- As with bloom filtering, z-ordering helps with data skipping by reducing the number of files that need to be scanned during a query. Z-ordering is typically used on columns with mid-range to high cardinality.
- Unlike bloom filtering columns chosen for z-ordering must be within the first 32 columns of a table.
- When usage patterns change, partitioning and z-ordering must be redone.
- z-ordering can be performed at the table level and the partition level. ordering can be done on a single partition or multiple partitions.
- z-ordering is more flexible than bloom filtering in that a new z-order can be created for a new partition or set of partitions and the ordering will apply to all new records in the table, without disturbing the z-ordering of existing records.
## Delta Lake Liquid Clustering
```python
# Databricks notebook source
# MAGIC %sql
# MAGIC use my_new_schema

# COMMAND ----------

# MAGIC %md
# MAGIC - Incremental clustering
# MAGIC - Flexible to change the columns
# MAGIC - smarter than a zorder
# MAGIC - visibility to the end users with one query
# MAGIC - possible to use this command while DDL (As with partitioning, it is defined when a table is created).

# COMMAND ----------

# MAGIC %md
# MAGIC https://medium.com/@srinidhichundru/deep-dive-into-liquid-clustering-z-ordering-and-partitioning-part-2-4c11eb18ba8f

# COMMAND ----------

# MAGIC %sql
# MAGIC create or replace table flight_details(
# MAGIC  `Year` INT ,
# MAGIC  `Month` INT ,
# MAGIC  `DayofMonth` INT ,
# MAGIC  `DayOfWeek` INT ,
# MAGIC  `DepTime` STRING,
# MAGIC  `CRSDepTime` INT ,
# MAGIC  `ArrTime` STRING,
# MAGIC  `CRSArrTime` INT ,
# MAGIC  `UniqueCarrier` STRING,
# MAGIC  `FlightNum` INT ,
# MAGIC  `TailNum` STRING,
# MAGIC  `ActualElapsedTime` STRING,
# MAGIC  `CRSElapsedTime` STRING,
# MAGIC  `AirTime` STRING,
# MAGIC  `ArrDelay` STRING,
# MAGIC  `DepDelay` STRING,
# MAGIC  `Origin` STRING,
# MAGIC  `Dest` STRING,
# MAGIC  `Distance` INT ,
# MAGIC  `TaxiIn` string ,
# MAGIC  `TaxiOut` string ,
# MAGIC  `Cancelled` INT ,
# MAGIC  `CancellationCode` STRING,
# MAGIC  `Diverted` INT ,
# MAGIC  `CarrierDelay` string ,
# MAGIC  `WeatherDelay` string ,
# MAGIC  `NASDelay` string ,
# MAGIC  `SecurityDelay` string ,
# MAGIC  `LateAircraftDelay` string )
# MAGIC  CLUSTER BY (Year, Dest) # Defines the liquid cluster based on year and destination.
# MAGIC

# COMMAND ----------

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/2007.csv', header=True, inferSchema=True).limit(10).display()

# COMMAND ----------

from pyspark.sql.functions import *

spark.read.csv('dbfs:/databricks-datasets/asa/airlines/2*.csv', header=True, inferSchema=True) \
    .write.mode('append').option('mergeSchema', 'true').saveAsTable('flight_details')

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail flight_details # Shows the columns used in liquid clustering under 'clusteringColumns'.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history flight_details # Clustering occurs during each write operation.

# COMMAND ----------

# show s3 files in aws

# COMMAND ----------

# MAGIC %sql
# MAGIC optimize flight_details

# COMMAND ----------

spark.table('flight_details').limit(100).write.mode('append').option('mergeSchema', 'true').saveAsTable('flight_details')

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history flight_details

# COMMAND ----------

spark.table('flight_details').limit(1000).write.mode('append').option('mergeSchema', 'true').saveAsTable('flight_details')

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history flight_details

# COMMAND ----------

# MAGIC %md
# MAGIC
# MAGIC #### changing clustering
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC ALTER TABLE flight_details cluster by (Month) # Changes clustering strategy to 'Month' instead of year and destination.

# COMMAND ----------

# MAGIC %sql
# MAGIC optimize flight_details # When the table is optimized, the table is re-clustered according to the altered parameters.

# COMMAND ----------

# MAGIC %sql
# MAGIC use my_new_schema

# COMMAND ----------

spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled", "false")

# COMMAND ----------

# MAGIC %sql
# MAGIC set spark.databricks.delta.retentionDurationCheck.enabled = false;
# MAGIC vacuum flight_details retain 0 hours; # Cleans unused data files.

# COMMAND ----------

# MAGIC %sql
# MAGIC alter table flight_details cluster by (UniqueCarrier, TailNum);
# MAGIC optimize flight_details;

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history flight_details

# COMMAND ----------

# MAGIC %sql
# MAGIC alter table flight_details clustered by None # Removes clustering from the table if it's no longer needed.

# COMMAND ----------
```
- Liquid clustering is a more advanced optimization strategy than z-ordering because it uses a more efficient algorithm to group data.
- Unlike z-ordering and bloom filtering, which only occur when an `optimize` command is run on a table, liquid clustering occurs during each write operation.

## Delta Lake Hands-On Demo

Concept review: [[02_technical_prep/Databricks Review|Databricks Review]].

```python
# Databricks notebook source
# MAGIC %md
# MAGIC 1. Learning about Delta Lake 
# MAGIC 2. Understanding _delta_log structure and meaning
# MAGIC 3. Understanding importance of file skipping

# COMMAND ----------

spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.optimizeWrite", False)
spark.conf.set("spark.databricks.delta.properties.defaults.autoOptimize.autoCompact", False)

# COMMAND ----------

# MAGIC %sql
# MAGIC use s3catalog.default; # Uses the default schema of the s3 catalog.

# COMMAND ----------

# MAGIC %sql
# MAGIC DROP TABLE IF EXISTS delta_lake_testing;
# MAGIC
# MAGIC CREATE OR REPLACE TABLE delta_lake_testing (ID INT, VALID BOOLEAN NOT NULL, NAME STRING)
# MAGIC using delta # Describes the table format (default is parquet).
# MAGIC LOCATION 's3://testdatabricks1992/delta_lake_testing'; # Creates and stores the delta table in the specified S3 location, which acts as a delta lake.

# COMMAND ----------

# MAGIC %sql
# MAGIC ALTER TABLE s3catalog.default.delta_lake_testing ALTER COLUMN VALID SET NOT NULL;
# MAGIC ALTER TABLE s3catalog.default.delta_lake_testing ADD CONSTRAINT ID_CHECK_1 CHECK (ID > 0 AND ID < 9999999)

# COMMAND ----------

# MAGIC %sql DESCRIBE s3catalog.default.delta_lake_testing # Describes the table metadata, including column names and data types.

# COMMAND ----------

# MAGIC %sql DESCRIBE DETAIL s3catalog.default.delta_lake_testing # Describes table details such as format, id, name, and location.

# COMMAND ----------

# MAGIC %sql
# MAGIC DESCRIBE HISTORY s3catalog.default.delta_lake_testing # Describes the transaction logs for the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC INSERT INTO s3catalog.default.delta_lake_testing VALUES(100, FALSE, 'BOB')

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC DESCRIBE HISTORY s3catalog.default.delta_lake_testing # Describes all transactions that have been performed on the table. Each row in this table has a version number.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing;

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into s3catalog.default.delta_lake_testing values (-1, True, "alice") # This insert violates the constraints set above and throws an error.

# COMMAND ----------

# MAGIC %sql
# MAGIC DESCRIBE HISTORY s3catalog.default.delta_lake_testing

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing

# COMMAND ----------

## show the files and describe history

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into s3catalog.default.delta_lake_testing values (2, FALSE, "bob"), (3, FALSE, "steve"), (4, FALSE, "ray")

# COMMAND ----------

# MAGIC %sql
# MAGIC DESCRIBE HISTORY s3catalog.default.delta_lake_testing

# COMMAND ----------

# MAGIC %sql
# MAGIC SELECT * FROM s3catalog.default.delta_lake_testing VERSION AS OF 4  # Retrieves the version of the table based on the version number in the transaction history table.

# COMMAND ----------

# MAGIC %sql
# MAGIC SELECT * FROM s3catalog.default.delta_lake_testing TIMESTAMP AS OF '2025-08-04T10:11:41.000+00:00'; # Retrieves the version of the table based on a specified timestamp.

# COMMAND ----------

## show parquet files

# COMMAND ----------

# MAGIC %md
# MAGIC Updates
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing;

# COMMAND ----------

# MAGIC %sql
# MAGIC update s3catalog.default.delta_lake_testing 
# MAGIC set name =  'BOB_updated_name'
# MAGIC where name  = 'BOB' # Updates the name of record(s) with a name of 'BOB'.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing

# COMMAND ----------

# MAGIC %sql 
# MAGIC describe history s3catalog.default.delta_lake_testing

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing version as of 5

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing

# COMMAND ----------

## show the history and s3 location


# COMMAND ----------

# MAGIC %sql
# MAGIC delete from s3catalog.default.delta_lake_testing where name = 'BOB_updated_name'

# COMMAND ----------


# COMMAND ----------

# MAGIC %md
# MAGIC MERGE

# COMMAND ----------

# MAGIC %sql
# MAGIC create or replace table updates (id Int, valid boolean, name string, operations string)
# MAGIC using delta 
# MAGIC LOCATION 's3://testdatabricks1992/updates';

# COMMAND ----------

# MAGIC %sql
# MAGIC INSERT INTO `updates` (id, valid, name, operations)
# MAGIC VALUES 
# MAGIC     (4, True, 'Ray', ''), 
# MAGIC     (3, True, 'Steve', 'Delete'), 
# MAGIC     (6, True, 'Bob', ''), 
# MAGIC     (2, True, 'Alice', 'Delete');

# COMMAND ----------

# MAGIC %sql
# MAGIC merge into s3catalog.default.delta_lake_testing USING `updates` ON s3catalog.default.delta_lake_testing.id = `updates`.id 
# MAGIC when matched and `updates`.operations = 'Delete' then
# MAGIC     delete
# MAGIC when matched then
# MAGIC     update set s3catalog.default.delta_lake_testing.valid = `updates`.valid, s3catalog.default.delta_lake_testing.name = `updates`.name
# MAGIC when not matched then
# MAGIC     insert (id, valid, name) values (`updates`.id, `updates`.valid, `updates`.name)

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from s3catalog.default.delta_lake_testing

# COMMAND ----------


for x in range(1, 10000):
    query = f"insert into  s3catalog.default.delta_lake_testing values ({x}, true, {x})"

    spark.sql(query) # A separate file is created in the S3 transaction log for each insert performed.

# COMMAND ----------


table_path = spark.sql("DESC DETAIL s3catalog.default.delta_lake_testing").collect()[0]['location']
table_path
# DBTITLE 1

# COMMAND ----------

log_files = dbutils.fs.ls(f"{table_path}/_delta_log/")
log_files

# COMMAND ----------

data_files = dbutils.fs.ls(f"{table_path}/")

data_files


# COMMAND ----------

lastest_json_file = sorted([f for f in log_files if f.name.endswith(".json")], key=lambda x: x.modificationTime, reverse=True)[0].path
lastest_json_file

# COMMAND ----------

latest_parquet_file = sorted([f for f in data_files if f.name.endswith("parquet")], key=lambda f: f.modificationTime, reverse=True)[0].path
latest_parquet_file

# COMMAND ----------

valuses = ", ".join([f"({x}, true, 'name {x}')" for x in range (100, 1010)]) + ", " + "(-1, true, 'error')"
query = f"insert into s3catalog.default.delta_lake_testing values {valuses}"
print(query)


# COMMAND ----------

spark.sql(query)

# COMMAND ----------

table_path = spark.sql("DESC DETAIL s3catalog.default.delta_lake_testing").collect()[0]['location']
log_files = dbutils.fs.ls(f"{table_path}/_delta_log/")
data_files = dbutils.fs.ls(f"{table_path}/")
lastest_json_file = sorted([f for f in log_files if f.name.endswith(".json")], key=lambda x: x.modificationTime, reverse=True)[0].path
latest_parquet_file = sorted([f for f in data_files if f.name.endswith("parquet")], key=lambda f: f.modificationTime, reverse=True)[0].path
lastest_json_file
latest_parquet_file

# COMMAND ----------

lastest_json_file

# COMMAND ----------
```

## Time Travel in Delta Lake

Concept review: [[02_technical_prep/Databricks Review|Databricks Review]].

```python
# Databricks notebook source
# MAGIC %md
# MAGIC  * delta.logRetentionDuration = "interval <interval>
# MAGIC  * delta.deletedFileRetentionDuration = "interval <interval>

# COMMAND ----------

# MAGIC %sql
# MAGIC use s3catalog.default; # Use the default schema for the S3 catalog.

# COMMAND ----------

dbutils.fs.rm('s3://testdatabricks1992/delta_schema_enforcement', True) # Removing the table from S3

# COMMAND ----------

# MAGIC %sql
# MAGIC DROP TABLE IF EXISTS delta_schema_enforcement; # Removing the table from data bricks.
# MAGIC
# MAGIC CREATE OR REPLACE TABLE delta_schema_enforcement (id int, name STRING)
# MAGIC using delta 
# MAGIC LOCATION 's3://testdatabricks1992/delta_schema_enforcement'; # Creating a new table using the same parameters.
# MAGIC
# MAGIC insert into delta_schema_enforcement values (1, "alis");
# MAGIC insert into delta_schema_enforcement values (2, "bob");
# MAGIC insert into delta_schema_enforcement values (3, "clare");

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC delete from delta_schema_enforcement # 'Accidentally' deletes all data from the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement # No data to show.

# COMMAND ----------

# MAGIC %sql
# MAGIC Describe history delta_schema_enforcement # Shows the transaction history of the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC
# MAGIC select * from delta_schema_enforcement@v3 # Selects version 3 of the table (the version before the delete operation).

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement timestamp as of '2025-08-06T19:03:31.000+00:00' # Selects the version of the table associated with the timestamp of v3.

# COMMAND ----------

# MAGIC %md
# MAGIC restore table

# COMMAND ----------

# MAGIC %sql
# MAGIC restore table delta_schema_enforcement to version as of 3; # Restores the table to version 3 (essentially undoing the delete operation).

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC describe history delta_schema_enforcement # The restore operation adds a fifth version to the history. It doesn't create a new table with a new history.

# COMMAND ----------

# MAGIC %md
# MAGIC Vaccum

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement # Deletes stale data files older than 7 days (default retention).

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement dry run # Shows which files would be deleted without actually deleting them.

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours dry run # Specifies a retention period of 0 days (7 days is the default).

# COMMAND ----------

spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled","false") # This setting needs to be set to false before running a vacuum command with a retention period shorter than 7 days.

# COMMAND ----------

# MAGIC %sql
# MAGIC insert into delta_schema_enforcement values (1, "Daisy");
# MAGIC delete from delta_schema_enforcement where name = 'Daisy'

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours dry run # Performs a dry-run vacuum on all stale data files.

# COMMAND ----------

# MAGIC %sql
# MAGIC vacuum delta_schema_enforcement retain 0 hours # Performs an actual dry-run vacuum of all stale data files.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history delta_schema_enforcement

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement@v6

# COMMAND ----------

# MAGIC %sql
# MAGIC select * from delta_schema_enforcement@v5

# COMMAND ----------

# MAGIC %sql
# MAGIC alter table delta_schema_enforcement
# MAGIC   set tblproperties ('delta.deletedFileRetentionDuration' = 'interval 14 Days')

# COMMAND ----------

# MAGIC %sql
# MAGIC desc Extended delta_schema_enforcement

# COMMAND ----------
```

## Delta Optimization – Default Small Files

Concept review: [[02_technical_prep/Databricks Review|Databricks Review]].

```sql
# Databricks notebook source
# MAGIC %sql
# MAGIC set spark.daatabricks.delta.properties.default.autoOptimize.optimizeWrite = false
# MAGIC set spark.databricks.delta.properties.default.autoOptimize.autoCompact = false
# MAGIC

# COMMAND ----------

# MAGIC %sql
# MAGIC SET spark.databricks.delta.autoCompact.enabled = false;
# MAGIC SET spark.databricks.delta.optimizeWrite.enabled = false; # Suspends optimization settings set to true by default.

# COMMAND ----------

# MAGIC %sql
# MAGIC use my_new_schema

# COMMAND ----------

# MAGIC %sql
# MAGIC CREATE OR REPLACE TABLE small_file_2 (id int, name string)

# COMMAND ----------

for i in range(1, 201): # Inserts 200 small files into the table.
    query = f" Insert into small_file_2 VALUES ({i}, '{i + i}')"

    spark.sql(query)

# COMMAND ----------

# MAGIC %sql
# MAGIC desc detail small_file_2 # Shows the number of files inserted into the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history small_file # Shows the 200 write operations performed on the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC select avg(id) from small_file_2 # Shows how performance (600 ms) is diminished by the excessive amount of small files in the table.

# COMMAND ----------

# MAGIC %sql
# MAGIC create or replace table big_file (id Int, data string)
# MAGIC

# COMMAND ----------

values = ', '.join(f"({i}, '{i + i}')" for i in range(1, 201))
query = f" Insert into big_file VALUES {values}"

spark.sql(query) # Inserts all the values at once, instead of performing 200 separate inserts.

# COMMAND ----------

query

# COMMAND ----------

# MAGIC %sql
# MAGIC select avg(id) from big_file # Query performance (29 ms) significantly improved because only one file needed to be scanned.

# COMMAND ----------

# MAGIC %sql
# MAGIC optimize small_file_2 # Compacts the 200 files in the table into 1 or 2 files.

# COMMAND ----------

# MAGIC %sql
# MAGIC select avg(id) from small_file_2 # Query performance (7 ms) improves after optimization was run on the table.

# COMMAND ----------

# MAGIC %md
# MAGIC #### Auto optimize
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC []()

# COMMAND ----------

Optimized write
auto compaction

# COMMAND ----------

MERGE
UPDATE with subqueries
DELETE with subqueries
CTAS statements and 
INSERT 

# COMMAND ----------

# MAGIC %sql
# MAGIC set spark.databricks.delta.autoCompact.enabled = true
# MAGIC   

# COMMAND ----------

spark.conf.get("spark.databricks.delta.autoCompact.minNumFiles") # Default is 50 files.

# COMMAND ----------

# MAGIC %sql
# MAGIC set spark.databricks.delta.autoCompact.minNumFiles = 10

# COMMAND ----------

# MAGIC %sql
# MAGIC Create or replace table small_file_auto (id Int, name string)
# MAGIC     
# MAGIC

# COMMAND ----------

for i in range(1, 30):
    query = f" Insert into small_file_auto VALUES ({i}, '{i + i}')"

    spark.sql(query)

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history small_file_auto

# COMMAND ----------

# MAGIC %sql
# MAGIC desc history small_file_2

# COMMAND ----------
```
