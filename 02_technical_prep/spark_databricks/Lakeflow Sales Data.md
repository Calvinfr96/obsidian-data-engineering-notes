---
aliases:
  - "sales_data.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Spark and DataBricks/sales_data.sql"
imported: 2026-09-24
---

# Lakeflow Sales Data

Review: [[02_technical_prep/Databricks Review|Databricks Review]]

```sql
-- Data Used in Building Demo Pipeline

-- East region sales 
-- drop table sales_west;

CREATE TABLE sales_east (
  sales_id INT PRIMARY KEY,
  customer_id INT,
  product_id INT,
  sale_quantity INT,
  sale_timestamp TIMESTAMP,
  sale_amount DECIMAL(10, 2)
);
-- Initial data load
INSERT INTO sales_east (sales_id, customer_id, product_id, sale_quantity, sale_timestamp, sale_amount) VALUES
  (1, 101, 1001, 5, '2022-01-01 10:00:00', 50.00),
  (2, 102, 1002, 3, '2022-01-02 11:00:00', 30.00),
  (3, 103, 1003, 7, '2022-01-02 11:15:00', 70.00),
  (4, 104, 1004, 2, '2022-01-02 11:45:00', 20.00),
  (5, 105, 1005, 4, '2022-01-02 12:00:00', 40.00);
-- Incremental data load
INSERT INTO sales_east (sales_id, customer_id, product_id, sale_quantity, sale_timestamp, sale_amount) VALUES
  (6, 105, 1003, 6, '2022-01-03 15:09:00', 60.00),
  (7, 102, 1007, 8, '2022-01-03 16:00:00', 80.00)
;
-- West region sales 
CREATE TABLE sales_west (
  sales_id INT PRIMARY KEY,
  customer_id INT,
  product_id INT,
  sale_quantity INT,
  sale_timestamp TIMESTAMP,
  sale_amount DECIMAL(10, 2)
);
-- Initial data load
INSERT INTO sales_west (sales_id, customer_id, product_id, sale_quantity, sale_timestamp, sale_amount) VALUES
  (8, 108, 1008, 5, '2022-01-01 10:00:00', 50.00),
  (9, 109, 1009, 3, '2022-01-02 11:00:00', 30.00),
  (10, 110, 1010, 7, '2022-01-02 11:15:00', 70.00),
  (11, 111, 1011, 2, '2022-01-02 11:45:00', 20.00),
  (12, 112, 1012, 4, '2022-01-02 12:00:00', 40.00);
-- Incremental data load
INSERT INTO sales_east (sales_id, customer_id, product_id, sale_quantity, sale_timestamp, sale_amount) VALUES
  (13, 112, 1003, 16, '2022-01-03 15:09:00', 60.00),
  (14, 109, 1007, 81, '2022-01-03 16:00:00', 80.00);

-- --------------------------------------------------
-- product data

create table products(
  product_id int primary key,
  product_name string,
  product_category string,
  product_price decimal(10,2),
  last_updated timestamp
);

INSERT INTO products (product_id, product_name, product_category, product_price, last_updated) VALUES
(1001, 'Laptop', 'Electronics', 1000.00, '2025-07-31 12:00:00'),
(1002, 'Phone', 'Electronics', 120.00, '2025-07-31 12:05:00'),
(1003, 'Monitor', 'Electronics', 100.00, '2025-07-31 12:10:00'),
(1004, 'Chair', 'Furniture', 110.00, '2025-07-31 12:15:00'),
(1005, 'Desk', 'Furniture', 150.00, '2025-07-31 12:20:00'),
(1006, 'Mouse', 'Electronics', 50.00, '2025-07-31 12:25:00'),
(1007, 'Keyboard', 'Electronics', 60.00, '2025-07-31 12:30:00'),
(1008, 'Lamp', 'Furniture', 130.00, '2025-07-31 12:35:00'),
(1009, 'Router', 'Electronics', 130.00, '2025-07-31 12:40:00'),
(1010, 'Table', 'Furniture', 130.00, '2025-07-31 12:45:00'),
(1011, 'Notebook', 'Stationery', 140.00, '2025-07-31 12:50:00'),
(1012, 'Pen', 'Stationery', 150.00, '2025-07-31 12:55:00');


-- Price change for product_id 203
INSERT INTO products VALUES
(1003, 'Monitor', 'Electronics', 90.00, '2025-08-02 08:00:00');

-- Name change for product_id 208
INSERT INTO products VALUES
(1008, 'Desk Lamp', 'Furniture', 130.00, '2025-08-02 08:10:00');


-- -----------------------------------------------------------------------------------------------


CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name string,
    `region` string,
    last_updated TIMESTAMP
);

INSERT INTO customers VALUES
(101, 'Alice', 'East', '2025-07-31 13:00:00'),
(102, 'Bob', 'East', '2025-07-31 13:05:00'),
(103, 'Charlie', 'East', '2025-07-31 13:10:00'),
(104, 'Diana', 'East', '2025-07-31 13:15:00'),
(105, 'Ethan', 'East', '2025-07-31 13:20:00'),
(106, 'Fiona', 'East', '2025-07-31 13:25:00'),
(107, 'George', 'West', '2025-07-31 13:30:00'),
(108, 'Hannah', 'West', '2025-07-31 13:35:00'),
(109, 'Ian', 'West', '2025-07-31 13:40:00'),
(110, 'Jane', 'West', '2025-07-31 13:45:00'),
(111, 'Kevin', 'West', '2025-07-31 13:50:00'),
(112, 'Laura', 'West', '2025-07-31 13:55:00');


-- Region change for customer 103
INSERT INTO customers VALUES
(103, 'Charlie', 'Central', '2025-08-02 08:30:00');

-- Name correction for customer 107
INSERT INTO customers VALUES
(107, 'George Smith', 'West', '2025-08-02 08:40:00');
```
