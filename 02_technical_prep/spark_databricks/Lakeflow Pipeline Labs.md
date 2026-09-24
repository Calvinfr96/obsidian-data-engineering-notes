---
aliases:
  - "databricks_lakeflow"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Spark and DataBricks/databricks_lakeflow.md"
imported: 2026-09-24
---

# Lakeflow Pipeline Labs

Review: [[02_technical_prep/Databricks Review|Databricks Review]]

## Contents

- [[#Introduction to LakeFlow Spark Declarative Pipelines (SDP)]]
- [[#Advantages and Disadvantages of LakeFlow Declarative Pipelines]]
- [[#Pipeline Setup Demo using LakeFlow Spark]]
- [[#Initial Setup and User Interface Walkthrough]]
- [[#Materialized View, Temporary View, and Streaming Table in LakeFlow Declarative Pipelines]]
- [[#Building Your First LakeFlow Declarative Pipeline]]
- [[#Sales Data Warehouse (DBW) Introduction using LakeFlow]]
- [[#Data Ingestion to Bronze Layer using LakeFlow Pipelines]]
- [[#Data Quality Expectations in LakeFlow Declarative Pipelines]]
- [[#Auto Change Data Capture (CDC) and Slowly Changing Dimension (SCD) Type 2 – Silver Layer]]
- [[#Sink Configuration in LakeFlow Declarative Pipelines]]
- [[#Supporting SQL]]

## Introduction to LakeFlow Spark Declarative Pipelines (SDP)
- SDP is a framework for building data pipelines where you declare the desired state and transformations of your data using SQL or Python code, and the platform automatically manages the execution, orchestration, and infrastructure. This approach simplifies data engineering by reducing boilerplate code and operational overhead compared to traditional procedural methods.
  - You tell the framework what to do with the data and tasks such as optimization, partitioning, joining, etc. is handled automatically by data bricks.
  - DataBricks Documentation: https://www.databricks.com/product/data-engineering/spark-declarative-pipelines
  - LakeFlow Declarative Pipeline Documentation: https://docs.databricks.com/aws/en/ldp
  - The pipelines created using this framework are capable of handling both batch and stream data processing.
- Important concepts:
  - Flow:
    - The fundamental data processing concept in SDP which supports batch and stream semantics.
    - Reads data from the source, applies user-defined transformations, and writes the result to a target.
    - Multiple flows can exist in a single pipeline.
  - Streaming Table:
    - An append-only delta table with additional support for streaming or incremental data processing.
    - Can be targeted by one or more flows in a pipeline.
  - Materialized View:
    - Results of a query that can be accessed in the same manner as a table.
    - Caches the results of a query and refreshes them on a specified interval.
  - Standard View / Temporary View:
    - Logical query that is executed each time a flow is triggered.
    - Slower than materialized views. 
  - Sink:
    - Pipelines provide functionality that allow you to write data to a wide range of sinks (data sets), or programmatically transform and stream data to a target(s) that you can write to with python.
    - Sinks act as a destination for data after it has been consumed from a source and transformed. A sink is capable of writing data to a different storage type or the same storage type.

## Advantages and Disadvantages of LakeFlow Declarative Pipelines
- Advantages:
  - Easier Development - Abstracts optimization. orchestration, and dependency-management tasks associated with building data pipelines. You simply tell it how the data should be transformed and where to send it, then everything else is handled on the back end.
  - Efficient Ingestion - Auto-optimization makes handling of large data sets more efficient.
  - Automated Orchestration - Necessary dependencies are handled automatically.
  - Auto-Incremental Load - Attempts to perform incremental loading of data before doing a full load if necessary. 
  - Unified Batch and Streaming Load
  - Automated Data Quality Checks - Automatically ensures erroneous / malformed data is properly handled.
  - Automated Monitoring - Automatically monitors data performance.
  - Serverless Compute - Automatically provisions compute resources.
- Disadvantages:
  - Not as flexible as a spark pipeline. 
- When to Use Streaming Table:
  - Queries defined against an incrementally growing and/or continuously changing data set.
  - Query results that need to be computed incrementally.
  - Pipelines that require high throughput and low latency for processing large amounts of data .
- When to Use Materialized View:
  - Caching data to save memory.
  - Data used by multiple downstream tables. Ensures tables will remain in sync as the data is changed or expanded.
  - Debugging
- When to Use Standard / Temporary View:
  - Break complex logic into simpler, more manageable pieces.

## Pipeline Setup Demo using LakeFlow Spark
- Before creating an SDP in DataBricks, ensure the 'LakeFlow Pipelines Editor' feature is enabled in user developer settings.
- SDPs are created under 'Jobs and Pipelines'. You will be given the opportunity to create a job, an ETL pipeline, or an ingestion pipeline.
- Settings:
  - One important pipeline setting is the source code directory, which defines the folder where all pipeline-related code is stored. Other important settings include the catalog and schema being used, as well as the type of compute resource being used. The compute resource can be either serverless or a specific, user-defined cluster.
  - External libraries, such as pandas, can be added in the pipeline environment section.
  - Runtime parameters are added in the configuration section.
- A pipeline dry-run ensures all pipeline settings are configured correctly and all pipeline-related code compiles and executes correctly.
- When a pipeline is run, the UI gives you access to each step that was run along with the DAG that shows the steps of each job and the related performance metrics.

## Initial Setup and User Interface Walkthrough
- Setting Up a Pipeline From an Existing Repository:
  - Grab the HTTPS clone link from the repo's github page.
  - Create a Git Folder in your workspace using the link.
- Once the folder is created, you can use the existing files to create the necessary resources needed to create the ETL pipeline.
  - Resources, such as catalogs, schemas, and tables are created in SQL files, while the pipeline-related code is written in python files.
- When creating the ETL pipeline, use the schema created in the above step and choose to create the pipeline using python.
- Once the pipeline is created, move the root folder of the pipeline to the root folder of the repo.

## Materialized View, Temporary View, and Streaming Table in LakeFlow Declarative Pipelines
- Creating a Materialized View
  ```python
  @dp.materialized_view(
      name = 'orders_mv', # name should be same as function name.
      comment = 'MV on source.orders for SDP purposes'
  )
  def orders_mv():
      df = spark.read.table('sdp_tutorial.source.orders') # Ensure the schema the pipeline is using is not the same as this schema.
      return df

  @dp.table() # streaming table
  def orders_stream():
      df = spark.readStream.table('sdp_tutorial.source.orders')
      return df

  @dp.temporary_view(
      name = 'orders_temp',
      comment = 'TV on source.orders for SDP purposes'
  )
  def orders_temp():
      df = spark.read.table('sdp_tutorial.source.orders')
      return df
  ```
  - When new records are appended to the underlying `sdp_tutorial.source.orders` table, the streaming table will only load those new records and the materialized view will show all records in the table, but add the additional records using an `append_only` operation.

## Building Your First LakeFlow Declarative Pipeline
```python
from pyspark import pipelines as dp
import pyspark.sql.functions as F

# Staging (Bronze) Layer
@dp.table(
    name = 'bronze_orders',
    comment = 'staging table on source.orders',
    table_properties = {
        'quality': 'bronze'
    }
)
def bronze_orders():
    df = spark.readStream.table('sdp_tutorial.source.orders')
    return df

# Sliver Layer (Minimal Filtering and Cleaning)
@dp.temporary_view(
    name = 'silver_orders',
    comment = 'silver view on bronze_orders, has some transformation on status'
)
def temporary_view():
    df = spark.readStream.table('bronze_orders')
    df.withColumn('order_status', F.upper(F.col('order_status')))
    return df

# Gold Layer
@dp.table(
    name = 'gold_layer',
    comment = 'gold aggregate table on silver_orders',
    table_properties = {
        'quality': 'gold'
    }  
)
def gold_layer():
    df = spark.readStream.table('silver_orders')
    df = df.groupBy('customer_id').agg(F.sum(F.col('order_value')).alias('customer_value'), F.count(F.col('order_id')).alias('total_sales'))
    return  df
```

## Sales Data Warehouse (DBW) Introduction using LakeFlow
- Fact Tables:
  - sales_west:
    - sale_id `INT PRIMARY KEY`
    - customer_id
    - product_id
    - sale_quantity
    - sale_timestamp
    - sale_amount
  - sales_east
    - sale_id `INT PRIMARY KEY`
    - customer_id
    - product_id
    - sale_quantity
    - sale_timestamp
    - sale_amount
- Dimension Tables:
  - products
    - product_id `INT PRIMARY KEY`
    - product_name
    - product_category
    - product_price
    - last_updated
  - customers
    - customer_id `INT PRIMARY KEY`
    - customer_name
    - region
    - last_updated
- Bronze Layer:
  - Combine sales_west and sales_east into combined_region_sales
  - Combine data from products and customers tables and perform data cleansing.
- Silver Layer (Update and Insert):
  - Capture changes from Bronze Layer.
- Gold Layer (SCD Type 2):
  - Track previous changes in Silver Layer.

## Data Ingestion to Bronze Layer using LakeFlow Pipelines
```python
from pyspark import pipelines as dp
from pyspark.sql import functions as F

# DataBricks Documents
# https://docs.databricks.com/aws/en/ldp/flow-examples

# Create an Empty Streaming Table
dp.create_streaming_table("bronze_regional_sales")

# Create Append Flow That Loads Data Into Target
@dp.append_flow(target = 'bronze_regional_sales')
def sales_west():
    df = spark.readStream.table('sdp_tutorial.source.sales_west')

    return (
        df.select(
            'sales_id',
            'customer_id',
            'product_id',
            'sale_quantity',
            'sale_timestamp',
            'sale_amount'
        )
    )

@dp.append_flow(target = 'bronze_regional_sales')
def sales_east():
    df = spark.readStream.table('sdp_tutorial.source.sales_east')

    return (
        df.select(
            'sales_id',
            'customer_id',
            'product_id',
            'sale_quantity',
            'sale_timestamp',
            'sale_amount'
        )
    )

# Create Bronze Product Table
@dp.table(name = 'bronze_product')
def bronze_product():
    return spark.readStream.table('sdp_tutorial.source.products')

# Create Bronze Customer Table
@dp.table(name = 'bronze_customer')
def bronze_customer():
    return spark.readStream.table('sdp_tutorial.source.customers')
```

## Data Quality Expectations in LakeFlow Declarative Pipelines
```python
from pyspark import pipelines as dp
from pyspark.sql import functions as F

# Perform Customer Data Quality Checks
customer_expectation_rules = {
    'valid customer id': "customer_id IS NOT NULL",
    'valid customer name': "customer_name IS NOT NULL",
    'valid customer region': "region IS NOT NULL"
}
quarantine_rules = "NOT({0})".format(" AND ".join(customer_expectation_rules.values()))

@dp.view(name = 'raw_customers')
def raw_customers():
    return spark.readStream.table('sdp_tutorial.source.customers')

@dp.table(
    name = 'customers_quarantine',
    comment = 'quarantine table for customers'
)
@dp.expect_all(customer_expectation_rules) # Raises warning when above quality checks fail.
def customers_quarantine():
    return (
        spark.readStream.table('raw_customers')
        .withColumn('is_quarantined', F.expr(quarantine_rules))
    )

@dp.view
def valid_customers():
    return spark.readStream.table('customers_quarantine').filter("is_quarantined = false")

@dp.view
def invalid_customers():
    return spark.read.table('customers_quarantine').filter("is_quarantined = true")

# Perform Product Data Quality Checks
product_expectation_rules = {
    'valid product id': "product_id IS NOT NULL",
    'valid product price': "product_price > 0"
}
quarantine_rules = "NOT({0})".format(" AND ".join(product_expectation_rules.values()))

@dp.view(name = 'raw_products')
def raw_products():
    return spark.readStream.table('sdp_tutorial.source.products')

@dp.table(
    name = 'products_quarantine',
    comment = 'quarantine table for products'
)
@dp.expect_all(product_expectation_rules) # Raises warning when above quality checks fail.
def products_quarantine():
    return (
        spark.readStream.table('raw_products')
        .withColumn('is_quarantined', F.expr(quarantine_rules))
    )

@dp.view
def valid_products():
    return spark.readStream.table('products_quarantine').filter("is_quarantined = false")

@dp.view
def invalid_products():
    return spark.read.table('products_quarantine').filter("is_quarantined = true")

# Perform Sales Data Quality Checks
sales_expectation_rules = {
    'valid sales id': "sales_id IS NOT NULL",
    'valid sales customer id': "customer_id IS NOT NULL",
    'valid sales product id': "product_id IS NOT NULL",
    'valid sales quantity': "sale_quantity > 0"
}
quarantine_rules = "NOT({0})".format(" AND ".join(sales_expectation_rules.values()))

@dp.view(name = 'raw_sales')
def raw_sales():
    return spark.readStream.table('sdp_tutorial.sdp_test.bronze_regional_sales')

@dp.table(
    name = 'sales_quarantine',
    comment = 'quarantine table for sales'
)
@dp.expect_all(sales_expectation_rules) # Raises warning when above quality checks fail.
def sales_quarantine():
    return (
        spark.readStream.table('raw_sales')
        .withColumn('is_quarantined', F.expr(quarantine_rules))
    )

@dp.view
def valid_sales():
    return spark.readStream.table('sales_quarantine').filter("is_quarantined = false")

@dp.view
def invalid_sales():
    return spark.read.table('sales_quarantine').filter("is_quarantined = true")
```

## Auto Change Data Capture (CDC) and Slowly Changing Dimension (SCD) Type 2 – Silver Layer
```python
from pyspark import pipelines as dp
from pyspark.sql import functions as F

# Create auto_cdc_flow
# DataBricks Documentation:
# https://docs.databricks.com/aws/en/ldp/developer/ldp-python-ref-apply-changes

# Step 1: Create an Empty Streaming Tables For Customers, Products, and Sales
dp.create_streaming_table('dim_customer')

dp.create_auto_cdc_flow(
    target = 'dim_customer',
    source = 'valid_customers',
    keys = ['customer_id'],
    sequence_by = 'last_updated',
    ignore_null_updates = False,
    apply_as_deletes = None,
    apply_as_truncates = None,
    column_list = None,
    except_column_list = None,
    stored_as_scd_type = 2,
    track_history_column_list = None,
    track_history_except_column_list = None,
    name = None,
    once = False
)

dp.create_streaming_table('dim_product')

dp.create_auto_cdc_flow(
    target = 'dim_product',
    source = 'valid_products',
    keys = ['product_id'],
    sequence_by = 'last_updated',
    ignore_null_updates = False,
    apply_as_deletes = None,
    apply_as_truncates = None,
    column_list = None,
    except_column_list = None,
    stored_as_scd_type = 2,
    track_history_column_list = None,
    track_history_except_column_list = None,
    name = None,
    once = False
)

dp.create_streaming_table('fact_sales')

dp.create_auto_cdc_flow(
    target = 'fact_sales',
    source = 'valid_sales',
    keys = ['sales_id', 'customer_id', 'product_id'],
    sequence_by = 'sale_timestamp',
    ignore_null_updates = False,
    apply_as_deletes = None,
    apply_as_truncates = None,
    column_list = None,
    except_column_list = None,
    stored_as_scd_type = 1 ,
    track_history_column_list = None,
    track_history_except_column_list = None,
    name = None,
    once = False
)

# Step 2: Perform Aggregations on Sales Data
@dp.table(
    name = 'business_sales',
    comment = 'sales of business with region',
    table_properties = {
        "pipelines.autoOptimize.managed": "true",
        "quality": "Gold"
    }
)
def business_sales():
    df_fact_sales = spark.read.table('fact_sales')
    df_dim_product = spark.read.table('dim_product')
    df_dim_customer = spark.read.table('dim_customer')

    df_join = (
        df_fact_sales
            .join(
                df_dim_customer, df_fact_sales.customer_id == df_dim_customer.customer_id, 'inner'
            )
            .join(
                df_dim_product, df_fact_sales.product_id == df_dim_product.product_id, 'inner'
            )
    )

    df_final = (
        df_join
            .withColumn('total_sales', F.col('sale_quantity') * F.col('sale_amount'))
            .select('region', 'product_category', 'total_sales')
    )

    df_agg = df_final.groupBy('region', 'product_category').agg(F.sum('total_sales').alias('total_revenue'))

    return df_agg
```

## Sink Configuration in LakeFlow Declarative Pipelines
```python
from pyspark import pipelines as dp

# Sink
# Documentation: https://docs.databricks.com/aws/en/ldp/ldp-sinks

dp.create_sink(
    name = 'region_business_sales',
    format = 'delta',
    options = {
        "tableName": "workspace.default.region_business_sales"
    }
)

@dp.append_flow(target = 'region_business_sales')
def region_business_sales():
    return spark.readStream.table('sdp_tutorial.sales_dbw.business_sales')
```

## Supporting SQL

- [[02_technical_prep/spark_databricks/Lakeflow Initial Setup]]
- [[02_technical_prep/spark_databricks/Lakeflow Sales Data]]
