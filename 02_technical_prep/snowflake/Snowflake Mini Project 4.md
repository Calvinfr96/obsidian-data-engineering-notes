---
aliases:
  - "mini_project_4.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Snowflake/Mini Projects/mini_project_4.sql"
imported: 2026-09-24
---

# Snowflake Mini Project 4

Review: [[02_technical_prep/Snowflake Review|Snowflake Review]]

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;

-- Create Database
CREATE OR REPLACE DATABASE TIMETRAVEL_DB;

-- Create Schema
USE DATABASE TIMETRAVEL_DB;
CREATE OR REPLACE SCHEMA TIMETRAVEL_DATA;

-- Create and Populate Table
CREATE OR REPLACE TABLE TIMETRAVEL_DB.TIMETRAVEL_DATA.EMPLOYEE (
    EMPLOYEE_ID STRING,
    FIRST_NAME STRING,
    LAST_NAME STRING,
    DEPARTMENT STRING,
    SALARY FLOAT,
    HIRE_DATE DATE
);

INSERT INTO TIMETRAVEL_DB.TIMETRAVEL_DATA.EMPLOYEE VALUES
    ('E1', 'John', 'Doe', 'Finance', 75000.50, '2020-01-15'), 
    ('E2', 'Jane', 'Smith', 'HR', 68000.00, '2018-03-20'), 
    ('E3', 'Alice', 'Johnson', 'IT', 92000.75, '2019-07-10'),
    ('E4', 'Bob', 'Williams', 'Sales', 58000.25, '2021-06-01'), 
    ('E5', 'Charlie', 'Brown', 'Marketing', 72000.00, '2022-04-22'), 
    ('E6', 'Emily', 'Davis', 'IT', 89000.10, '2017-11-12'), 
    ('E7', 'Frank', 'Miller', 'Finance', 83000.30, '2016-09-05'), 
    ('E8', 'Grace', 'Taylor', 'Sales', 61000.45, '2023-02-11'),
    ('E9', 'Hannah', 'Moore', 'HR', 67000.80, '2020-05-18'), 
    ('E10', 'Jack', 'White', 'Marketing', 70000.90, '2019-12-25');

-- 'Accidentally' Delete Employees E2 and E7
DELETE FROM TIMETRAVEL_DB.TIMETRAVEL_DATA.EMPLOYEE WHERE EMPLOYEE_ID IN ('E2', 'E7');

-- Recover Selected Data
TRUNCATE TIMETRAVEL_DB.TIMETRAVEL_DATA.EMPLOYEE;
INSERT INTO TIMETRAVEL_DB.TIMETRAVEL_DATA.EMPLOYEE
    SELECT *
    FROM TIMETRAVEL_DB.TIMETRAVEL_DATA.EMPLOYEE
    BEFORE (STATEMENT => '01c2b78d-0000-d200-0017-3bf70007f032')
    WHERE EMPLOYEE_ID IN ('E2', 'E7');
```
