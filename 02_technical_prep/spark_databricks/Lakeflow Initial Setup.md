---
aliases:
  - "initial_setup.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Spark and DataBricks/initial_setup.sql"
imported: 2026-09-24
---

# Lakeflow Initial Setup

Review: [[02_technical_prep/Databricks Review|Databricks Review]]

```sql
-- Common SQL query DO NOT run this more than once
-- Create a new catalog for the SDP tutorial
CREATE CATALOG IF NOT exists SDP_tutorial;

-- Create new schema for use of SDP tutorials
CREATE SCHEMA IF NOT EXISTS SDP_tutorial.source;

--Start of SQL command for Orders table 
-- Create orders table
CREATE TABLE SDP_tutorial.source.orders (
  order_id INT,
  customer_id INT,
  order_date DATE,
  order_status STRING,
  order_value DECIMAL(10,2)
);

-- Insert data into orders
INSERT INTO SDP_tutorial.source.orders VALUES
(1, 1, '2022-01-01', 'shipped', 100.00),
(2, 2, '2022-01-02', 'pending', 200.00),
(3, 3, '2022-01-03', 'returned', 300.00),
(4, 4, '2022-01-04', 'returned', 400.00),
(5, 5, '2022-01-05', 'shipped', 500.00)
;

INSERT INTO SDP_tutorial.source.orders VALUES
(6, 1, '2022-01-21', 'shipped', 100.00),
(7, 2, '2022-01-22', 'pending', 200.00);
-- Check the table
select * from sdp_tutorial.source.orders;
--End of SQL command for Orders table
```
