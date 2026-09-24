---
aliases:
  - "spark_databricks_introduction"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Spark and DataBricks/spark_databricks_introduction.md"
imported: 2026-09-24
---

# PySpark and Databricks Labs

Review: [[02_technical_prep/system_design/Data Processing Systems (Spark)|Data Processing Systems (Spark)]] · [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]] · [[02_technical_prep/Databricks Review|Databricks Review]]

## Contents

- [[#Setting Up DataBricks Account]]
- [[#Cluster Creation and Notebook Fundamentals]]
- [[#DataBricks Project 1]]
- [[#Create Volumes & Link S3]]
- [[#DataBricks Project 2]]
- [[#Pandas on Spark]]
- [[#PySpark Reading CSV Files]]
- [[#Pyspark Joins DataBricks]]
- [[#Pyspark Explode DataBricks]]
- [[#PySpark Pivot DataBricks]]
- [[#DataBricks Project 3]]
- [[#PySpark Handling Bad Records]]
- [[#PySpark Optimizations (Reading DAGs)]]
- [[#Understanding Spark UI For PySpark and DataBricks]]
- [[#PySpark Cache, Persist, & Storage Levels in DataBricks]]
- [[#PySpark Skew Data & Salting Techniques in Databricks]]
- [[#PySpark UDF & Vectorized UDFs in Databricks]]

## Setting Up DataBricks Account
- Set up account using the Free Edition, then create a basic S3 bucket in AWS to link with the account.
- When setting up a workspace, create an external location, specifying the name of the S3 bucket you created as `s3://{bucket_name}`. Setting up this external location will automatically link the S3 bucket with your data bricks account, using the personal access token generated during creation of the external location.
- After setting up the external location, create a notebook. Once the notebook is created, use the following command to create a catalog:
  ```sql
  create catalog 'calvinfr1-demo' managed location 's3://dea-databricks-demo-bucket-cf1/calvinfr1-demo/';
  ```
- Next, create a schema:
  ```sql
  create schema `calvinfr1-demo`.my_schema managed location 's3://dea-databricks-demo-bucket-cf1/calvinfr1-demo/my_schema';
  ```
- Next, create a table:
  ```python
  df.write.format('delta').saveAsTable("`calvinfr1-demo`.my_schema.employee")
  ```
## Cluster Creation and Notebook Fundamentals
- Creating a Cluster:
  - Create a new 'compute' (not available in Free edition). Specify the policy as 'unrestricted', deselect 'Photon acceleration' to reduce cost, specify worker type (memory and min/max number of cores or single node), specify the 'terminate after' time to stop the compute after a specified amount of inactivity.
  - Within a cluster, you can create notebooks. In these notebooks, you will create SQL/Python code to perform data engineering tasks.
  - When a cluster is created, it automatically records metrics such as CPU and memory utilization to help with optimizing the performance of tasks that you create, helping to reduce your overall cost burden.
## DataBricks Project 1
```python
# Databricks notebook source
# DBTITLE 1,import required lib
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# COMMAND ----------

customer = spark.table('samples.bakehouse.sales_customers')

# COMMAND ----------

customer.show() # Shows the customer table, but not as neatly as customer.display(), which is a function specific to data bricks.

# COMMAND ----------

customer_schema

# COMMAND ----------

# DBTITLE 1,read customers tables


customer = customer.withColumn('full_name', F.concat_ws(' ', F.col('first_name'), F.col('last_name')))

# COMMAND ----------

customer.display()

# COMMAND ----------

# DBTITLE 1,Read transaction table
transaction = spark.table('samples.bakehouse.sales_transactions')

# COMMAND ----------

transaction.select('customerID').distinct().count()

# COMMAND ----------

customer.select("customerID").distinct().count()

# COMMAND ----------

customer.display()

# COMMAND ----------

# DBTITLE 1,Join customer and transaction
customer_transaction = customer.join(
    transaction,
    F.substring(customer.customerID.cast("string"), -3, 3) == F.substring(transaction.customerID.cast("string"), -3, 3),
    'inner'
)

# COMMAND ----------

customer_transaction.display()

# COMMAND ----------

transaction.select('customerID').distinct().orderBy('customerID').display()

# COMMAND ----------

customer.select('customerID').distinct().orderBy('customerID').display()

# COMMAND ----------

# DBTITLE 1,clean dataframe and remove duplicate columns
customer_transaction_clean = customer_transaction.drop(transaction.customerID)
display(customer_transaction_clean)

# COMMAND ----------

# DBTITLE 1,Group and Agg function
from pyspark.sql.functions import count, avg

avg_orders = customer_transaction_clean.groupBy('customerID', 'full_name').agg(count('*').alias('orders'))

display(avg_orders.orderBy('orders'))

# COMMAND ----------

# DBTITLE 1,Customers who had the min sales and max sales


w = Window.orderBy(F.desc('orders'))
max_orders = avg_orders.withColumn('rank', F.rank().over(w)).filter(F.col('rank') == 1).drop('rank')

w_asc = Window.orderBy('orders')
min_orders = avg_orders.withColumn('rank', F.rank().over(w_asc)).filter(F.col('rank') == 1).drop('rank')

display(max_orders)
display(min_orders)

# COMMAND ----------

# DBTITLE 1,Top 10 Customers who had the most sales
revenue_per_customer = customer_transaction_clean.groupBy('customerID', 'Full_name').agg(
    F.sum('totalPrice').alias('total_revenue')
)

w_revenue = Window.orderBy(F.desc('total_revenue'))
top_customers = revenue_per_customer.withColumn('rank', F.rank().over(w_revenue)).filter(F.col('rank') <= 10).drop('rank')

display(top_customers)

# COMMAND ----------
```
## Create Volumes & Link S3
- Within data bricks, under catalog, you can create a new external location within a specified schema. When creating an external location for AWS, you need to specify the name of an existing S3 bucket. Generate and copy the personal access token from the UI. This will be needed to create CloudFormation stack following prompts from the data bricks UI.
- In addition to views, materialized views, and tables, you can also create volumes within a schema. Volumes are generally used to store unstructured data. The main difference between a volume and an external location is that a volume is not integrated with a cloud platform such as AWS.
- Uploading a Data Set to a Volume:
  - Files can be uploaded manually to a volume (created under a specific catalog and schema) using the data bricks UI. Once these files are uploaded into the volume, they can be manipulated using python code in a notebook.
- Uploading and Displaying Files:
  ```python
  from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType, LongType, DecimalType

  # Loading Data Files Into DataBricks
  csv_file = "/Volumes/workspace/default/external_data/sampledata_csv.csv" # You can also specify '*.csv' to read every CSV file.
  json_file = "/Volumes/workspace/default/external_data/sampledata_json.json"
  parquet_file = "/Volumes/workspace/default/external_data/sampledata_parquet.parquet"

  csv_data = spark.read.format('csv').load(csv_file) # Expects field delimiter to be a comma
  # csv_data.display() # No header was present in the CSV file, so we can create one manually as follows:

  # Reading CSV Data
  custom_schema = StructType([
      StructField('c_custom_key', LongType(), True), # column name, data type, nullable
      StructField('c_name', StringType(), True),
      StructField('c_address', StringType(), True),
      StructField('c_nation_key', LongType(), True),
      StructField('c_phone', StringType(), True),
      StructField('c_account_balance', DecimalType(18, 2), True),
      StructField('c_market_segment', StringType(), True),
      StructField('c_comment', StringType(), True)])

  csv_data_custom_schema = spark.read.format('csv') \
      .option('header', True) \
      .schema(custom_schema) \
      .load(csv_file)
  display(csv_data_custom_schema)

  # Reading JSON Data
  json_data = spark.read.json(json_file)
  json_data.display()

  # Reading Parquet Data
  parquet_data = spark.read.parquet(parquet_file)
  display(parquet_data)
  ```
## DataBricks Project 2
```python
# Databricks notebook source
# MAGIC %md
# MAGIC ### Agenda
# MAGIC - Learn to read csv files with headers and schema
# MAGIC - extract columns, and display
# MAGIC - filter datasets
# MAGIC - rename columns 
# MAGIC - select specific columns
# MAGIC - order by column values
# MAGIC - agg function count, min, max
# MAGIC - distinct
# MAGIC
# MAGIC ### Dataset
# MAGIC - San Francisco Fire Department Public Dataset
# MAGIC - https://www.kaggle.com/datasets/imankity/san-francisco-fire-department-public-dataset
# MAGIC
# MAGIC ### Analysis
# MAGIC - What were all the different types of fire calls in 2018?
# MAGIC - What months within the year 2018 saw the highest number of fire calls?
# MAGIC - Which neighborhood in San Francisco generated the most fire calls in 2018
# MAGIC - Which neighborhoods had the worst response times to fire calls in 2018
# MAGIC - Which week in the year in 2018 had the most fire calls?
# MAGIC - Is there a correlation between neighborhood, zip code, and number of fire calls

# COMMAND ----------

from pyspark.sql import functions as F

# COMMAND ----------

fire_department_data  = "/Volumes/workspace/default/external_data/databricks_project2_data.csv"

df = (
    spark
    .read
    .format('csv')
    .option('header', 'true')
    .option('inferSchema', 'true') # Infers data type of column. May incorrectly infer a data type as string.
    .load(fire_department_data)
)
df.printSchema() # Prints the schema (Column name, data type, isNullable)
df.columns # Prints the column names as a list.
len(df.columns) # Number of columns.
type(df.columns)

df.show(10, False) # Shows the first 10 rows of the data frame (not as neatly as display()).

df.display()

# COMMAND ----------

# MAGIC %md
# MAGIC Projections and filters

# COMMAND ----------

few_df = (
    df.select("Incident Number", "Available DtTm", "Call Type")
    .where(F.col("Call Type")!= "Medical Incident")
)
few_df.display() # Selects and displays the specified columns from the data frame.

# COMMAND ----------

# MAGIC %md
# MAGIC Renaming, adding, and dropping columns. 

# COMMAND ----------

df.select('Delay').display()

# COMMAND ----------

renamed_df = (
    df.withColumn("ResponseDelayedinMins", F.col("Delay").cast("decimal")) # This command adds the 'ResponseDelayedinMins' column to the data frame, populating it with the data from the 'Delay' column that is casted from a string to a decimal.
)

renamed_df.where(F.col("ResponseDelayedinMins") > 5).display()

# COMMAND ----------

# MAGIC %md
# MAGIC Casting columns type

# COMMAND ----------

fire_ts_df = (
    renamed_df.withColumn("IncidentDate", F.to_timestamp(F.col("CallDate"), "MM/dd/yyyy"))
    .withColumn("OnWatchDate", F.to_timestamp(F.col("WatchDate"), "MM/dd/yyyy"))
    .withColumn("AvailableDtTS", F.to_timestamp(F.col("AvailableDtTm"), "MM/dd/yyyy hh:mm:ss a"))
    .drop(*["CallDate","WatchDate","AvailableDtTm"]) # Drops the existing columns after using the data in those columns to create the new columns (effectively, a replacement).
)

fire_ts_df.select("IncidentDate", "OnWatchDate", "AvailableDtTS").display() # Confirms the columns were added properly.

# COMMAND ----------

(fire_ts_df
 .select(F.year('IncidentDate'))
 .distinct()
 .orderBy(F.year('IncidentDate'))
 .display())

# COMMAND ----------

# MAGIC %md
# MAGIC Aggregations

# COMMAND ----------

(fire_ts_df
 .select("CallType")
 .where(F.col("CallType").isNotNull())
 .groupBy("CallType")
 .count() # The same thing as COUNT(*) in SQL.
 .orderBy("count", ascending=False)
 .display())

# COMMAND ----------

# MAGIC %md
# MAGIC min(), max(), sum(), and avg()

# COMMAND ----------

(fire_ts_df.select(F.sum("NumAlarms"), F.avg("ResponseDelayedinMins"),
F.min("ResponseDelayedinMins"), F.max("ResponseDelayedinMins"))
.display())

# COMMAND ----------

(fire_ts_df.groupBy("CallType").agg(F.avg("ResponseDelayedinMins"),
F.min("ResponseDelayedinMins"), F.max("ResponseDelayedinMins"))
.display())

# COMMAND ----------

# MAGIC %md
# MAGIC What were all the different types of fire calls in 2018?

# COMMAND ----------

fire_ts_df_2018 = fire_ts_df.where("year(IncidentDate) >= 2018")
fire_ts_df_2018.display()

# COMMAND ----------

fire_ts_df_2018.select("callType").distinct().display()

# COMMAND ----------

# MAGIC %md
# MAGIC  What months within the year 2018 saw the highest number of fire calls?

# COMMAND ----------

(fire_ts_df_2018
 .select(F.month("IncidentDate").alias("IncidentMonth"),"CallType")
 .groupBy(F.col("IncidentMonth"))
 .agg(F.count("CallType").alias("count_of_incident"))
 .orderBy(F.col("count_of_incident").desc(), "IncidentMonth",)).display()

# COMMAND ----------

# MAGIC %md
# MAGIC Which neighborhood in San Francisco generated the most fire calls in 2018

# COMMAND ----------

(fire_ts_df_2018
 .select(F.month("IncidentDate").alias("IncidentMonth"),"CallType", "Neighborhood")
 .groupBy(F.col("Neighborhood"))
 .agg(F.count("CallType").alias("count_of_incident"))
 .orderBy(F.col("count_of_incident").desc())).display()

# COMMAND ----------

# MAGIC %md
# MAGIC Which neighborhoods had the worst response times to fire calls in 2018

# COMMAND ----------

(fire_ts_df_2018
 .select(F.month("IncidentDate").alias("IncidentMonth"),"CallType", "Neighborhood","ResponseDelayedinMins")
 .groupBy(F.col("Neighborhood"),F.col("CallType"))
 .agg(F.avg("ResponseDelayedinMins").alias("avg_ResponseDelayedinMins"), F.count("CallType").alias("Total_incident"))
 .orderBy(F.col("avg_ResponseDelayedinMins").desc())).display()

# COMMAND ----------

# MAGIC %md
# MAGIC  Which week in the year in 2018 had the most fire calls?

# COMMAND ----------

(fire_ts_df_2018
 .select(F.month("IncidentDate").alias("IncidentMonth"), F.weekofyear("IncidentDate").alias("IncidentWeek"),"CallType")
 .groupBy(F.col("IncidentWeek"))
 .agg(F.count("CallType").alias("count_of_incident"))
 .orderBy(F.col("count_of_incident").desc(), "IncidentWeek",)).display()

# COMMAND ----------

# MAGIC %md
# MAGIC Is there a correlation between neighborhood, zip code, and number of fire calls

# COMMAND ----------

output_df = (fire_ts_df_2018
 .select("IncidentDate","CallType", "Neighborhood","ResponseDelayedinMins", "zipcode")
 .groupBy(*["zipcode", "Neighborhood"])
 .agg(F.avg("ResponseDelayedinMins").alias("avg_ResponseDelayedinMins"), F.count("CallType").alias("Total_incident"))
 .orderBy(F.col("avg_ResponseDelayedinMins").desc()))

# COMMAND ----------

output_df.display()

# COMMAND ----------

# MAGIC %md
# MAGIC • How can we use Parquet files or SQL tables to store this data and read it back?

# COMMAND ----------
```
## Pandas on Spark
- Working with the pandas library on large datasets can become tedious because it can only work on single-threaded applications. Spark solved this issue by developing a pandas API on top of spark. This enables scaling of the pandas library to larger, multi-threaded applications
  - [Documentation](https://spark.apache.org/docs/latest/api/python/getting_started/quickstart_ps.html)
- Working With Pandas on Spark:
  ```python
  # Databricks notebook source
  # MAGIC %md
  # MAGIC What is Pandas?
  # MAGIC

  # COMMAND ----------

  # MAGIC %md
  # MAGIC - Get one column: 
  # MAGIC   - df['Username']
  # MAGIC - Extract multiple columns:
  # MAGIC   - df[['Username', 'Age']]
  # MAGIC - Extract Range of rows:
  # MAGIC   - df['UserName'][:5]
  # MAGIC - Extract single cell:
  # MAGIC   - df['Username'][5]
  # MAGIC - Sort by a column:
  # MAGIC   - df.sort_value(['Username'])
  # MAGIC - Find data distribution:
  # MAGIC   - number_of_users = df['City'].count_values()
  # MAGIC - Plot
  # MAGIC   - number_of_users.plot(kind = 'bar')

  # COMMAND ----------

  # MAGIC %md
  # MAGIC https://spark.apache.org/docs/latest/api/python/getting_started/quickstart_ps.html

  # COMMAND ----------

  import pyspark.sql.functions as F
  import pyspark.pandas as ps
  import pandas as pd
  import os

  # COMMAND ----------

  os.environ['PYARROW_IGNORE_TIMEZONE'] = '1' # Eliminates tedious warnings.
  spark.conf.set("spark.sql.ansi.enabled", "false")
  spark.conf.set("spark.executorEnv.PYARROW_IGNORE_TIMEZONE", "1")

  # COMMAND ----------

  # DBTITLE 1,Creating a pandas df
  p_df = ps.DataFrame({
      "id" : [1, 2, 3, 4, 5],
      "name" : ["a", "b", "c", "d", "e"],
      "age" : [10, 20, 30, 40, 50]
  })

  p_df

  # COMMAND ----------

  # DBTITLE 1,mean of age
  p_df["age"].mean()

  # COMMAND ----------

  p_df["age_after_10_year"] = p_df["age"] * 10 # Adds a new column to the data frame using standard pandas syntax.

  # COMMAND ----------

  p_df

  # COMMAND ----------

  # DBTITLE 1,pandas data stats
  p_df.describe() # provides statistics, such as count, median, and standard deviation, for each column in the data frame.

  # COMMAND ----------

  p_df.display()

  # COMMAND ----------

  filtered_ps_df = p_df[p_df["age"] > 30] # Filters the data frame for consumers older than 30, using standard pandas syntax.
  filtered_ps_df.display() # The 'display' function  specific to spark works on a pandas data frame as well.
                        

  # COMMAND ----------

  spark_df = p_df.to_spark() # converts pandas data frame to a spark data frame.
  spark_df.display()

  # COMMAND ----------

  ps_df_from_spark = ps.DataFrame(spark_df) # Converts a spark data frame back to a pandas data frame.
  ps_df_from_spark.display()

  # COMMAND ----------
  ```
## PySpark Reading CSV Files
```python
# Databricks notebook source
csv_file = "/Volumes/workspace/default/external_datasets/sf-fire-calls.csv"

# COMMAND ----------

# DBTITLE 1,Reading csv file with infer_schema, header and separation
df = spark.read.format('csv').option('header', True).option('inferSchema', True).option('sep',',').load(csv_file)
df.display()

# COMMAND ----------

df = spark.read.format('csv').option('header', False).option('inferSchema', True).option('sep',',').load(csv_file)
df.display() # Will show the first row (with the column names) as a row, instead of using it to derive the names of columns. Columns will receive default naming of _c0, _c1, _c2, ect.

# COMMAND ----------

df = spark.read.format('csv').option('header', True).option('inferSchema', True).load(csv_file)
df.display() # There is no need to specify the separator with "option('sep',',')" if it is a comma.

# COMMAND ----------

df = spark.read.format('csv').options(header=True, inferSchema=True, sep=',').load(csv_file)
df.display() # specify all options at the same time, instead of using 'option' to specify settings individually.

# COMMAND ----------

df = spark.read.csv(header=True, inferSchema=True, sep='|', path=csv_file)
df.display() # Avoids needing to specify CSV with 'format' and allows you to specify options at the same time.

# COMMAND ----------

df.printSchema()

# COMMAND ----------

df.schema

# COMMAND ----------

csv_file_2 = "/Volumes/workspace/default/external_datasets/sampledata_csv.csv"

# COMMAND ----------

from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType, LongType, DecimalType

custom_schema = StructType([
    StructField('c_custkey', LongType(), True), 
    StructField('c_name', StringType(), True), 
    StructField('c_address', StringType(), True), 
    StructField('c_nationkey', LongType(), True), 
    StructField('c_phone', StringType(), True), 
    StructField('c_acctbal', DecimalType(18,2), True), 
    StructField('c_mktsegment', StringType(), True), 
    StructField('c_comment', StringType(), True)])

csv_data_custom_schema = spark.read.format('csv') \
    .schema(custom_schema) \ # There is no need to specify "option('header', True)" when using a custom schema.
    .load(csv_file)

csv_data_custom_schema.display()

# COMMAND ----------

# DBTITLE 1,Reading multiple files
csv_file_list = [
    '/Volumes/workspace/default/external_datasets/csv_file_example/part-00000-tid-2658311340503090957-69ac7bdd-0734-4fab-984b-2fb16125d781-213-1-c000.csv',
    
    '/Volumes/workspace/default/external_datasets/csv_file_example/part-00001-tid-2658311340503090957-69ac7bdd-0734-4fab-984b-2fb16125d781-209-1-c000.csv'
] 

df = spark.read.csv(header=True, inferSchema=True, sep=',', path=csv_file)
df.display()
df.count()

# COMMAND ----------

csv_folder = '/Volumes/workspace/default/external_datasets/csv_file_example/'
df = spark.read.csv(header=True, inferSchema=True, sep=',', path=csv_folder)
df.display()
df.count()

# COMMAND ----------
```
- It's always recommended to pass a custom schema when reading a file, because there will be an error if there is a difference between the custom schema and actual table schema, which helps detect unexpected schema changes.
## Pyspark Joins DataBricks
```python
# Databricks notebook source
# MAGIC %md
# MAGIC ## Joins
# MAGIC - A join in PySpark combines rows from two DataFrames based on a common key (or condition).
# MAGIC It’s similar to SQL joins — you can use join() or SQL-style syntax.
# MAGIC
# MAGIC ### Basic Syntax
# MAGIC   - df_joined = df1.join(df2, on="id", how="inner")
# MAGIC
# MAGIC   - https://stackoverflow.com/questions/53949197/isnt-sql-a-left-join-b-just-a
# MAGIC   - https://blog.codinghorror.com/a-visual-explanation-of-sql-joins/

# COMMAND ----------

# DBTITLE 1,creating the dataframe
data1 = [
    (1, "Pirate"),
    (2, "Monkey"),
    (3, "Ninja"),
    (4, "Spaghetti"),
    (None, "Unknown_Left")   # Null id in left
]

data2 = [
    (1, "Rutabaga"),
    (2, "Pirate"),
    (3, "Darth Vader"),
    (None, "Unknown_Right"),  # Null id in right
    (5, "Knight")
]

# COMMAND ----------

df1 = spark.createDataFrame(data1, ["id", "name"])
df2 = spark.createDataFrame(data2, ["id", "name"])

print("=== Table 1 ===")
df1.display()

print("=== Table 2 ===")
df2.display()

# COMMAND ----------

df1 = df1.withColumnRenamed("name", "name1")
df2 = df2.withColumnRenamed("name", "name2")

# COMMAND ----------

# DBTITLE 1,INNER JOIN - Keeps only rows with matching 'id' in both DataFrames.

inner_df = df1.join(df2, on="['id','name']", how="inner")
inner_df.display()

# COMMAND ----------

# DBTITLE 1,LEFT JOIN - Keeps all rows from LEFT (df1), fills unmatched RIGHT columns with null
left_df = df1.join(df2, on="id", how="left")
left_df.display()

# COMMAND ----------

# DBTITLE 1,RIGHT JOIN - Keeps all rows from RIGHT (df2), fills unmatched LEFT columns with null
right_df = df1.join(df2, on="id", how="right")
right_df.display()

# COMMAND ----------

# DBTITLE 1,FULL OUTER JOIN (same as OUTER)  - Keeps all rows from both DataFrames, fills nulls when no match
full_outer_df = df1.join(df2, on="id", how="full_outer")
full_outer_df.display()

# COMMAND ----------

# DBTITLE 1,LEFT OUTER JOIN - Same as Left join, all the rows from the Left table and matching or nulls from right
left_outer_df = df1.join(df2, on="id", how="left_outer")
left_outer_df.display()

# COMMAND ----------

# DBTITLE 1,RIGHT OUTER - Same as RIGHT join — all rows from right with matches or nulls
right_outer_df = df1.join(df2, on="id", how="right_outer")
right_outer_df.display()

# COMMAND ----------

# DBTITLE 1,LEFT ANTI  - Returns rows from LEFT that have NO matching 'id' in RIGHT
left_anti_df = df1.join(df2, on="id", how="left_anti")
left_anti_df.display() # Returns rows that are unique to the left table (not shared with the right table).

# COMMAND ----------

# DBTITLE 1,LEFT SEMI JOIN - Returns rows from LEFT that HAVE a match in RIGHT (no right columns).
left_semi_df = df1.join(df2, on="id", how="left_semi") # Same as inner join, but only returns columns from the left table.
left_semi_df.display()

# COMMAND ----------

# DBTITLE 1,CROSS JOIN - Cartesian product: every row in df1 joins with every row in df2
cross_df = df1.crossJoin(df2) # Avoid performing cross joins on large data sets.
cross_df.display()

# COMMAND ----------

# DBTITLE 1,NULL-SAFE JOIN (eqNullSafe)
# MAGIC %md
# MAGIC ##### Normal join condition ignores null == null, so we use eqNullSafe() or <=> operator.
# MAGIC - NULL = NULL        →  UNKNOWN  (not true)
# MAGIC - NULL != NULL       →  UNKNOWN  (also not true)
# MAGIC
# MAGIC ### Issues with NULL's in the data
# MAGIC - Unexpected missing joins — rows you thought should match (both have NULLs) won’t.
# MAGIC
# MAGIC - Data loss — especially in INNER joins.
# MAGIC
# MAGIC - Duplicate null rows — in OUTER joins.
# MAGIC
# MAGIC - Wrong metrics — counts, aggregations, or mappings may undercount records.

# COMMAND ----------

df_nullsafe = df1.join(df2, on=(df1["id"].eqNullSafe(df2["id"])), how="left")
df_nullsafe.display()

# COMMAND ----------

inner_df.display()

# COMMAND ----------
```
## Pyspark Explode DataBricks
```python
# Databricks notebook source
# MAGIC %md
# MAGIC ## Explode
# MAGIC https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.explode.html

# COMMAND ----------

# MAGIC %md
# MAGIC - when arrays or list are passed into this function, it creates a new row for each element in array. when a map is passed, it creates two new columns for one key and value and each key-value pair splits into a new row.
# MAGIC
# MAGIC ##### variants
# MAGIC - Explode
# MAGIC - Explode_outer
# MAGIC - posexplode (position explode)
# MAGIC - posexplode_outer
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC | Function             | Keeps Null Rows | Adds Position Index | Description                            |
# MAGIC | -------------------- | --------------- | ------------------- | -------------------------------------- |
# MAGIC | `explode()`          | ❌               | ❌                   | Flattens array; drops nulls            |
# MAGIC | `posexplode()`       | ❌               | ✅                   | Flattens array with position           |
# MAGIC | `explode_outer()`    | ✅               | ❌                   | Flattens array; keeps nulls            |
# MAGIC | `posexplode_outer()` | ✅               | ✅                   | Flattens array with position and nulls |
# MAGIC

# COMMAND ----------

import pyspark.sql.functions as F

# COMMAND ----------

data = [
    (1, "CustomerA", [
        {"partkey": 101, "qty": 5, "price": 100},
        {"partkey": 102, "qty": 2, "price": 250}
    ]),
    (2, "CustomerB", [
        {"partkey": 103, "qty": 1, "price": 500}
    ]),
    (3, "CustomerC", None)  # Missing line items
]

orders_df = spark.createDataFrame(data, ["order_id", "customer_name", "line_items"])

orders_df.display()
orders_df.printSchema()


# COMMAND ----------

df_explode = orders_df.select(
    "order_id",
    "customer_name",
    F.explode("line_items").alias("item")
)
df_explode.display()


# COMMAND ----------

# DBTITLE 1,select exploded columns

df_explode.select(
    "order_id",
    "customer_name",
    F.col("item.partkey").alias("partkey"), # Dot notation is used to access the attributes in each item after the explosion (similar to syntax used in snowflake).
    F.col("item.qty").alias("quantity"),
    F.col("item.price").alias("price")
).show()

# COMMAND ----------

df_posexplode = orders_df.select(
    "order_id",
    "customer_name",
    F.posexplode("line_items").alias("pos", "item") # creates an additional 'pos' column defining the zero-based index of each array element.
)
df_posexplode.display()


# COMMAND ----------

df_posexplode.select(
    "order_id",
    "pos",
    F.col("item.partkey").alias("partkey"),
    F.col("item.qty").alias("quantity"),
    F.col("item.price").alias("price")
).display()

# COMMAND ----------

df_explode_outer = orders_df.select(
    "order_id",
    "customer_name",
    F.explode_outer("line_items").alias("item")
)
df_explode_outer.display() # Includes null values.


# COMMAND ----------

df_explode_outer.select(
    "order_id",
    "customer_name",
    F.col("item.partkey").alias("partkey"),
    F.col("item.qty").alias("quantity"),
    F.col("item.price").alias("price")
).display()


# COMMAND ----------

df_posexplode_outer = orders_df.select(
    "order_id",
    "customer_name",
    F.posexplode_outer("line_items").alias("pos", "item")
)
df_posexplode_outer.display()


# COMMAND ----------


df_posexplode_outer.select(
    "order_id",
    "pos",
    "customer_name",
    F.col("item.partkey").alias("partkey"),
    F.col("item.qty").alias("quantity"),
    F.col("item.price").alias("price")
).display()

# COMMAND ----------

data = [
    (1, ["red", "blue", "green"]),
    (2, ["yellow"]),
    (3, []),
    (4, None)
]

df = spark.createDataFrame(data, ["id", "colors"])

# COMMAND ----------

df_exploded = df.select(df.id, F.explode(df.colors).alias("color"))
df_exploded.display()


# COMMAND ----------


df_outer = df.select(df.id, F.explode_outer(df.colors).alias("color"))
df_outer.display()


# COMMAND ----------
```
## PySpark Pivot DataBricks
```python
# Databricks notebook source
# MAGIC %md
# MAGIC Objective
# MAGIC
# MAGIC - Understand how to pivot data — turning rows into columns.
# MAGIC
# MAGIC - Understand how to unpivot (melt) data — turning columns back into rows.
# MAGIC
# MAGIC - Learn real-world analytical use cases using Sales / Orders data inspired by TPC-H.

# COMMAND ----------

# MAGIC %md
# MAGIC In analytics, data often needs reshaping:
# MAGIC
# MAGIC - Pivot: summarize data and transform unique row values into columns.
# MAGIC
# MAGIC - Unpivot (melt): perform the reverse, converting multiple columns into key–value pairs for easier analysis or aggregation.

# COMMAND ----------

data = [
    ("North", "Q1", 12000),
    ("North", "Q2", 15000),
    ("South", "Q1", 10000),
    ("South", "Q2", 9000),
    ("East", "Q1", 8000),
    ("East", "Q2", 9500),
]

sales_df = spark.createDataFrame(data, ["region", "quarter", "revenue"])

sales_df.display()


# COMMAND ----------

# DBTITLE 1,Pivot
pivot_df = (
    sales_df
    .groupBy("region")
    .pivot("quarter")     # column whose values become new column names
    .sum("revenue")       # aggregation function
)

pivot_df.display() # Each row in the pivot table represents the total revenue for each region and quarter.


# COMMAND ----------

# MAGIC %md
# MAGIC stack(n, col_name1, value1, col_name2, value2, …)
# MAGIC
# MAGIC   - n = number of columns to unpivot.
# MAGIC
# MAGIC   - Each pair ('Q1', Q1) creates one row with key–value pairs.
# MAGIC
# MAGIC   - The output aliases define the new columns (here: quarter, revenue).

# COMMAND ----------

# DBTITLE 1,Unpivot
unpivot_df = (
    pivot_df
    .selectExpr(
        "region",
        "stack(2, 'Q1', Q1, 'Q2', Q2) as (quarter, revenue)"
    )
)

unpivot_df.display() # region, quarter, and revenue will be the columns of the un-pivoted table.


# COMMAND ----------

data2 = [
    ("North", "Q1", 12000),
    ("North", "Q2", 15000),
    ("South", "Q1", 10000),
    ("East", "Q2", 9500),
]

sales_df2 = spark.createDataFrame(data2, ["region", "quarter", "revenue"])

sales_df2.groupBy("region").pivot("quarter").sum("revenue").display()


# COMMAND ----------
```
- Performing a pivot on a table is the process of transforming the rows (unique row values) into columns and columns into rows.
- When pivoting in PySpark, you need to specify which column (the unique rows in the column) will be used to create the columns in the pivot table.
- Next, you need to specify how you want to group and aggregate data in the existing table. The grouped (unique) values in the specified column become the rows in the pivot table and the aggregation populates the value of each row.
## DataBricks Project 3
```python
*** DATASET - coffee_data.json***

[
  {
    "coffee": {
      "region": [
        {"id": 1, "name": "John Doe"},
        {"id": 2, "name": "Don Joeh"}
      ],
      "country": {"id": 2, "company": "ACME"}
    },
    "brewing": {
      "region": [
        {"id": 1, "name": "John Doe"},
        {"id": 3, "name": "Jane Smith"}
      ],
      "country": {"id": 2, "company": "ACME"}
    }
  },
  {
    "coffee": {
      "region": [
        {"id": 4, "name": "Anna Lee"}
      ],
      "country": {"id": 5, "company": "CoffeeCorp"}
    },
    "brewing": {
      "region": null,
      "country": {"id": 6, "company": "BrewCo"}
    }
  }
]

#--------------------------------------------------------------------------------------------------------

# Databricks notebook source
# MAGIC %md
# MAGIC Below is a full mini-project that uses your dataset and integrates these functions:
# MAGIC - `explode, explode_outer`
# MAGIC - `union`
# MAGIC - `when (CASE WHEN equivalent)`
# MAGIC - `pivot`

# COMMAND ----------

# MAGIC %md
# MAGIC ## Project Title: Analyzing Coffee Brewing Data from Nested JSON
# MAGIC
# MAGIC #### Objective
# MAGIC Students will learn how to:
# MAGIC - Read and flatten a nested JSON dataset.
# MAGIC - Use explode and explode_outer to handle arrays with and without nulls.
# MAGIC - Combine two DataFrames using union.
# MAGIC - Apply conditional logic using when.
# MAGIC - Reshape the data using pivot.

# COMMAND ----------


from pyspark.sql.functions import explode, explode_outer, col, when


df = spark.read.option("multiline", True).json("/Volumes/workspace/default/external_datasets/coffee_data.json")
df.printSchema()
df.display()


# COMMAND ----------

df.select(col("coffee.region").getItem(0).getField("name")).display()

# COMMAND ----------

# DBTITLE 1,Coffee Regions
# Using explode: ignores nulls
coffee_df = df.select(explode("coffee.region").alias("region"), col("coffee.country.company").alias("company"))
coffee_df.display()

# COMMAND ----------

# DBTITLE 1,Brewing Regions
brewing_df = df.select(explode_outer("brewing.region").alias("region"), col("brewing.country.company").alias("company"))
brewing_df.display()

# COMMAND ----------

# DBTITLE 1,Combine Coffee and Brewing Data using UNION
combined_df = coffee_df.union(brewing_df)
combined_df.display()

# COMMAND ----------

from pyspark.sql.functions import lit

enriched_df = combined_df.withColumn(
    "company_type",
    when(col("company") == "ACME", lit("Local")) # Using 'lit()' here is required. You can't just type the string by itself. 
    .when(col("company") == "BrewCo", lit("Partner"))
    .otherwise(lit("International"))
)
enriched_df.display() # The above statement acts similarly to a case statement. The 'company_name' column will be filled with either 'Local' or 'Partner' depending on the company name.


# COMMAND ----------

# DBTITLE 1,Pivot to Compare Employee Counts per Company
from pyspark.sql.functions import count

pivot_df = (
    enriched_df.groupBy("region.name")
    .pivot("company")
    .agg(count(lit(1)))
)

pivot_df.display()


# COMMAND ----------

df.schema.fields

# COMMAND ----------

def flatten(df):
    """
    Recursively flattens all StructType and ArrayType columns in a DataFrame.
    StructType columns are expanded into individual columns.
    ArrayType columns are exploded into multiple rows.
    """

    from pyspark.sql.types import StructType, ArrayType
    from pyspark.sql.functions import col, explode_outer

    complex_fields = dict([
        (field.name, field.dataType)
        for field in df.schema.fields
        if isinstance(field.dataType, (ArrayType, StructType))
    ])

    while len(complex_fields) != 0:
        col_name = list(complex_fields.keys())[0]
        print(f"Processing: {col_name}, Type: {type(complex_fields[col_name])}")

        # Flatten StructType
        if isinstance(complex_fields[col_name], StructType):
            expanded = [
                col(f"{col_name}.{k.name}").alias(f"{col_name}_{k.name}")
                for k in complex_fields[col_name]
            ]
            df = df.select("*", *expanded).drop(col_name)

        # Explode ArrayType
        elif isinstance(complex_fields[col_name], ArrayType):
            df = df.withColumn(col_name, explode_outer(col_name))

        # Recompute remaining complex fields
        complex_fields = dict([
            (field.name, field.dataType)
            for field in df.schema.fields
            if isinstance(field.dataType, (ArrayType, StructType))
        ])

    return df


# COMMAND ----------

flat_df = flatten(df)
flat_df.printSchema()
flat_df.display()


# COMMAND ----------

# DBTITLE 1,Challenge for Students
# MAGIC %md
# MAGIC - Add a `source` column (“coffee” or “brewing”) before doing the union.
# MAGIC - Write one query showing:
# MAGIC   - Total number of people per company.
# MAGIC   - Number of people unique to each source.
# MAGIC - Experiment replacing `explode` with `explode_outer` and explain the difference.
```
## PySpark Handling Bad Records
```python
# Databricks notebook source
# MAGIC %md
# MAGIC ### Handling Malformed Data in PySpark Using PERMISSIVE, DROPMALFORMED, and FAILFAST Modes
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC Students will learn how Spark handles dirty or inconsistent input data using the three modes of the mode option in file reading:
# MAGIC
# MAGIC - PERMISSIVE (default): Keeps corrupt records in a separate column.
# MAGIC
# MAGIC - DROPMALFORMED: Skips bad records.
# MAGIC
# MAGIC - FAILFAST: Stops execution if any malformed record is found.

# COMMAND ----------

df_permissive = (
    spark.read
    .option("header", True)
    .csv("/Volumes/workspace/default/external_datasets/customer_data.csv")
)

df_permissive.display()
df_permissive.printSchema()


# COMMAND ----------

df_permissive.schema

# COMMAND ----------

from pyspark.sql.types import *
user_def_schema = StructType([StructField('id', StringType(), True),
 StructField('name', StringType(), True),
 StructField('age', StringType(), True),
 StructField('_corrupt_record', StringType(), True)])

# COMMAND ----------

df_permissive = (
    spark.read
    .option("header", True)
    .option("mode", "PERMISSIVE")
    .schema(user_def_schema)
    .csv("/Volumes/workspace/default/external_datasets/customer_data.csv")
)

df_permissive.display()
df_permissive.printSchema()


# COMMAND ----------

df_permissive = (
    spark.read
    .option("header", True)
    .option("mode", "DROPMALFORMED")
    .schema(user_def_schema)
    .csv("/Volumes/workspace/default/external_datasets/customer_data.csv")
)

df_permissive.display()
df_permissive.printSchema()


# COMMAND ----------

df_permissive = (
    spark.read
    .option("header", True)
    .option("mode", "FAILFAST")
    .schema(user_def_schema)
    .csv("/Volumes/workspace/default/external_datasets/customer_data.csv")
)

df_permissive.display()
df_permissive.printSchema()


# COMMAND ----------
```
- naming of the `_corrupt_record` column is important for `PERMISSIVE` record handling. The column **must** be named `_corrupt_record` in order to capture the corrupt record. It records the entire, comma delimited record, not just the non-compliant attribute(s).

## PySpark Optimizations (Reading DAGs)

Concept review: [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]].

```python
# Databricks notebook source
from pyspark.sql.types import *
import pyspark.sql.functions as F

# COMMAND ----------


df_transactions = spark.table('samples.bakehouse.sales_transactions')

# COMMAND ----------

df_transactions.display()

# COMMAND ----------

df_customers = spark.table('samples.bakehouse.sales_customers')

# COMMAND ----------

df_customers.display()

# COMMAND ----------

df_customers.filter("state = 'New York'").display()

# COMMAND ----------

# MAGIC %md
# MAGIC # Spark's Query Plans

# COMMAND ----------

# MAGIC %md
# MAGIC Spark Execution — see [[02_technical_prep/system_design/Data Processing Systems (Spark)]] for the execution model.
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC - https://bigdataperformance.substack.com/p/understanding-apache-sparks-execution
# MAGIC - https://medium.com/@Sanjay007/understanding-the-execution-process-of-apache-spark-4209374febb8
# MAGIC - https://www.analyticsvidhya.com/blog/2021/08/understand-the-internal-working-of-apache-spark/

# COMMAND ----------

# MAGIC %md
# MAGIC # Narrow Transformations
# MAGIC - `filter` rows where `state='New York'`
# MAGIC - `add` a new column: adding `first_name` and `last_name`
# MAGIC - `select` relevant columns

# COMMAND ----------

df_narrow_transform = (
    df_customers
    .filter(F.col("state") == "New York")
    .withColumn("name", F.concat_ws(" ", F.col("first_name"), F.col("last_name")))
    .withColumn("customerID", F.col('customerID').cast("Int") - 1000000)
)

df_narrow_transform.display()
df_narrow_transform.explain(True)

# COMMAND ----------

# MAGIC %md
# MAGIC # Wide Transformations
# MAGIC 1. Repartition
# MAGIC 2. Coalesce
# MAGIC 3. Joins
# MAGIC 4. GroupBy
# MAGIC    - `count`
# MAGIC    - `countDistinct`
# MAGIC    - `sum`

# COMMAND ----------

# MAGIC %md
# MAGIC ## 1. Repartition

# COMMAND ----------

df_transactions.repartition(24).explain(True) # Divides data evenly amongst specified number of partitions.

# COMMAND ----------

# MAGIC %md
# MAGIC ## 2. Coalesce

# COMMAND ----------

df_transactions.coalesce(5).explain(True) # Used to reduce the number of partitions data is spread across.

# COMMAND ----------

# MAGIC %md
# MAGIC ## 3. Joins

# COMMAND ----------

spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# COMMAND ----------

df_narrow_transform.count()

# COMMAND ----------

df_transactions.count()

# COMMAND ----------

df_customers = (df_customers.withColumn("customerID", F.col('customerID').cast("Int") - 1000000))

# COMMAND ----------

df_joined = (
    df_transactions.join(
        df_customers,
        how="inner",
        on="customerID"
    )
)

# COMMAND ----------

df_joined.count()

# COMMAND ----------

df_joined.display()

# COMMAND ----------

df_joined.explain(True)

# COMMAND ----------

# MAGIC %md
# MAGIC ## 4. GroupBy

# COMMAND ----------

df_transactions.printSchema()

# COMMAND ----------

# MAGIC %md
# MAGIC ### GroupBy Count

# COMMAND ----------

df_state_counts = (
    df_joined
    .groupBy("state")
    .count()
)

# COMMAND ----------

df_state_counts.explain(True)

# COMMAND ----------

df_txn_amt_city = (
    df_joined
    .groupBy("state")
    .agg(F.sum("totalPrice").alias("txn_amt"))
)

# COMMAND ----------

df_txn_amt_city.explain(True)

# COMMAND ----------

# MAGIC %md
# MAGIC ### GroupBy Count Distinct 

# COMMAND ----------

df_txn_per_state = (
    df_joined
    .groupBy("customerID")
    .agg(F.countDistinct("state").alias("city_count"))
)

# COMMAND ----------

df_joined.display()

# COMMAND ----------

df_txn_per_state.display()
df_txn_per_state.explain(True)

# COMMAND ----------

# MAGIC %md
# MAGIC # 5. Interesting Observations

# COMMAND ----------

df_customer_gt_50 = (
    df_customers
    .filter(F.col("age").cast("int") > 50)
)
df_customer_gt_50.show(5, False)
df_customer_gt_50.explain(True)

# COMMAND ----------
```

## Understanding Spark UI For PySpark and DataBricks

Concept review: [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]].

```python
# Databricks notebook source
# MAGIC %md
# MAGIC # Imports & Configuration

# COMMAND ----------

from pyspark.sql.types import *
import pyspark.sql.functions as F

# COMMAND ----------


df_transactions = spark.table('samples.bakehouse.sales_transactions') # Will not create a job to run this command because no action was called upon.

# COMMAND ----------

# DBTITLE 1,reading file
source_file_path = '/Volumes/workspace/default/external_datasets/sampledata_parquet.parquet'
df = spark.read.parquet(source_file_path) # Creates a job to read file metadata.

# COMMAND ----------

df_transactions.display()

# COMMAND ----------

df_customers = spark.table('samples.bakehouse.sales_customers')

# COMMAND ----------

df_customers.display()

# COMMAND ----------

# MAGIC %md
# MAGIC # Narrow Transformations
# MAGIC - `filter` rows where `city='boston'`
# MAGIC - `add` a new column: adding `first_name` and `last_name`
# MAGIC - `alter` an exisitng column: adding 5 to `age` column
# MAGIC - `select` relevant columns

# COMMAND ----------

df_narrow_transform = (
    df_customers
    .filter(F.col("state") == "New York")
    .withColumn("name", F.concat_ws(" ", F.col("first_name"), F.col("last_name")))
    .withColumn("customerID", F.col('customerID').cast("Int") - 1000000)
    .select("customerID", "first_name", "last_name", "email_address", "phone_number", "address", "city", "state")
)


df_narrow_transform.write.format("noop").mode("overwrite").save("/Volumes/workspace/default/external_datasets/customer_data")

# COMMAND ----------

df_narrow_transform.display()

# COMMAND ----------

df_customer_gt_50 = (
    df_customers
    .withColumn("customerID", F.col('customerID').cast("Int") - 1000000)
    .filter(F.col("customerID") < 1000010)
)
df_customer_gt_50.write.format("noop").mode("overwrite").save("/Volumes/workspace/default/external_datasets/customer_data")

# COMMAND ----------

df_customer_gt_50.display()

# COMMAND ----------

# MAGIC %md
# MAGIC ## 1. Joins

# COMMAND ----------

# MAGIC %md
# MAGIC ### Sort Merge Join

# COMMAND ----------

df_customers = (
    df_customers
    .withColumn("customerID", F.col('customerID').cast("Int") - 1000000)
)

# COMMAND ----------

spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# COMMAND ----------

df_joined = (
    df_transactions.join(
        df_customers,
        how="inner",
        on="customerID"
    )
)

# COMMAND ----------

df_joined.explain(True)

# COMMAND ----------

df_joined.write.format("noop").mode("overwrite").save("/Volumes/workspace/default/external_datasets/customer_data")

# COMMAND ----------

# MAGIC %md
# MAGIC ### Broadcast Join

# COMMAND ----------

spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)

# COMMAND ----------

df_broadcast_joined = (
    df_transactions.join(
        F.broadcast(df_customers),
        how="inner",
        on="customerID"
    )
)

# COMMAND ----------

df_broadcast_joined.write.format("noop").mode("overwrite").save("/Volumes/workspace/default/external_datasets/customer_data")

# COMMAND ----------

# MAGIC %md
# MAGIC ## 2. GroupBy

# COMMAND ----------

# MAGIC %md
# MAGIC ### GroupBy Count

# COMMAND ----------

df_transactions.printSchema()

# COMMAND ----------

df_city_counts = (
    df_customers
    .groupBy("state")
    .count()
)

# COMMAND ----------

df_city_counts.display()

# COMMAND ----------

df_txn_amt_city = (
    df_broadcast_joined
    .groupBy("state")
    .agg(F.sum("totalPrice").alias("txn_amt"))
)

# COMMAND ----------

df_txn_amt_city.display()

# COMMAND ----------

# MAGIC %md
# MAGIC ### GroupBy Count Distinct 

# COMMAND ----------

df_txn_per_state = (
    df_joined
    .groupBy("state")
    .agg(F.countDistinct("customerID").alias("city_count"))
)

# COMMAND ----------

df_txn_per_state.display()

# COMMAND ----------
```

## PySpark Cache, Persist, & Storage Levels in DataBricks

Concept review: [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]].

```python
# Databricks notebook source
# MAGIC %md
# MAGIC # Imports & Configuration

# COMMAND ----------

from pyspark.sql.types import *
import pyspark.sql.functions as F

# COMMAND ----------


df_transactions = spark.table('samples.bakehouse.sales_transactions')

# COMMAND ----------

# DBTITLE 1,reading file
source_file_path = '/Volumes/workspace/default/external_datasets/sampledata_parquet.parquet'
df = spark.read.parquet(source_file_path)

# COMMAND ----------

df_transactions.display()

# COMMAND ----------

df_customers = spark.table('samples.bakehouse.sales_customers')

# COMMAND ----------

df_customers.display()

# COMMAND ----------

df_narrow_transform = (
    df_customers
    .filter(F.col("state") == "New York")
    .withColumn("name", F.concat_ws(" ", F.col("first_name"), F.col("last_name")))
    .withColumn("customerID", F.col('customerID').cast("Int") - 1000000)
    .select("customerID", "first_name", "last_name", "email_address", "phone_number", "address", "city", "state")
)


# COMMAND ----------

df_narrow_transform.display()

# COMMAND ----------

# MAGIC %md
# MAGIC ### Broadcast Join

# COMMAND ----------

spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 10485760)

# COMMAND ----------

df_broadcast_joined = (
    df_transactions.join(
        F.broadcast(df_narrow_transform),
        how="inner",
        on="customerID"
    )
)

# COMMAND ----------


df_broadcast_joined.write.format("noop").mode("overwrite").save("/Volumes/workspace/default/external_datasets/customer_data")

# COMMAND ----------

df_broadcast_joined.explain(True)

# COMMAND ----------

# MAGIC %md
# MAGIC ## 2. GroupBy

# COMMAND ----------

# MAGIC %md
# MAGIC ### GroupBy Count

# COMMAND ----------

df_transactions.printSchema()

# COMMAND ----------

df_broadcast_joined.cache().count() # Allows spark to re-use this dataframe in subsequent operations without needing to fetch the data again.

# COMMAND ----------

df_city_counts = (
    df_broadcast_joined
    .groupBy("state")
    .count()
)

# COMMAND ----------

df_city_counts.explain(True)

# COMMAND ----------

df_city_counts.display()

# COMMAND ----------

df_txn_amt_city = (
    df_broadcast_joined
    .groupBy("state")
    .agg(F.sum("totalPrice").alias("txn_amt"))
)

# COMMAND ----------

df_txn_amt_city.explain(True)

# COMMAND ----------

df_txn_amt_city.display()

# COMMAND ----------

# MAGIC %md
# MAGIC ## `StorageLevel` Types:
# MAGIC
# MAGIC
# MAGIC - `DISK_ONLY`: CPU efficient, memory efficient, slow to access, data is serialized when stored on disk
# MAGIC - `DISK_ONLY_2`: disk only, replicated 2x
# MAGIC - `DISK_ONLY_3`: disk only, replicated 3x
# MAGIC
# MAGIC - `MEMORY_AND_DISK`: spills to disk if there's no space in memory
# MAGIC - `MEMORY_AND_DISK_2`: memory and disk, replicated 2x
# MAGIC - `MEMORY_AND_DISK_DESER`(default): same as `MEMORY_AND_DISK`, deserialized in both for fast access
# MAGIC
# MAGIC - `MEMORY_ONLY`: CPU efficient, memory intensive
# MAGIC - `MEMORY_ONLY_2`: memory only, replicated 2x - for resilience, if one executor fails
# MAGIC
# MAGIC **Note**: 
# MAGIC - `SER` is CPU intensive, memory saving as data is compact while `DESER` is CPU efficient, memory intensive
# MAGIC - Size of data on disk is lesser as data is in serialized format, while deserialized in memory as JVM objects for faster access
# MAGIC
# MAGIC  
# MAGIC ```

# COMMAND ----------

# MAGIC %md
# MAGIC | Storage Level | useDisk | useMemory | useOffHeap | Deserialized | Replication | Description |
# MAGIC | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
# MAGIC | **DISK_ONLY** | True | False | False | False | 1 | Stores the RDD/DataFrame only on disk. |
# MAGIC | **DISK_ONLY_2** | True | False | False | False | 2 | Same as DISK_ONLY, but replicates each partition to two cluster nodes. |
# MAGIC | **DISK_ONLY_3** | True | False | False | False | 3 | Same as DISK_ONLY, but replicates each partition to three cluster nodes. |
# MAGIC | **MEMORY_AND_DISK** | True | True | False | True | 1 | Stores in memory; spills to disk if memory is full. |
# MAGIC | **MEMORY_AND_DISK_2** | True | True | False | True | 2 | Same as MEMORY_AND_DISK, but replicates each partition to two cluster nodes. |
# MAGIC | **MEMORY_AND_DISK_DESER**| True | True | False | True | 1 | Stores in memory (deserialized); spills to disk if full. |
# MAGIC | **MEMORY_ONLY** | False | True | False | True | 1 | Stores in memory only. If it doesn't fit, it is not cached. |
# MAGIC | **MEMORY_ONLY_2** | False | True | False | True | 2 | Same as MEMORY_ONLY, but replicates each partition to two cluster nodes. |
# MAGIC | **OFF_HEAP** | True | True | True | False | 1 | Stores data in off-heap memory (experimental/external block store). |
# MAGIC | **NONE** | False | False | False | False | 1 | No persistence (data is not cached). |

# COMMAND ----------

from pyspark.storagelevel import StorageLevel

# COMMAND ----------

df_broadcast_joined.unpersist() # Removes from memory.
df_broadcast_joined.persist(StorageLevel.DISK_ONLY_3)

df_txn_amt_city = (
    df_broadcast_joined
    .groupBy("state")
    .agg(F.sum("totalPrice").alias("txn_amt"))
)

# COMMAND ----------

df_txn_amt_city.display()

# COMMAND ----------
```

## PySpark Skew Data & Salting Techniques in Databricks

Concept review: [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]].

```python
# Databricks notebook source
# MAGIC %md
# MAGIC <h2> Imports & Configuration </h2>

# COMMAND ----------

from pyspark.sql.types import *
import pyspark.sql.functions as F


# COMMAND ----------

spark.conf.set("spark.sql.shuffle.partitions", "3")
spark.conf.get("spark.sql.shuffle.partitions")
spark.conf.set("spark.sql.adaptive.enabled", "false")

# COMMAND ----------

# MAGIC %md
# MAGIC <h2> Simulating Skewed Join </h2>

# COMMAND ----------

df_uniform = spark.createDataFrame([i for i in range(1000000)], IntegerType())
df_uniform.display()

# COMMAND ----------

(
    df_uniform
    .withColumn("partition", F.spark_partition_id())
    .groupBy("partition")
    .count()
    .orderBy("partition")
    .display()
)

# COMMAND ----------

df0 = spark.createDataFrame([0] * 999990, IntegerType()).repartition(1)
df1 = spark.createDataFrame([1] * 15, IntegerType()).repartition(1)
df2 = spark.createDataFrame([2] * 10, IntegerType()).repartition(1)
df3 = spark.createDataFrame([3] * 5, IntegerType()).repartition(1)
df_skew = df0.union(df1).union(df2).union(df3)
df_skew.display()

# COMMAND ----------

(
    df_skew
    .withColumn("partition", F.spark_partition_id())
    .groupBy("partition")
    .count()
    .orderBy("partition")
    .display()
)

# COMMAND ----------

df_joined_c1 = df_skew.join(df_uniform, "value", 'inner')

# COMMAND ----------

(
    df_joined_c1
    .withColumn("partition", F.spark_partition_id())
    .groupBy("partition")
    .count()
    .display()
)

# COMMAND ----------

# MAGIC %md
# MAGIC Simulating Uniform Distribution Through Salting

# COMMAND ----------

SALT_NUMBER = int(spark.conf.get("spark.sql.shuffle.partitions"))
SALT_NUMBER

# COMMAND ----------

df_skew = df_skew.withColumn("salt", (F.rand() * SALT_NUMBER).cast("int"))

# COMMAND ----------

df_skew.display()

# COMMAND ----------

df_uniform = (
    df_uniform
    .withColumn("salt_values", F.array([F.lit(i) for i in range(SALT_NUMBER)]))
    .withColumn("salt", F.explode(F.col("salt_values")))
)

# COMMAND ----------

df_uniform.display()

# COMMAND ----------

df_joined = df_skew.join(df_uniform, ["value", "salt"], 'inner')

# COMMAND ----------

(
    df_joined
    .withColumn("partition", F.spark_partition_id())
    .groupBy("value", "partition")
    .count()
    .orderBy("value", "partition")
    .display()
)

# COMMAND ----------

# MAGIC %md
# MAGIC # Salting In Aggregations

# COMMAND ----------

df_skew.groupBy("value").count().display()

# COMMAND ----------

(
    df_skew
    .withColumn("salt", (F.rand() * SALT_NUMBER).cast("int"))
    .groupBy("value", "salt")
    .agg(F.count("value").alias("count"))
    .groupBy("value")
    .agg(F.sum("count").alias("count"))
    .display()
)
```

## PySpark UDF & Vectorized UDFs in Databricks

Concept review: [[02_technical_prep/Spark Optimization Review|Spark Optimization Review]].

```sql
# Databricks notebook source
# MAGIC %md
# MAGIC # PySpark UDFs, Pandas UDFs (Vectorized UDFs)

# COMMAND ----------

# MAGIC %md
# MAGIC
# MAGIC | Feature            | PySpark Standard UDF | Pandas UDF (Vectorized) | mapPartitions                        | foreachPartition                                 |
# MAGIC |--------------------|---------------------|------------------------|--------------------------------------|--------------------------------------------------|
# MAGIC | Execution Style    | Row-by-Row          | Batch/Vectorized       | Partition-by-Partition               | Partition-by-Partition                           |
# MAGIC | Input to Function  | Single PySpark column value | Pandas Series/DataFrame (Batch) | Iterator of RDD/DataFrame rows      | Iterator of RDD/DataFrame rows                   |
# MAGIC | Output Type        | Returns a new column value | Returns a new Pandas Series/DataFrame | New RDD/DataFrame (Transformation) | No return value (Action)                         |
# MAGIC | Data Transfer      | High Serialization/Deserialization (JVM ↔ Python) for every row. | Reduced overhead using Apache Arrow for efficient batch transfer. | Low overhead; processes an entire partition (batch) in the Python worker. | Low overhead; executes side-effect logic on an entire partition. |
# MAGIC | Performance        | Slowest (Use as a last resort) | Fast (Recommended for custom logic) | "Fastest for external I/O (e.g., API calls)" | Not for generating data; efficient for side effects. |
# MAGIC | Use Case           | "Simple, non-critical logic where no built-in Spark function exists." | "Complex math, string ops, or ML inference that benefit from Pandas/NumPy vectorization." | Connecting to a database or calling an external API once per partition. | "Writing results to a database or Kafka topic, initializing a connection once per executor." |
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC ## PySpark Standard UDFs

# COMMAND ----------

# MAGIC %md
# MAGIC A PySpark Standard UDF is a Python function registered with Spark to operate on a DataFrame.
# MAGIC
# MAGIC Specifications
# MAGIC Operation: Processes data row-by-row.
# MAGIC
# MAGIC Overhead: Very high due to the constant round-trip (Serialization/Deserialization) of data between the JVM (Spark) and the Python Interpreter (Worker) for each row.
# MAGIC
# MAGIC Syntax: Uses @udf(returnType=...) or udf(func, returnType).
# MAGIC
# MAGIC Best Practice: AVOID this method, especially on large datasets. Always check for a built-in Spark function first.

# COMMAND ----------

df = spark.table('samples.bakehouse.sales_customers')

# COMMAND ----------

# You need a simple function to prefix a column value if a certain condition is met, and a built-in function isn't suitable.

from pyspark.sql.functions import udf, col
from pyspark.sql.types import StringType

# 1. Define the Python function
def add_prefix(Fname, Lname):
  if (Fname and Lname) :
    return "User_" + Fname + Lname
  return None

# 2. Register the UDF with a return type
prefix_udf = udf(add_prefix, StringType())

# 3. Apply the UDF (Slower row-by-row execution)
df = df.withColumn("Prefixed_Name", prefix_udf(col("first_name"), col("last_name")))

# COMMAND ----------

df.select('first_name','last_name','Prefixed_Name').display()

# COMMAND ----------

# Register BOTH for DataFrame API and SQL
spark.udf.register("prefix_name", add_prefix, StringType())
# Create a temp view
df.createOrReplaceTempView("raw_users")
# Run sql on the view
result = spark.sql("""
    SELECT 
        first_name,
        last_name,
        prefix_name(first_name, last_name) AS prefixed_name
    FROM raw_users
""")

result.display()

# COMMAND ----------

# MAGIC %md
# MAGIC ## Pandas UDFs (Vectorized UDFs)

# COMMAND ----------

# MAGIC %md
# MAGIC Pandas UDFs, also known as Vectorized UDFs, leverage Apache Arrow to pass data as batches (Pandas Series/DataFrames) between the JVM and Python. This significantly reduces the serialization overhead, making them much faster than standard UDFs
# MAGIC
# MAGIC Specifications
# MAGIC - Operation: Processes data batch-by-batch (vectorized).
# MAGIC - Technology: Uses Apache Arrow for efficient, columnar data transfer.
# MAGIC - Input/Output: Functions accept and return Pandas Series or DataFrames.
# MAGIC - Syntax: Uses @pandas_udf(returnType=...). You must specify the return data type.
# MAGIC - Types:
# MAGIC   - Scalar: (Series $\to$ Series) Works like withColumn on a column.
# MAGIC   - Grouped Map: (DataFrame $\to$ DataFrame) Works like a groupBy().apply() on an entire group.
# MAGIC   - Grouped Aggregate: (Series $\to$ Scalar) Performs an aggregation within groups.
# MAGIC - Best Practice: This is the preferred way to implement custom Python logic in PySpark when a built-in function is not available.

# COMMAND ----------

import pandas as pd
from pyspark.sql import SparkSession
from pyspark.sql.functions import pandas_udf, col
from pyspark.sql.types import DoubleType, StructType, StructField, IntegerType

# COMMAND ----------


schema = StructType([
    StructField("User_ID", IntegerType(), False),
    StructField("Activity_Level", DoubleType(), False),
    StructField("Total_Purchases", DoubleType(), False)
])

data = [
    (101, 5.0, 100.0),
    (102, 1.5, 500.0),
    (103, 8.0, 50.0),
    (104, 3.2, 750.0),
    (105, 9.1, 120.0)
]

df = spark.createDataFrame(data, schema)

df.display()


# COMMAND ----------

@pandas_udf(DoubleType(), functionType=PandasUDFType.SCALAR) # The return type is a Double (for the Loyalty Score)
def calculate_loyalty_score(activity: pd.Series, purchases: pd.Series) -> pd.Series:
    """
    Calculates the Loyalty Score using vectorized operations.
    Input: Two Pandas Series (one for each input column batch).
    Output: One Pandas Series (the calculated score batch).
    """
    # Use vectorized arithmetic (NumPy/Pandas) for fast batch processing
    return (activity ** 2 * 0.5) + (purchases * 1.2)

result_df = df.withColumn(
    "Loyalty_Score",
    calculate_loyalty_score(col("Activity_Level"), col("Total_Purchases"))
)

print("--- Result with Loyalty Score ---")
result_df.display()

# COMMAND ----------

data = [
    ("C001", 5, 200.0),
    ("C001", 3, 150.0),
    ("C002", 2, 120.0),
    ("C002", 4, 180.0),
    ("C002", 1,  90.0)
]

schema = StructType([
    StructField("customer_id", StringType()),
    StructField("visits", IntegerType()),
    StructField("amount_spent", DoubleType())
])

df = spark.createDataFrame(data, schema)
df.display()


# COMMAND ----------

from pyspark.sql.functions import pandas_udf, PandasUDFType

# Output schema of the UDF
result_schema = StructType([
    StructField("customer_id", StringType()),
    StructField("total_visits", IntegerType()),
    StructField("total_spent", DoubleType()),
    StructField("avg_transaction", DoubleType()),
])

@pandas_udf(result_schema, functionType=PandasUDFType.GROUPED_MAP)
def customer_stats(pdf: pd.DataFrame) -> pd.DataFrame:
    """
    pdf : Pandas DataFrame containing each group (customer)
    """

    return pd.DataFrame([{
        "customer_id": pdf["customer_id"].iloc[0],
        "total_visits": pdf["visits"].sum(),
        "total_spent": pdf["amount_spent"].sum(),
        "avg_transaction": pdf["amount_spent"].mean()
    }])


# COMMAND ----------

result_df = df.groupBy("customer_id").apply(customer_stats)
result_df.display()

# COMMAND ----------

# MAGIC %md
# MAGIC ## Pandas API on Spark: Apply and Transform Functions

# COMMAND ----------

# MAGIC %md
# MAGIC The Pandas API on Spark allows you to use familiar Pandas syntax to run code on a distributed Spark cluster. Within this API, applyInPandas and transform are high-level, vectorized ways to implement custom logic.

# COMMAND ----------

# MAGIC %md
# MAGIC GroupedData.applyInPandas(func, schema)
# MAGIC
# MAGIC - This function is the most powerful and flexible method for applying a complex custom transformation to grouped data. It's the vectorized, distributed equivalent of a Pandas groupby().apply().
# MAGIC
# MAGIC

# COMMAND ----------

# MAGIC %md
# MAGIC  | Feature                | GroupedData.applyInPandas (Grouped Map)                                                                                   |
# MAGIC  |------------------------|---------------------------------------------------------------------------------------------------------------------------|
# MAGIC  | Input to Function      | A Pandas DataFrame containing all rows for a single group.                                                                |
# MAGIC  | Output from Function   | A Pandas DataFrame (with a schema defined by the user).                                                                   |
# MAGIC  | Transformation Type    | Grouped Transformation. It requires a shuffle to bring all rows of the same group onto a single executor.                 |
# MAGIC  | Output Length          | Arbitrary Length. The output DataFrame can have a different number of rows than the input group. This is the key difference from a Grouped Map Pandas UDF. |
# MAGIC  | Use Case               | Custom Group-Level Processing: Calculating group-specific metrics (e.g., z-score normalization within a group), training a separate ML model per group (e.g., forecasting for each store), or complex feature engineering that requires all group data. |
# MAGIC  | Alternative            | This is typically an alternative to using a combination of window functions and complex aggregations.                     |

# COMMAND ----------

import pandas as pd
from pyspark.sql.functions import col
from pyspark.sql.types import StructType, StructField, StringType, DoubleType,FloatType

# COMMAND ----------

# Sample data
data = [
    ("device_1", 23.5),
    ("device_1", 24.1),
    ("device_1", 22.8),
    ("device_2", 30.0),
    ("device_2", 29.4),
    ("device_3", 40.2),
    ("device_3", 41.1),
    ("device_3", 39.9)
]

# Define schema
schema = StructType([
    StructField("device_id", StringType(), True),
    StructField("reading", FloatType(), True)
])

# Create DataFrame
sensor_df = spark.createDataFrame(data, schema)

sensor_df.display()

# COMMAND ----------

# You have sensor readings from thousands of devices and need to normalize the reading for each device based on its own mean and standard deviation.
result_schema = StructType([
    StructField("device_id", StringType(), True),
    StructField("reading", DoubleType(), True),
    StructField("Z_Score", DoubleType(), True)
])
# Define the Pandas function (operates on one group's data)

def normalize_group(pdf: pd.DataFrame) -> pd.DataFrame:
    # pdf is a Pandas DataFrame for a single Device_ID
    mean_val = pdf['reading'].mean()
    std_val = pdf['reading'].std()
    
    # Calculate the Z-score (standardization)
    pdf['Z_Score'] = (pdf['reading'] - mean_val) / std_val
    return pdf


# 3. Apply the transformation
normalized_df = (
    sensor_df
    .groupBy("device_id") # Group the data
    .applyInPandas(normalize_group, schema=result_schema) # Apply the function
)

# COMMAND ----------

normalized_df.display()

# COMMAND ----------

data = [
    ("StoreA", "ProductX", 100.0),
    ("StoreA", "ProductY", 500.0),
    ("StoreA", "ProductX", 200.0), # ProductX total: 300.0
    ("StoreB", "ProductZ", 800.0),
    ("StoreB", "ProductZ", 100.0), # ProductZ total: 900.0
    ("StoreB", "ProductA", 1500.0), # ProductA total: 1500.0 (The winner for StoreB)
    ("StoreC", "ProductK", 10.0)
]

schema = StructType([
    StructField("Store_ID", StringType(), True),
    StructField("Product_Name", StringType(), True),
    StructField("Revenue", DoubleType(), True)
])

df = spark.createDataFrame(data, schema)
df.display()

# COMMAND ----------

# Define the Function that accepts and returns a WHOLE Pandas DataFrame
def calculate_top_product(pdf: pd.DataFrame) -> pd.DataFrame:
    """
    This function receives a Pandas DataFrame containing ALL rows for one group (e.g., 'StoreA').
    It returns a Pandas DataFrame with the top product for that group.
    """
    # Calculate total revenue for each product in the group
    grouped_revenue = pdf.groupby('Product_Name').agg(
        Total_Revenue=('Revenue', 'sum')
    ).reset_index()
    
    # Find the product with the maximum total revenue
    top_product = grouped_revenue.loc[grouped_revenue['Total_Revenue'].idxmax()]
    
    # Create a final Pandas DataFrame to return (must include the group key)
    result_df = pd.DataFrame({
        'Store_ID': [pdf['Store_ID'].iloc[0]], # Get the Store_ID from the input
        'Top_Product': [top_product['Product_Name']],
        'Max_Revenue': [top_product['Total_Revenue']]
    })
    
    return result_df

# The schema must match the structure of the Pandas DataFrame returned by the function.
output_schema = StructType([
    StructField("Store_ID", StringType(), True),
    StructField("Top_Product", StringType(), True),
    StructField("Max_Revenue", DoubleType(), True)
])


result_df = (
    df
    .groupBy("Store_ID") # The grouping key
    .applyInPandas(calculate_top_product, schema=output_schema)
)
# 

# Show the Result
print("--- Top Product per Store (Result) ---")
result_df.display()


# COMMAND ----------

# MAGIC %md
# MAGIC DataFrame.transform(func)
# MAGIC
# MAGIC - The transform function is a helper method on a DataFrame that allows you to chain and reuse complex DataFrame transformations. It is not a data transformation itself, but a code transformation wrapper.

# COMMAND ----------

# MAGIC %md
# MAGIC  | Feature                | DataFrame.transform                                                                 |
# MAGIC  |------------------------|------------------------------------------------------------------------------------|
# MAGIC  | Input to Function      | A PySpark DataFrame.                                                               |
# MAGIC  | Output from Function   | A PySpark DataFrame.                                                               |
# MAGIC  | Transformation Type    | Wrapper/Utility. It enables cleaner, reusable, and more modular code.              |
# MAGIC  | Output Length          | Same as input length (It's typically used for transformations like adding/renaming columns). |
# MAGIC  | Use Case               | Modularizing ETL: Creating reusable functions for cleaning, feature creation, or schema refinement that you can apply across multiple DataFrames in a pipeline. |
# MAGIC  | Alternative            | Writing a long chain of .withColumn(...) calls.                                    |

# COMMAND ----------

from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, IntegerType, StringType
from pyspark.sql.functions import current_timestamp, sha2

# COMMAND ----------


user_data = [
    (1, "Alice"),
    (2, "Bob"),
    (3, "Charlie"),
    (4, "David"),
    (5, "Emma")
]

user_schema = StructType([
    StructField("ID", IntegerType(), True),
    StructField("Value", StringType(), True)
])

raw_user_df = spark.createDataFrame(user_data, user_schema)


raw_user_df.display()

# COMMAND ----------

# ==== CREATE SAMPLE RAW LOGS ====
log_data = [
    (100, "Login"),
    (101, "ViewPage"),
    (102, "ClickButton"),
    (103, "Logout"),
    (104, "Login"),
    (105, "AddToCart")
]

log_schema = StructType([
    StructField("ID", IntegerType(), True),
    StructField("Value", StringType(), True)
])

raw_logs = spark.createDataFrame(log_data, log_schema)

raw_logs.display()

# COMMAND ----------

# You need a single function to perform a sequence of standard feature engineering steps (like casting types and adding a timestamp column) that must be applied to several different raw tables
from pyspark.sql import DataFrame
from pyspark.sql.functions import current_timestamp, sha2, concat


def standard_prep(df: DataFrame) -> DataFrame:
    # Example 1: Add a Processing Timestamp
    df = df.withColumn("Proc_TS", current_timestamp())
    # Example 2: Create a unique row hash for change detection
    df = df.withColumn(
        "Row_Hash",
        sha2(concat(df["ID"].cast("string"), df["Value"].cast("string")), 256)
    )
    return df

raw_user_data = raw_user_df
clean_user_data = raw_user_data.transform(standard_prep)

raw_log_data = raw_logs
clean_log_data = raw_log_data.transform(standard_prep)

# COMMAND ----------

clean_user_data.display()
clean_log_data.display()

# COMMAND ----------

# MAGIC %md
# MAGIC | Feature                        | GroupedData.applyInPandas(func, schema)                                                                 | DataFrame.transform(func)                                      |
# MAGIC |---------------------------------|--------------------------------------------------------------------------------------------------------|----------------------------------------------------------------|
# MAGIC | Primary Goal                   | Data Transformation (Custom, group-level logic)                                                         | Code Transformation (Chaining, code reuse/modularity)          |
# MAGIC | Scope                          | Operates on grouped data (e.g., all rows for Store_ID = 'A').                                          | Operates on the entire DataFrame.                              |
# MAGIC | Input to Function              | A Pandas DataFrame (containing the data for one group).                                                | A PySpark DataFrame.                                           |
# MAGIC | Output from Function           | A Pandas DataFrame.                                                                                    | A PySpark DataFrame.                                           |
# MAGIC | Distribution                   | Requires a shuffle to bring all rows of a group to one worker.                                         | No inherent shuffle; operates on partitions sequentially unless an internal function (like groupBy) forces one. |
# MAGIC | Output Row Count               | Can produce an arbitrary number of rows (e.g., 100 input rows can yield 1 output row, or 500 output rows). | Preserves the row count (typically used to add/modify columns).|
# MAGIC | Schema Requirement             | Mandatory output schema definition is required.                                                        | Schema is inferred by the Spark operations within the function.|
# MAGIC | Use Case                       | Group-Level Analytics/ML: Normalizing features per group, training models per store, calculating custom statistics that require the full context of the group. | Pipeline Building: Reusable feature engineering steps, custom ETL functions, chaining complex operations cleanly. |

# COMMAND ----------

# MAGIC %md
# MAGIC ## Key Difference Summary
# MAGIC - applyInPandas `is for what you do with the data (complex, group-specific, vectorized calculation)`.
# MAGIC
# MAGIC - transform `is for how you structure your PySpark code (clean, modular, reusable pipeline steps)`.
```
