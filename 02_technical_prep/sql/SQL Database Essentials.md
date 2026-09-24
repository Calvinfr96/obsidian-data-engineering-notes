---
aliases:
  - "sql_database_essentials"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Notes/sql_database_essentials.md"
imported: 2026-09-24
---

# SQL Database Essentials

Review: [[02_technical_prep/sql/SQL Optimization Principles|SQL Optimization Principles]]

## Contents

- [[#What is a Database]]
- [[#Key Database Concepts: SQL Foundations]]
- [[#Introducing SQL: Structured Query Language]]
- [[#DDL Basics in SQL]]
- [[#DML Basics in SQL]]

## What is a Database
- Data is raw facts and figures. It can take the form of numbers, text, dates, images, etc.
- Data becomes useful when it is organized and structured.
- A database is a structured collection of data that allows for efficient storage, access, and management.
- A real-world analogy of a database would be a bookstore.
  - Here, the store is the database, the shelves are tables, the books are records (rows), and the book details, such as author, title, and genre, are fields (columns).
  - Another way of looking at a table is like a collection of items. Each row is an item and each column is a specific detail about that item.
- Just as a well-designed bookstore makes it easy to find the books you need, a well-designed database makes it easy to find the data you need.
- Databases are helpful because they help keep data clean, organized, and scalable and make it easy to update. They also help to reduce duplication and errors.
  - They also help to enable powerful searching and filtering and enable multi-user access.
- A **Relational Database** is a special kind of database that stores data in related tables, where each table focuses on a single entity.
  - Tables in relational databases are linked using keys. For example:
    - In a Bookstore (database), there could be a Customers, Books, and Orders Table. Each table has its own ID.
    - The Orders table relates to the Customers and Books Table by containing a column for Customer ID and Book ID, in addition to Order ID and Date.
  - Relational databases work so well because they help keep data normalized (store only once), enforce consistency, allow for flexible querying via SQL, and are secure and efficient.

## Key Database Concepts: SQL Foundations
- A table is a core unit in a database. A table is like a spreadsheet. In a table, rows represent records (individual data entries) and columns represent fields (types of data).
  - For example, in a Students Table, each row would represent a student and each column would represent a piece of information about that student, such as ID, name, or age.
- A Primary Key is a column or a combination of columns that uniquely identifies a row. **No two rows can have the same Primary Key.** In the Students Table example above, Student ID would be the Primary Key.
- A Foreign Key is a column that references a Primary Key from another table. Foreign Keys are used to link related data across tables.
  - For example, in a Courses Table, Student ID could be a Foreign Key while Course ID would be the Primary Key.
  - Another example would be an Enrollments Table, where Student ID and Course ID would be Foreign Keys and Enrollment ID would be the Primary Key.

### Common Data Types
- The different data types in SQL represent different ways of describing data. Common SQL data types include:
- INTEGER: Whole numbers (e.g. age).
- VARCHAR: Text (e.g. name).
- DATE: Calendar dates (e.g. birthdate).

## Introducing SQL: Structured Query Language
- SQL is a "querying" language (**NOT a programming language**) that is used to "talk" to databases. It's used to retrieve, insert, update, and delete data.
  - SQL has simple syntax and is designed to be human readable.
- SQL is **querying**, not programming:
  - There are no loops or functions.
  - You're not building something, you're asking questions.
  - It's designed for **data manipulation**, not software logic.
- The two main flavors of SQL are Data Manipulation (DML) and Data Definition (DDL).
  - **DML is what is primarily used in Data Engineering.** It is how data engineers interact with data inside tables. Examples include SELECT, INSERT, UPDATE, and DELETE.
  - DDL is used to construct the data environment and define the structure of the database. Examples include CREATE, ALTER, and DROP.
  - Using the Bookstore analogy, DDL is like building the shelves and DML is like placing books on the shelves.
  - The following four commands form the core of DML:
    - SELECT: Retrieve data.
    - INSERT: Add new data.
    - UPDATE: Change existing data.
    - DELETE: Remove data.
  - The following three commands form the core of DDL:
    - CREATE TABLE: Make a new table.
    - ALTER TABLE: Change the structure of a table.
    - DROP TABLE: Remove a table.
    - These three functions are just enough to understand the environment.
- Example SQL Commands:
  - `SELECT name FROM customers_prc` gives the name of each record in the `customers_prc` table (the list of customer names).

## DDL Basics in SQL
- DDL stands for Data Definition Language and forms the backbone of any database. The most common DDL operations are CREATE, ALTER, and DROP. These define what your tables look like, what type of data they store, and how the schema is built.
  - CREATE TABLE:
    - Used to define a new table. Must include the table name, column names, and data types. It can also include constraints, such as `PRIMARY KEY`.
    - Example:
      ```sql
      CREATE TABLE Students (
        ID INT PRIMARY KEY,
        Name VARCHAR(50),
        Major VARCHAR(30)
      );
      ```
    - In the above example, the `INT` datatype represents whole numbers and the `VARCHAR(n)` datatype represents text with a maximum length. As in Typescript with variables, columns are defined by specifying the name of column first, then the data type, then any constraints.
  - ALTER TABLE:
    - Used to add, remove or change columns in a table. For example, adding an Email Column to the the Students .
    - Example:
      ```sql
      ALTER TABLE Students ADD Email VARCHAR(100);
      ```
  - DROP TABLE:
    - **Permanently** removes a table and its data, **use with extreme caution**.
    - Example:
      ```sql
      DROP TABLE Students;
      ```
- DDL is often used by database engineers or administrators. Data analysts mostly use DML
- Knowing DDL helps you understand data structure and communicate with DB teams.
- Employees Table Example:
  ```sql
  CREATE TABLE Employees (
    Employee_ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Department VARCHAR(50),
    Hire_Date DATE
  );
  ```
  - Notice, when defining columns, you specify the column name, then the data type, then any constraints. Constraints can include whether the column is a primary or foreign key.
  - Similar to some programming languages, you end statements with a semicolon.

## DML Basics in SQL
- DML stands for Data Manipulation Language. The core DML operations in SQL are `INSERT`, `UPDATE`, and `DELETE`. These are used to add, change, and delete data from tables.
- `INSERT`:
  - Used to add new data (rows) to a table according to its pre-defined schema.
  - Syntax (Inserting a single row):
    ```sql
    INSERT INTO Students (ID, Name, Major)
    VALUES (1, 'Alice', 'Biology');
    ```
  - Syntax (Inserting multiple rows):
    ```sql
    INSERT INTO Students (ID, Name, Major) VALUES
    (2, 'Bob', 'History'),
    (3, 'Cathy', 'Math');
    ```
  - Key Notes:
    - Always match data types defined in the table schema.
    - Always specify the columns of the rows being added.
    - Always ensure **unique** Primary Key values.
- `UPDATE`:
  - Used to modify existing rows in a table.
  - Syntax (Updating a single field):
    ```sql
    UPDATE Students SET Major = 'Physics' WHERE ID = 1;
    ```
  - Syntax (Updating multiple fields):
    ```sql
    UPDATE Students SET Major = 'Chemistry', Name = 'Alicia' WHERE ID = 1;
    ```
  - Key Notes:
    - **Always** clause `WHERE`, specifying a condition, to avoid affecting all rows.
- `DELETE`:
  - Used to remove records from a table.
  - Syntax:
    ```sql
    DELETE FROM Students WHERE ID = 1;
    ```
  - Key Notes:
    - **Always** include the `WHERE` clause.
    - Records containing foreign keys cannot be removed until the connection between records is eliminated. For example, if Student ID is used as a Foreign Key in the Enrollments Table, then that student cannot be deleted until the enrollment is deleted.
- `SELECT`:
  - Used to select specific rows in a table.
  - Syntax (Selecting specific columns):
    ```sql
    SELECT Major, Name FROM Students
    WHERE ID = 1;
    ```
  - Syntax (Selecting all columns):
    ```sql
    SELECT * FROM Students
    WHERE ID = 1;
    ```
- Key Takeaways
  - Never forget the `WHERE` clause in `UPDATE` and `DELETE` operations.
  - Always verify an `UPDATE` or `DELETE` operation by using to `SELECT` first to ensure you're performing the operation on the correct data. This will help to ensure the `WHERE` clause is targeting the correct data.
