---
aliases:
  - "snowflake_notes"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Snowflake/snowflake_notes.md"
imported: 2026-09-24
---

# Snowflake Worked Examples

Review: [[02_technical_prep/Snowflake Review|Snowflake Review]]

Use [[02_technical_prep/Snowflake Review|Snowflake Review]] for the concept review. These are the additional worked examples from the course notes.

## Contents

- [[#Snowflake Database/Schema]]
- [[#Snowflake Table Types]]
- [[#Snowflake View Types]]
- [[#Snowflake Stages]]
- [[#Snowflake File Formats]]
- [[#Snowflake Data Loading Approaches]]
- [[#Snowflake Bulk Data Load]]
- [[#Snowflake Continuous Data Loading]]
- [[#Snowflake Streams]]
- [[#Snowflake Tasks]]
- [[#Snowflake Time Travel and Fail Safe]]
- [[#Snowflake Cloning]]
- [[#Loading Semi-Structured Data]]
- [[#Example data files]]

## Snowflake Database/Schema

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Databases and schemas in snowflake can either be created using the UI or with SQL DDL code. Before running any SQL queries in snowflake, the role and warehouse through which the SQL code will be run must first be defined.

```sql
-- Initial Setup (Selecting the execution role and warehouse)
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;

-- Create Database
CREATE OR REPLACE DATABASE MYDB;

-- Create Schema
USE DATABASE MYDB; -- First, select that database for which you want to create the schema.
CREATE OR REPLACE SCHEMA MYSCHEMA;
```

## Snowflake Table Types

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Creating Tables in Snowflake:

```sql
-- Initial Setup (Selecting the execution role, warehouse, and schema)
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Create Permanent Table
CREATE OR REPLACE TABLE PERMANENT_TABLE ( -- Table type not specified, permanent table created by default.
    ID INT,
    NAME STRING
);

-- Create Transient Table
CREATE OR REPLACE TRANSIENT TABLE TRANSIENT_TABLE (
    ID INT,
    NAME STRING
);

-- Create Temporary Table
CREATE OR REPLACE TEMPORARY TABLE TEMPORARY_TABLE (
    ID INT,
    NAME STRING
);

-- Show All Tables
SHOW TABLES; -- Displays table details for all tables under a given schema (you typically won't specify table type in the name of a table)

-- Check Data Retention Period at Account/Database/Schema/Table Level
SHOW PARAMETERS LIKE 'DATA_RETENTION_TIME_IN_DAYS' IN ACCOUNT; -- Account Level.
SHOW PARAMETERS IN DATABASE MYDB; -- Database Level.
SHOW PARAMETERS IN SCHEMA MYDB.MYSCHEMA; -- Schema Level.
SHOW PARAMETERS IN TABLE MYDB.MYSCHEMA.PERMANENT_TABLE; -- Table Level.

-- Changing Data Retention Period of PERMANENT_TABLE
ALTER TABLE PERMANENT_TABLE SET DATA_RETENTION_TIME_IN_DAYS = 90; -- Maximum allowable retention for a permanent table.
```

## Snowflake View Types

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Creating Views in Snowflake:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Create Table
CREATE OR REPLACE TABLE EMPLOYEE (
    ID INT,
    NAME STRING,
    DEPARTMENT STRING,
    SALARY INT
);

-- Inserting Table Data
INSERT INTO EMPLOYEE (ID, NAME, DEPARTMENT, SALARY)
VALUES
    (1, 'Pat Fay', 'HR', 50000),
    (2, 'Donald OConnell', 'IT', 75000),
    (3, 'Steve King', 'Sales', 60000),
    (4, 'Susan Marvis', 'IT', 80000),
    (5, 'Jennifer Whalen', 'Marketing', 55000);

-- Extract Data From Table
SELECT * FROM EMPLOYEE;

-- Create Standard View For IT Employees (IT Records Only)
CREATE OR REPLACE VIEW IT_EMPLOYEE AS ( -- Creates standard view by default.
    SELECT ID, NAME, SALARY
    FROM EMPLOYEE
    WHERE DEPARTMENT = 'IT'
);

-- Select Data From IT_EMPLOYEE View
SELECT * FROM IT_EMPLOYEE;

-- Create Secure View For HR Employees
CREATE OR REPLACE SECURE VIEW HR_EMPLOYEE AS (
    SELECT ID, NAME, DEPARTMENT
    FROM EMPLOYEE
    WHERE DEPARTMENT = 'HR'
);

-- Select Data From HR_EMPLOYEE View
SELECT * FROM HR_EMPLOYEE;

-- Create View That Aggregates Salaries By Department
CREATE OR REPLACE VIEW DEPARTMENT_SALARIES AS (
    SELECT
        DEPARTMENT,
        SUM(SALARY) AS TOTAL_SALARY -- Alias required in snowflake.
    FROM EMPLOYEE
    GROUP BY DEPARTMENT
);

-- Select Data From DEPARTMENT_SALARIES View
SELECT * FROM DEPARTMENT_SALARIES; -- Snowflake will perform aggregation on the fly when the query is run. The query results are not cached.

-- Create Materialized View That Aggregates Salaries By Department
CREATE OR REPLACE MATERIALIZED VIEW DEPARTMENT_SALARIES_MAT AS (
    SELECT
        DEPARTMENT,
        SUM(SALARY) AS TOTAL_SALARY -- Alias required in snowflake.
    FROM EMPLOYEE
    GROUP BY DEPARTMENT
);

-- Select Data From DEPARTMENT_SALARIES_MAT View
SELECT * FROM DEPARTMENT_SALARIES_MAT; -- Snowflake will pre-compute and cache the query results before running the SELECT statement.

-- Show All Views Under Schema
SHOW VIEWS;
```

## Snowflake Stages

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Working With Stages:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Accessing User Stage (Lists files uploaded to the stage)
LIST @~; -- Created automatically when user account is created.

-- Create Customer Table
CREATE OR REPLACE TABLE CUSTOMER (
    ID INT,
    NAME STRING,
    AGE INT,
    STATE STRING
);

-- Access Table Stage
LIST @%CUSTOMER; -- Created automatically when table is created.

-- Create Named Internal Stage
CREATE OR REPLACE STAGE CUSTOMER_STAGE;

-- Access Named Stage
LIST @CUSTOMER_STAGE; -- Manually created stage. Also manually uploaded file to stage using the UI

-- Copy Data From Stage Into Table
COPY INTO CUSTOMER
FROM @CUSTOMER_STAGE
FILE_FORMAT = (TYPE = 'CSV', SKIP_HEADER = 1);

-- Verify Data Was Loaded From Stage To Table
SELECT * FROM CUSTOMER;
```

CSV File Used:

```sql
"ID","NAME","AGE","STATE"
1,"Donald",25,"New York"
2,"Douglas",35,"Chicago"
3,"Jennifer",40,"California"
4,"Michael",55,"Texas"
5,"Pat",60,"Florida"
```

## Snowflake File Formats

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Example File Format:

```sql
CREATE FILE FORMAT DEMO_FF
(
  TYPE = 'CSV'
  FIELD_DELIMITER = ','
  RECORD_DELIMITER - '\n'
  SKIP_HEADER = 1
);
```

Working With File Formats:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Create STUDENT Table
CREATE OR REPLACE TABLE STUDENT (
    ID INT,
    NAME STRING,
    MARKS INT
);

-- Create Named Stage For STUDENT Table
CREATE OR REPLACE STAGE STUDENT_STAGE;

-- List STUDENT_STAGE
LIST @STUDENT_STAGE;

-- Copy Data From File Into STUDENT Table
COPY INTO STUDENT
FROM @STUDENT_STAGE
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1); -- Creates file format each time the COPY command is run.

-- Select Data From STUDENT Table
SELECT * FROM STUDENT;

-- Truncate (Remove Data and Metadata) FROM STUDENT Table
TRUNCATE TABLE STUDENT;

-- Create CSV File Format
CREATE OR REPLACE FILE FORMAT CSV_FORMAT
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    RECORD_DELIMITER = '\n'
    SKIP_HEADER = 1;

-- Copy Data From File Into STUDENT Table Using CSV_FORMAT
COPY INTO STUDENT
FROM @STUDENT_STAGE
FILE_FORMAT = (FORMAT_NAME = CSV_FORMAT);

-- Create JSON File Format
CREATE OR REPLACE FILE FORMAT JSON_FORMAT
    TYPE = 'JSON';

--- List All File Formats
SHOW FILE FORMATS;
```

## Snowflake Data Loading Approaches

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

The command can be modified as follows to create a pipe with auto-ingest enabled:

```sql
CREATE PIPE snow_pipe_db.public.my_pipe AUTO_INGEST = TRUE AS
  COPY INTO snow_pipe_db.public.my_table
  FROM @snow_pipe_db.public.my_stage
  FILE_FORMAT = (TYPE = 'JSON');
```

## Snowflake Bulk Data Load

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

External Stage Example:

```sql
-- Create External Stage
CREATE STAGE my_s3_stage
  url = 's3://my_bucket/encrypted_files/'
  CREDENTIALS = (
    aws_key_id = '*****'
    aws_secret_key = '*****'
  )
  ENCRYPTION = (master_key = '********')
  FILE_FORMAT = my_csv_format;

-- Load Data
COPY INTO my_table
FROM @my_s3_stage
PATTERN = '.*sales.*.csv';
```

Working With Bulk Loading and External Stages:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Create Storage Integration Between Snowflake and S3
CREATE OR REPLACE STORAGE INTEGRATION MY_S3_INT
    TYPE = EXTERNAL_STAGE
    STORAGE_PROVIDER = 'S3'
    ENABLED = TRUE
    STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::515424600331:role/my_snowflake_role' -- IAM Role ARN
    STORAGE_ALLOWED_LOCATIONS = ('s3://dea-snowflake-data-bucket-cf1/bulk_data/'); -- S3 Bucket Folder URI

-- Create Trust Relationship Between S3 and Snowflake
DESCRIBE INTEGRATION MY_S3_INT;
-- Use the STORAGE_AWS_IAM_USER_ARN property as the principal in the trust policy of the my_snowflake_role IAM role.
-- Use the STORAGE_AWS_EXTERNAL_ID property as the external ID in the trust policy of the my_snowflake_role IAM role.

-- Create File Format
CREATE OR REPLACE FILE FORMAT MY_CSV_FILE_FORMAT
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    RECORD_DELIMITER = '\n'
    SKIP_HEADER = 1;

-- Create External Stage
CREATE OR REPLACE STAGE MY_S3_STAGE
    STORAGE_INTEGRATION = MY_S3_INT -- Integration which has trust relationship with S3 bucket.
    URL = 's3://dea-snowflake-data-bucket-cf1/bulk_data/' -- Same as the STORAGE_ALLOWED_LOCATIONS of the storage integration.
    FILE_FORMAT = MY_CSV_FILE_FORMAT;

-- Access The S3 External Stage
LIST @MY_S3_STAGE; -- Shows files uploaded to the S3 Bucket configured above.

-- Create User Table
CREATE OR REPLACE TABLE USER (
    ID INT,
    NAME STRING,
    LOCATION STRING,
    EMAIL STRING
);

-- Load Data From S3 Into USER Table
COPY INTO USER
FROM @MY_S3_STAGE
FILE_FORMAT = (FORMAT_NAME = MY_CSV_FILE_FORMAT);

-- Confirm Data Loaded Into USER Table
SELECT * FROM USER;
```

IAM Role Trust Policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::670441838025:user/84ji1000-s"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "UJC16642_SFCRole=2_YVmC/rMqanCa7WANqVJOacDFryg="
                }
            }
        }
    ]
}
```

CSV File Used:

```sql
"ID","NAME","LOCATION","EMAIL"
1,John,USA,"john@email.com"
2,Sam,Canada,"sam@email.com"
3,Kyle,Paris,"kyle@email.com"
4,Ashley,London,"ashley@email.com"
5,Pat,Spain,"pat@email.com"
```

## Snowflake Continuous Data Loading

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Working With Continuous Loading Using Snow Pipe and External Stage:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

CREATE OR REPLACE STORAGE INTEGRATION MY_SNOWPIPE_INT
    TYPE = EXTERNAL_STAGE
    STORAGE_PROVIDER = 'S3'
    ENABLED = TRUE
    STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::515424600331:role/my_snowpipe_role' -- Snow Pipe Role ARN
    STORAGE_ALLOWED_LOCATIONS = ('s3://dea-snowflake-data-bucket-cf1/event_data/'); -- S3 Bucket Folder URI

-- Create Trust Relationship Between S3 and Snowflake
DESCRIBE INTEGRATION MY_SNOWPIPE_INT;
-- Pull STORAGE_AWS_ROLE_ARN and STORAGE_AWS_EXTERNAL_ID from the results and use that to update the IAM role trust policy.

-- Create JSON File Format
CREATE OR REPLACE FILE FORMAT MY_JSON_FORMAT
    TYPE = 'JSON';

-- Create External S3 Stage
CREATE OR REPLACE STAGE MY_SNOWPIPE_STAGE
    STORAGE_INTEGRATION = MY_SNOWPIPE_INT
    URL = 's3://dea-snowflake-data-bucket-cf1/event_data/'
    FILE_FORMAT = MY_JSON_FORMAT;

-- Validate Snow Pipe Stage
LIST @MY_SNOWPIPE_STAGE

-- Create Event Table
CREATE OR REPLACE TABLE EVENT (
    EVENT VARIANT -- Data type used for semi-structured data.
);

-- Create Snow Pipe (Automatically adds files from S3 to the EVENT table)
CREATE OR REPLACE PIPE MY_EVENT_PIPE
    AUTO_INGEST = TRUE AS
    COPY INTO EVENT
    FROM @MY_SNOWPIPE_STAGE
    FILE_FORMAT = (FORMAT_NAME = MY_JSON_FORMAT);

-- Check Status of Snow Pipe (Including the last time a file was uploaded)
SELECT SYSTEM$PIPE_STATUS('MY_EVENT_PIPE');

-- Create Notification Channel (Receive S3 Notifications About New File Uploads)
SHOW PIPES; -- Provides information about the Snowflake SQS queue ARN that will get the notifications.
-- Use the ARN to create an S3 event notification in the AWS account where the S3 bucket was created.

-- Verify Data in EVENT Table
SELECT * FROM EVENT;
```

IAM Role Trust Policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::670441838025:user/84ji1000-s"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "UJC16642_SFCRole=2_llk29j8BV6CkFDkdh6mUtjAYZa4="
                }
            }
        }
    ]
}
```

JSON File Used:

```json
{
  "type": "INFO",
  "messageId": "msg-001",
  "message": "Lambda function started successfully.",
  "timestamp": "2025-11-11T06:30:00Z"
}
```

## Snowflake Streams

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Working With Streams:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Create Source Table 1
CREATE OR REPLACE TABLE SOURCE_TABLE1 (
    ID INT,
    NAME STRING,
    CREATE_DATE DATE
);
INSERT INTO SOURCE_TABLE1 VALUES
    (1, 'Tom', '2025-10-31'),
    (2, 'Sam', '2025-10-31');

SELECT * FROM SOURCE_TABLE1 ORDER BY ID;

-- Create Standard Stream and Make Changes
CREATE OR REPLACE STREAM STANDARD_STREAM ON TABLE SOURCE_TABLE1;
INSERT INTO SOURCE_TABLE1 VALUES
    (3, 'Johnny', '2025-11-01');
DELETE FROM SOURCE_TABLE1 WHERE ID = 2;
UPDATE SOURCE_TABLE1 SET NAME = 'Jeff' WHERE ID = 1; -- Shows up as an delete-insert pair in the stream. ISUPDATE is set to TRUE for both rows.

SELECT * FROM STANDARD_STREAM ORDER BY ID; -- Will show a record of the above operations, but not the first two INSERT operations sense they were executed before the stream was created.

-- Create Source Table 2
CREATE OR REPLACE TABLE SOURCE_TABLE2 (
    ID INT,
    NAME STRING,
    CREATE_DATE DATE
);
INSERT INTO SOURCE_TABLE2 VALUES
    (1, 'Tom', '2025-10-31'),
    (2, 'Sam', '2025-10-31');

SELECT * FROM SOURCE_TABLE2 ORDER BY ID;

-- Create Append-Only Stream and Make Changes
CREATE OR REPLACE STREAM APPEND_ONLY_STREAM ON TABLE SOURCE_TABLE2 APPEND_ONLY = TRUE;
INSERT INTO SOURCE_TABLE2 VALUES
    (3, 'Johnny', '2025-11-01');
DELETE FROM SOURCE_TABLE2 WHERE ID = 2; -- Not recorded in append-only stream.
UPDATE SOURCE_TABLE2 SET NAME = 'Jeff' WHERE ID = 1; -- Not recorded in append-only stream.
INSERT INTO SOURCE_TABLE2 VALUES
    (4, 'Tony', '2025-11-02');

SELECT * FROM APPEND_ONLY_STREAM ORDER BY ID;

-- Create Target Table to Load Data From Stream
CREATE OR REPLACE TABLE TARGET_TABLE (
    ID INT,
    NAME STRING,
    CREATE_DATE DATE
);
INSERT INTO TARGET_TABLE
SELECT ID, NAME, CREATE_DATE FROM APPEND_ONLY_STREAM; -- Using the append-only stream to populate a newly-created table.

SELECT * FROM TARGET_TABLE ORDER BY ID;
SELECT * FROM APPEND_ONLY_STREAM ORDER BY ID; -- Stream is now empty since the data was added to a new table.
INSERT INTO SOURCE_TABLE2 VALUES
    (5, 'Rock', '2025-11-03');
INSERT INTO TARGET_TABLE
SELECT ID, NAME, CREATE_DATE FROM APPEND_ONLY_STREAM; -- Only adds the recently-inserted record with ID = 5 to the table, not all of the other inserted records.

-- Create Insert-Only Stream (External Tables Only)
CREATE OR REPLACE EXTERNAL TABLE EXT_TABLE
LOCATION = @MY_S3_STAGE
FILE_FORMAT = MY_CSV_FILE_FORMAT

CREATE OR REPLACE STREAM MY_EXT_STREAM
ON EXTERNAL TABLE EXT_TABLE
INSERT_ONLY = TRUE;
```

## Snowflake Tasks

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

User-Defined Task - Allows you to manage compute resources for individual tasks by specifying an existing, **correctly-sized** virtual warehouse when creating the task.

```sql
CREATE TASK my_task
WAREHOUSE = COMPUTE_WH
SCHEDULE = '5 MINUTE'
AS
INSERT INTO employees VALUES (EMPLOYEE_SEQUENCE, 'F_Name', 'L_Name', '101);
```

Serverless Task - Allows you to execute tasks by relying on compute resources managed by snowflake.

```sql
CREATE TASK my_task_serverless
USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE = 'XSMALL' -- Automatically upgraded by snowflake if necessary.
SCHEDULE = '5 MINUTE'
AS
INSERT INTO employees VALUES (EMPLOYEE_SEQUENCE, 'F_Name', 'L_Name', '101);
```

NONCRON Notation:

```sql
CREATE TASK my_task
WAREHOUSE = COMPUTE_WH
SCHEDULE = '5 MINUTE'
AS
INSERT INTO employees VALUES (EMPLOYEE_SEQUENCE, 'F_Name', 'L_Name', '101);
```

CRON Notation:

```sql
CREATE TASK my_task
WAREHOUSE = COMPUTE_WH
SCHEDULE = 'USING CRON*10**SUN UTC' -- Every Sunday at 10AM UTC
AS
INSERT INTO employees VALUES (EMPLOYEE_SEQUENCE, 'F_Name', 'L_Name', '101);
```

Working With Tasks:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Executing SQL Actions Without Tasks
CREATE OR REPLACE TABLE SOURCE_TABLE (
    ID INT,
    NAME STRING,
    CREATE_DATE DATE
);
INSERT INTO SOURCE_TABLE VALUES
    (1, 'Tom', '2025-10-31'),
    (2, 'Sam', '2025-10-31');
SELECT * FROM SOURCE_TABLE;

CREATE OR REPLACE TABLE TARGET_TABLE (
    ID INT,
    NAME STRING,
    CREATE_DATE DATE,
    CREATE_DAY STRING,
    CREATE_MONTH STRING,
    CREATE_YEAR STRING
);
INSERT INTO TARGET_TABLE
    SELECT
        A.ID,
        A.NAME,
        A.CREATE_DATE,
        DAY(A.CREATE_DATE) AS CREATE_DAY,
        MONTH(A.CREATE_DATE) AS CREATE_MONTH,
        YEAR(A.CREATE_DATE) AS CREATE_YEAR
    FROM SOURCE_TABLE A
    LEFT JOIN TARGET_TABLE B -- Need to join with TARGET_TABLE to ensure only non-existent records are added from SOURCE_TABLE.
    ON A.ID = B.ID
    WHERE B.ID IS NULL;
SELECT * FROM TARGET_TABLE;
INSERT INTO SOURCE_TABLE VALUES
    (3, 'Elon', '2025-11-01'); -- Only this new record will be added to TARGET_TABLE when above INSERT statement is run.

-- Executing SQL Actions With Tasks
CREATE OR REPLACE TABLE TARGET_TABLE_TASK (
    ID INT,
    NAME STRING,
    CREATE_DATE DATE,
    CREATE_DAY STRING,
    CREATE_MONTH STRING,
    CREATE_YEAR STRING
);

CREATE OR REPLACE TASK MY_TARGET_TABLE_TASK
    WAREHOUSE = COMPUTE_WH
    SCHEDULE = '1 MINUTE'
AS
    INSERT INTO TARGET_TABLE_TASK
        SELECT
            A.ID,
            A.NAME,
            A.CREATE_DATE,
            DAY(A.CREATE_DATE) AS CREATE_DAY,
            MONTH(A.CREATE_DATE) AS CREATE_MONTH,
            YEAR(A.CREATE_DATE) AS CREATE_YEAR
        FROM SOURCE_TABLE A
        LEFT JOIN TARGET_TABLE_TASK B
        ON A.ID = B.ID
        WHERE B.ID IS NULL;

SHOW TASKS; -- Task will show as suspended.
ALTER TASK MY_TARGET_TABLE_TASK RESUME; -- Task started/resumed.
-- Check the status of a task, including execution history.
SELECT * FROM TABLE (INFORMATION_SCHEMA.TASK_HISTORY(TASK_NAME => 'MY_TARGET_TABLE_TASK'));
INSERT INTO SOURCE_TABLE VALUES
    (4, 'Jeff', '2025-11-02');  -- Insert new row to test task execution.
SELECT * FROM TARGET_TABLE_TASK;
```

## Snowflake Time Travel and Fail Safe

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Setting Table Retention Period:

```sql
CREATE OR REPLACE TABLE MY_TABLE
SET DATA_RETENTION_TIME_IN_DAYS = 90;

ALTER TABLE MY_TABLE
SET DATA_RETENTION_TIME_IN_DAYS = 30;
```

Querying, Cloning, or Restoring Table With Time Travel:

```sql
-- Querying
SELECT * FROM MY_TABLE
AT (TIMESTAMP => 'Mon, 01 May 2025 16:20:00 -700'::timestamp)

SELECT * FROM MY_TABLE
BEFORE (STATEMENT => 'sql_query_statement_id')

-- Cloning
CREATE TABLE RESTORED_TABLE CLONE MY_TABLE
AT (TIMESTAMP => 'Mon, 09 May 2015 01:01:00 +300'::timestamp)

CREATE DATABASE RESTORED_DB CLONE MY_DB
BEFORE (STATEMENT => 'sql_query_statement_id')

-- Restoring Objects
UNDROP TABLE/SCHEMA/DATABASE
```

Working With Time Travel:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Create Table With Time Travel Enabled
CREATE OR REPLACE TABLE TIME_TRAVEL_TABLE (
    ID INT,
    NAME STRING
);
INSERT INTO TIME_TRAVEL_TABLE VALUES
    (1, 'John'),
    (2, 'Sam'),
    (3, 'Elon'),
    (4, 'Mark');
SELECT * FROM TIME_TRAVEL_TABLE;

SET RETENTION_TIME = CURRENT_TIMESTAMP; -- Sets the retention time as the current time. The state of the table at this point will be saved.
DELETE FROM TIME_TRAVEL_TABLE WHERE ID = 4; -- 'Accidentally' delete row.
SELECT * FROM TIME_TRAVEL_TABLE
AT (TIMESTAMP => $RETENTION_TIME::timestamp_tz);
SELECT * FROM TIME_TRAVEL_TABLE
BEFORE (STATEMENT => '01c2a6f8-0000-c8ef-0017-3bf700043366'); -- Query ID retrieved from query history under Monitoring.

CREATE OR REPLACE TABLE TIME_TRAVEL_CLONE_TABLE AS (
    SELECT * FROM TIME_TRAVEL_TABLE
    BEFORE (STATEMENT => '01c2a6f8-0000-c8ef-0017-3bf700043366')
); -- Create new table using recovered data.
SELECT * FROM TIME_TRAVEL_CLONE_TABLE;

-- Restoring Data to Original Table
TRUNCATE TABLE TIME_TRAVEL_TABLE;
INSERT INTO TIME_TRAVEL_TABLE
    SELECT * FROM TIME_TRAVEL_CLONE_TABLE;
DROP TABLE TIME_TRAVEL_CLONE_TABLE; -- No longer needed after data has been restored to original table.

-- Using UNDROP to Restore Data
DROP TABLE TIME_TRAVEL_TABLE;
UNDROP TABLE TIME_TRAVEL_TABLE; -- Also works with schemas and databases.
SELECT * FROM TIME_TRAVEL_TABLE;

-- Working With Time Travel
CREATE OR REPLACE TABLE DROP_TABLE (
    ID INT,
    NAME STRING
);
INSERT INTO DROP_TABLE VALUES
    (1, 'John'),
    (2, 'Sam'),
    (3, 'Elon'),
    (4, 'Mark');
SELECT * FROM DROP_TABLE;

DROP TABLE DROP_TABLE;
UNDROP TABLE DROP_TABLE;
SHOW TABLES; -- Used to identify the current retention period for DROP_TABLE. The default 'retention_time' is 1 day. The UNDROP command won't work beyond 1 day.

ALTER TABLE DROP_TABLE
SET DATA_RETENTION_TIME_IN_DAYS = 0;
DROP TABLE DROP_TABLE;
UNDROP TABLE DROP_TABLE; -- Doesn't work because the retention period was set to 0 days.
```

## Snowflake Cloning

Concept review: [[02_technical_prep/Snowflake Review|Snowflake Review]].

Cloning a Database:

```sql
CREATE DATABASE TEST_DB
CLONE PROD_DB;
```

Working With Clones:

```sql
-- Initial Setup
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE SCHEMA MYDB.MYSCHEMA;

-- Clone Database
CREATE OR REPLACE DATABASE MYDB_CLONE
CLONE MYDB;

DROP DATABASE MYDB_CLONE; -- Delete clone after work is completed.
```


## Loading Semi-Structured Data
- [Link to Snowflake documentation](https://docs.snowflake.com/en/user-guide/semistructured-intro).
- Unlike structured data in a CSV file, semi-structured data does not conform to a fixed schema, but does contain tags, labels, or other markers that identify unique entities within the data. Semi-structured data can also be organized hierarchically, while structured data is flat (think of nested JSON vs a single table).
- Snowflake supports the following semi-structured data formats: JSON, Avro, Optimized Row Columnar (ORC), Parquet, and XML.
- Javascript Object Notation (JSON) is a lightweight, text-based data interchange format that can be read by humans and machines. It is commonly used to store and transmit data between servers and web applications.
- JSON uses a syntax based on key-value pairs and arrays. It is not tied to a specific programming language and can be easily parsed and generated by many languages.
- Example JSON:
  ```sql
  {
    "Name": "Alice",
    "Age": 30,
    "City: "New York"
  } 
  ```
- Working With JSON:
  ```sql
  -- Initial Setup
  USE ROLE ACCOUNTADMIN;
  USE WAREHOUSE COMPUTE_WH;

  -- Create JSON Database and Table
  CREATE OR REPLACE DATABASE JSON_DB;
  CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE ( -- Creates table using PUBLIC schema.
    DATA VARIANT -- Data type associated with JSON and other semi-structured data.
  );

  -- Manually Insert JSON Into Table (Not Using Stage)
  INSERT INTO JSON_DB.PUBLIC.JSON_TABLE (DATA) -- Specifies the DATA column of the table.
  SELECT(PARSE_JSON ('{"Name": "Alice", ""Age": 30, "City: "New York"}));
  SELECT * FROM JSON_DB.PUBLIC.JSON_TABLE; 
  ```

### JSON Flattening (Non-Hierarchical)
- Flattening JSON in snowflake involves converting JSON to a format that can be used to store data into a table.
- Retrieving a JSON Attribute From a Table: `SELECT DATA:Name::STRING FROM JSON_DB.PUBLIC.JSON_TABLE;`. You basically use the format `column_name:attribute_name`. The `::STRING` casts the value as a string since snowflake cannot derive the data type from the JSON.
- Flattening Non-Hierarchical JSON:
  ```sql
  CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE_FLATTENED AS
    SELECT
      DATA:Name:STRING AS NAME,
      DATA:City::STRING AS CITY,
      DATA:Age::INTEGER AS AGE
    FROM JSON_DB.PUBLIC.JSON_TABLE;
  SELECT * FROM JSON_DB.PUBLIC.JSON_TABLE_FLATTENED;
  ```

### JSON Flattening (Hierarchical)
```sql
CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE1 (
  DATA VARIANT
);

INSERT INTO JSON_DB.PUBLIC.JSON_TABLE1 (DATA)
  SELECT PARSE_JSON ('
    {
      "name": "Jane Doe",
      "age": 25,
      "isStudent": true,
      "address": {
        "street": "456 Oak St",
        "city": "Somewhere",
        "zip": "67890"
      }
    }
  ');

CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE1_FLAT AS (
  SELECT
    DATA:name::STRING AS name,
    DATA:age::INTEGER AS age,
    DATA:isStudent::BOOLEAN AS is_student,
    DATA:address:street::STRING AS address_street, -- The format used to access top-level attributes is also used with nested attributes.
    DATA:address:city::STRING AS address_city,
    DATA:address:zip::STRING AS address_zip
  FROM JSON_DB.PUBLIC.JSON_TABLE1
);
SELECT * FROM JSON_DB.PUBLIC.JSON_TABLE1_FLAT;
```

### JSON Flattening (Non-Hierarchical Array)
```sql
CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE2 (
  DATA VARIANT
);

INSERT INTO JSON_DB.PUBLIC.JSON_TABLE2 (DATA)
  SELECT PARSE_JSON ('
    {
      "name": "John Doe",
      "age": 28,
      "isStudent": false,
      "hobbies": ["Reading", "Cycling", "Cooking"]
    }
  ');

CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE2_FLAT AS (
  SELECT
    DATA:name::STRING AS name,
    DATA:age::INTEGER AS age,
    DATA:isStudent:BOOLEAN AS is_student,
    hobby.value::STRING AS hobby
  FROM JSON_DB.PUBLIC.JSON_TABLE2,
  LATERAL FLATTEN (INPUT => DATA:hobbies) AS hobby
);
```
- The `LATERAL FLATTEN` function flattens the array into individual rows, creating three rows for John Doe where all values are the same except hobby. It does so by creating `INDEX` and `VALUE` columns to denote the position and value of each element in the JSON array.

### JSON Flattening (Hierarchical Array)
```sql
CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE3 (
  DATA VARIANT
);

INSERT INTO JSON_DB.PUBLIC.JSON_TABLE3 (DATA)
  SELECT PARSE_JSON ('
    {
      "student": {
        "courses": [
          {
            "course_name": "Mathematics",
            "details": {
              "credits": 3,
              "grade": "A"
            }
          },
          {
            "course_name": "Physics",
            "details": {
              "credits": 4,
              "grade": "B"
            }
          }
        ],
        "id": 1,
        "info": {
          "age": 20,
          "name": "Alice Johnson"
        }
      }
    }
  ');

CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE3_FLAT AS (
  SELECT
    DATA:student:id::INTEGER AS student_id,
    DATA:student:info:name::STRING AS student_name,
    DATA:student:info:age::INTEGER AS student_age,
    courses.value:course_name::STRING AS course:name,
    courses.value:details:credits::INTEGER AS course_credits.
    courses.value:details:grade::STRING AS course_grade
  FROM JSON_DB.PUBLIC.JSON_TABLE3,
  LATERAL FLATTEN (INPUT => DATA:student:courses) AS courses
);
```

### JSON Flattening (Hierarchical Multiple Array)
```sql

CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE4 (
  DATA VARIANT
);

INSERT INTO JSON_DB.PUBLIC.JSON_TABLE4 (DATA)
  SELECT PARSE_JSON ('
    {
      "company": {
        "department": {
          "head": "Jane Smith",
          "name": "Engineering"
        },
        "id": 201,
        "teams": [
          {
            "members": [
              {
                "name": "Alice Brown",
                "role": "UI Developer",
                "skills": [
                  {
                    "experience": 10,
                    "technology": [
                      "HTML",
                      "CSS",
                      "JavaScript",
                      "React"
                    ]
                  }
                ]
              },
              {
                "name": "Bob Green",
                "role": "UX Designer",
                "skills": [
                  {
                    "experience": 7,
                    "technology": [
                      "Figma",
                      "Sketch",
                      "Adobe XD",
                      "Illustrator"
                    ]
                  }
                ]
              }
            ],
            "team_name": "Frontend"
          },
          {
            "members": [
              {
                "name": "Charlie White",
                "role": "Backend Engineer",
                "skills": [
                  {
                    "experience": 15,
                    "technology": [
                      "Python",
                      "Django",
                      "PostgreSQL",
                      "Redis"
                    ]
                  }
                ]
              },
              {
                "name": "David Black",
                "role": "DevOps Engineer",
                "skills": [
                  {
                    "experience": 10,
                    "technology": [
                      "AWS",
                      "Docker",
                      "Kubernetes",
                      "Terraform"
                    ]
                  }
                ]
              }
            ],
            "team_name": "Backend"
          }
        ]
      }
    }
  ');

CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE4_FLAT AS (
  SELECT
    DATA:company:id::INTEGER AS company_id
    DATA:company:department:name::STRING AS department_name,
    DATA:company:department:head::STRING AS department_head,
    teams.value:team_name::STRING AS team_name,
    members.value:name::STRING AS member_name,
    members.value:role::STRING AS member_role,
    skills.value:experience::INTEGER AS member_experience
    technology.value::STRING AS member_technology -- Shows each technology in the array in a separate row.
  FROM JSON_DB.PUBLIC.JSON_TABLE4,
  LATERAL FLATTEN (INPUT => DATA:company:teams) AS teams,
  LATERAL FLATTEN (INPUT => teams.value:members) AS members.
  LATERAL FLATTEN (INPUT => members.value:skills) AS skills,
  LATERAL FLATTEN (INPUT => skills.value:technology) AS technology
);
```
- Each time you reference `flattened_json_array.value` in the `SELECT` statement, it represents a single element in that array. If there are individual, non-array JSON fields in that value, they are referenced using the colon notation (i.e. `flattened_json_array.value:attribute`).
- This table represents a flattened, one-dimensional view of the nested JSON.
- The `FLATTEN` function explodes compound values into multiple rows and returns a set of rows with specific columns, including `VALUE`, which is referenced in the `SELECT` statement.
- The `LATERAL` keyword allows the `FLATTEN` function to access data from the "outer" table (the main table being queried) on a row-by-row basis.
- The reason there's a comma after `JSON_DB.PUBLIC.JSON_TABLE3` is because the output of the `LATERAL FLATTEN` function is treated as its own table. It's not a clause like `GROUP BY` or `WHERE`.

### JSON Flattening (Multiple JSON Array)
```sql
CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE5 (
  DATA VARIANT
);

INSERT INTO JSON_DB.PUBLIC.JSON_TABLE5 (DATA)
  SELECT PARSE_JSON ('
    [
      {
        "id": 1,
        "name": "Alice Johnson",
        "position": "Software Engineer",
        "department": "IT",
        "salary": 75000
      },
      {
        "id": 2,
        "name": "Bob Smith",
        "position": "Product Manager",
        "department": "Product",
        "salary": 85000
      },
      {
        "id": 3,
        "name": "Carol Lee",
        "position": "UX Designer",
        "department": "Design",
        "salary": 70000
      }
    ]
  ');

CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE5_FLAT AS (
  SELECT
    employees.value:id::INTEGER AS employee_id,
    employees.value:name::STRING AS employee_name,
    employees.value:position::STRING AS employee_position,
    employees.value:department::STRING AS employee_department,
    employees.value:salary::INTEGER AS employee_salary
  FROM JSON_DB.PUBLIC.JSON_TABLE5,
  LATERAL FLATTEN (INPUT => DATA) AS employees
);
```

### JSON Flattening (Multiple JSON Keys)
```sql
CREATE OR REPLACE TABLE JSON_DB.PUBLIC.JSON_TABLE6 (
  DATA VARIANT
);

INSERT INTO JSON_DB.PUBLIC.JSON_TABLE6 (DATA)
  SELECT PARSE_JSON ('
    {
      "university": {
        "Faculty of Engineering": {
          "dean": "Dr. Emily Carter",
          "departments": {
            "Computer Science": {
              "head": "Dr. Alan Turing",
              "courses": {
                "Algorithms": {
                  "credits": 4,
                  "enrolled_students": 120
                },
                "Machine Learning": {
                  "credits": 3,
                  "enrolled_students": 90
                }
              }
            },
            "Mechanical Engineering": {
              "head": "Dr. Grace Hopper",
              "courses": {
                "Thermodynamics": {
                  "credits": 4,
                  "enrolled_students": 80
                },
                "Fluid Mechanics": {
                  "credits": 3,
                  "enrolled_students": 75
                }
              }
            }
          },
          "total_students": 1000,
          "total_courses": 4
        },
        "Faculty of Science": {
          "dean": "Dr. Carl Sagan",
          "departments": {
            "Physics": {
              "head": "Dr. Richard Feynman",
              "courses": {
                "Quantum Mechanics": {
                  "credits": 4,
                  "enrolled_students": 70
                },
                "Classical Mechanics": {
                  "credits": 4,
                  "enrolled_students": 80
                }
              }
            },
            "Biology": {
              "head": "Dr. Jane Goodall",
              "courses": {
                "Genetics": {
                  "credits": 3,
                  "enrolled_students": 100
                },
                "Ecology": {
                  "credits": 3,
                  "enrolled_students": 90
                }
              }
            }
          },
          "total_students": 700,
          "total_courses": 4
        }
      },
      "name": "Central University",
      "total_students": 1700,
      "total_courses": 8
    }
  ');

CREATE OR REPLACE JSON_DB.PUBLIC.JSON_TABLE6_FLAT AS (
  SELECT
    DATA:name::STRING AS university_name
    DATA:total_courses::INTEGER AS university_total_courses
    DATA:total_students::INTEGER AS university_total_students,
    DATA:university.key::STRING AS faculty_name,
    DATA:university.value:dean::STRING AS faculty_dean_name,
    DATA:university.value:total_courses::INTEGER AS faculty:total_courses,
    DATA:university.value:total_students::INTEGER AS faculty_total_students,
    DATA:departments.key::STRING AS department_name,
    DATA:departments.value:head::STRING AS department_head_name,
    DATA:courses.key::STRING AS course_name
    DATA:courses.value:credits::INTEGER AS course_credits,
    DATA:courses.value::enrolled_students::INTEGER AS course_enrolled_students
  FROM JSON_DB.PUBLIC.JSON_TABLE6,
  LATERAL FLATTEN (INPUT => DATA:university) AS university,
  LATERAL FLATTEN (INPUT => university.value:departments) AS departments,
  LATERAL FLATTEN (INPUT => departments.value:courses) AS courses
);
```
- The `LATERAL FLATTEN` function can be used to flatten arrays, as well as nested JSON. Nested JSON needs to be flattened when one JSON objects contains multiple keys that represent a JSON object.
- The university key contains multiple keys that each represent a JSON object, which is why it need to be flattened.
- When the object is flattened, it returns two keys representing the faculty of engineering and faculty of science. The key represents the faculty name. The value of each key represents the details of each faculty.
- Within the faculty value, there is `departments`, which also contains multiple JSON objects. When `departments` is flattened, the key represents the name and the value represents the details.
- Within the department values, there is `courses`, which also contains multiple JSON objects. When `courses` is flattened, the key represents the name and the value represents the details.
- In summary, flattening is used to flatten out keys that represent arrays, or keys that contain multiple JSON objects. When an array is flattened, you typically just need the value (represents array elements). When a nested JSON object is flattened, you need the key to get the name of each nested JSON object and value to get the details of that object.

## Example data files

- [[02_technical_prep/snowflake/customers.csv|customers.csv]]
- [[02_technical_prep/snowflake/event_1.json|event_1.json]]
- [[02_technical_prep/snowflake/event_2.json|event_2.json]]
- [[02_technical_prep/snowflake/event_3.json|event_3.json]]
- [[02_technical_prep/snowflake/students.csv|students.csv]]
- [[02_technical_prep/snowflake/users.csv|users.csv]]
