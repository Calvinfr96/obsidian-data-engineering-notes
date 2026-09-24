---
aliases:
  - "sql_beginner_lessons.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Practice Code/sql_beginner_lessons.sql"
imported: 2026-09-24
---

# SQL Beginner Lesson Exercises

Review: [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#Question 1: SELECT FROM]]
- [[#Question 2: WHERE]]
- [[#Question 3: COMPARISON OPERATORS]]
- [[#Question 4: LOGICAL OPERATORS]]
- [[#Question 5: LIKE OPERATOR]]
- [[#Question 6: IN OPERATOR]]
- [[#Question 7: BETWEEN OPERATOR]]
- [[#Question 8: IS NULL]]
- [[#Question 9: AND OPERATOR]]
- [[#Question 10: OR OPERATOR]]
- [[#Question 11: NOT OPERATOR]]
- [[#Question 12: ORDER BY]]
- [[#Question 13: LIMIT OFFSET]]

## Question 1: SELECT FROM

### Prompt

```text
Retrieve the email of customers with aliases "Customer_Email".

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar

  Sample Input:
  customers 
  | customer_id | customer_name | age | city | email                | address              |
  |-------------|---------------|-----|------|----------------------|----------------------|
  | 1           | John Doe      | 30  | NYC  | john.doe@email.com   | 123 Main Street      |
  | 2           | Jane Smith    | 25  | LA   | jane.smith@email.com | 456 Oak Avenue       |
  | 3           | Mike Johnson  | 35  | CHI  | mike.johnson@email.com| 789 Pine Boulevard  |
```

### Solution and working notes

```sql
SELECT
  email AS Customer_Email
FROM customers; -- Correct!
```

## Question 2: WHERE

### Prompt

```text
Find out the employees working in 'finance' department with salary at least 70000. Display employee_id, first_name, last_name of the employee.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int

  Sample Input:
  employees
  | employee_id | first_name | last_name | department | salary | manager_id |
  |-------------|------------|-----------|------------|--------|------------|
  | 1           | John       | Doe       | IT         | 60000.00| 3         |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4         |
  | 3           | Mike       | Johnson   | IT         | 80000.00| 5         |
  | 4           | Emily      | Davis     | Finance    | 70000.00| 5         |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL      |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name
FROM employees
WHERE department = 'Finance' AND salary >= 70000 -- Correct!
```

## Question 3: COMPARISON OPERATORS

### Prompt

```text
Find out the product having unit price at least 150 and unit price not more than 700. Display the product_name.

  Table: products
  ColumnName      DataType
  product_id      int
  product_name    varchar
  category        varchar
  unit_price      decimal

  Sample Input:
  products  
  | product_id | product_name | category  | unit_price |
  |------------|--------------|-----------|------------|
  | 1          | Laptop       | Electronics | 800.00   |
  | 2          | Smartphone   | Electronics | 500.00   |
  | 3          | Coffee Maker | Appliances  | 50.00    |
  | 4          | Blender       | Appliances | 200.00   |
  | 5          | Headphones    | Electronics | 120.00  |
```

### Solution and working notes

```sql
SELECT
  product_name
FROM products
WHERE unit_price >= 150 AND unit_price <= 700; -- Correct!
```

## Question 4: LOGICAL OPERATORS

### Prompt

```text
Retrieve the employee_id, first_name, and last_name of employees who either have a salary greater than 70,000 or are in the 'Finance' department.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int

  Sample Input:
  employees
  | employee_id | first_name | last_name | department | salary | manager_id |
  |-------------|------------|-----------|------------|--------|------------|
  | 1           | John       | Doe       | IT         | 60000.00| 3         |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4         |
  | 3           | Mike       | Johnson   | IT         | 80000.00| 5         |
  | 4           | Emily      | Davis     | Finance    | 70000.00| 5         |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL      |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name
FROM employees
WHERE salary > 70000 OR department = 'Finance'; -- Correct!
```

## Question 5: LIKE OPERATOR

### Prompt

```text
Retrieve the product_id, product_name, and category of products whose names end with the letter "e" and belong to either the "Electronics" or "Appliances" category.

  Table: products 
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal

  Sample Input:
  products
  | product_id | product_name | category  | unit_price |
  |------------|--------------|-----------|------------|
  | 1          | Laptop       | Electronics | 800.00   |
  | 2          | Smartphone   | Electronics | 500.00   |
  | 3          | Coffee Maker | Appliances  | 50.00    |
  | 4          | Blender       | Appliances | 200.00   |
  | 5          | Headphones    | Electronics | 120.00  |
```

### Solution and working notes

```sql
SELECT
  product_id,
  product_name,
  category
FROM products
WHERE product_name LIKE "%e" AND (category = "Electronics" OR category = "Appliances"); -- Correct!
```

## Question 6: IN OPERATOR

### Prompt

```text
Retrieve the product_id, product_name, and unit_price of products that belong to either the "Electronics" or "Appliances" category.

  Table: products 
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal

  Sample Input:
  products
  | product_id | product_name | category  | unit_price |
  |------------|--------------|-----------|------------|
  | 1          | Laptop       | Electronics | 800.00   |
  | 2          | Smartphone   | Electronics | 500.00   |
  | 3          | Coffee Maker | Appliances  | 50.00    |
  | 4          | Blender       | Appliances | 200.00   |
  | 5          | Headphones    | Electronics | 120.00  |
```

### Solution and working notes

```sql
SELECT
  product_id,
  product_name,
  unit_price
FROM products
WHERE category IN ("Electronics", "Appliances"); -- Correct!
```

## Question 7: BETWEEN OPERATOR

### Prompt

```text
Retrieve the order_id, order_date, and order_amount of orders placed between January 1, 2023, and June 30, 2023.

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar

  Sample Input:
  orders 
  | order_id | order_date | order_amount | customer_id | order_status | shipping_address  |
  |----------|------------|--------------|-------------|--------------|-------------------|
  | 1        | 2023-03-15 | 100.00       | 101         | Pending      | 123 Main Street   |
  | 2        | 2023-05-20 | 150.00       | 102         | Shipped      | 456 Oak Avenue    |
  | 3        | 2023-02-10 | 200.00       | 103         | Completed    | 789 Pine Boulevard|
  | 4        | 2023-06-25 | 80.00        | 104         | Pending      | 101 Elm Lane      |
  | 5        | 2023-04-05 | 120.00       | 105         | Shipped      | 202 Maple Street  |
```

### Solution and working notes

```sql
SELECT
  order_id,
  order_date,
  order_amount
FROM orders
WHERE order_date BETWEEN '2023-01-01' AND '2023-06-30'; -- Correct!
```

## Question 8: IS NULL

### Prompt

```text
Retrieve the order_id and order_date as "date" of orders where the "customer_id" is missing or null.

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar

  Sample Input:
  orders 
  | order_id | order_date | order_amount | customer_id | order_status | shipping_address  |
  |----------|------------|--------------|-------------|--------------|-------------------|
  | 1        | 2023-03-15 | 100.00       | 101         | Pending      | 123 Main Street   |
  | 2        | 2023-05-20 | 150.00       | NULL        | Shipped      | 456 Oak Avenue    |
  | 3        | 2023-02-10 | 200.00       | 103         | Completed    | 789 Pine Boulevard|
  | 4        | 2023-06-25 | 80.00        | NULL        | Pending      | 101 Elm Lane      |
  | 5        | 2023-04-05 | 120.00       | 105         | Shipped      | 202 Maple Street  |
```

### Solution and working notes

```sql
SELECT
  order_id,
  order_date AS date -- Apparently, no error is thrown for using date (a keyword) as an alias without using quotes.
FROM orders
WHERE customer_id IS NULL; -- Correct!
```

## Question 9: AND OPERATOR

### Prompt

```text
Retrieve the order_id, order_date and order_amount of orders with a "Status" of "Processing" and a "order_amount" greater than 300.

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar

  Sample Input:
  orders  
  | order_id | order_date | order_amount | customer_id | order_status | shipping_address  |
  |----------|------------|--------------|-------------|--------------|-------------------|
  | 1        | 2023-03-15 | 100.00       | 101         | Pending      | 123 Main Street   |
  | 2        | 2023-05-20 | 400.00       | 102         | Processing   | 456 Oak Avenue    |
  | 3        | 2023-02-10 | 200.00       | 103         | Completed    | 789 Pine Boulevard|
  | 4        | 2023-06-25 | 500.00       | 104         | Processing   | 101 Elm Lane      |
  | 5        | 2023-04-05 | 120.00       | 105         | Shipped      | 202 Maple Street  |
```

### Solution and working notes

```sql
SELECT
  order_id,
  order_date,
  order_amount
FROM orders
WHERE order_status = "Processing" AND order_amount > 300; -- Correct!
```

## Question 10: OR OPERATOR

### Prompt

```text
Retrieve the employee_id, first_name, last_name, and salary of employees who work in the "Sales" department or have a salary greater than 60000.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int

  Sample Input:
  employees  
  | employee_id | first_name | last_name | department | salary | manager_id |
  |-------------|------------|-----------|------------|--------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3         |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4         |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5         |
  | 4           | Emily      | Davis     | Finance    | 70000.00| 5         |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL      |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name,
  salary
FROM employees
WHERE department = "Sales" OR salary > 60000; -- Correct!
```

## Question 11: NOT OPERATOR

### Prompt

```text
Retrieve the employee_id, first_name, last_name, and department of employees who are not in the "Marketing" or "IT" departments.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int

  Sample Input:
  employees 
  | employee_id | first_name | last_name | department | salary | manager_id |
  |-------------|------------|-----------|------------|--------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3         |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4         |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5         |
  | 4           | Emily      | Davis     | IT         | 70000.00| 5         |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL      |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name,
  department
FROM employees
WHERE department NOT IN ("Marketing", "IT"); -- Correct!

-- This also works:
SELECT
  employee_id,
  first_name,
  last_name,
  department
FROM employees
WHERE NOT department = "Marketing" OR department = "IT"; -- Note: You don't need to use parentheses here to group the OR statement.
```

## Question 12: ORDER BY

### Prompt

```text
IMPORTANT NOTE: When checking your answer for queries involving ORDER BY, make sure to  manually compare your output against against 
  the expected output. The question may be marked as correct even if the rows of the result set are not in the correct order.

  Retrieve the product_id, product_name, category and unit_price of products. Order it by products higher to lower prices.

  Table: products 
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal

  Sample Input:
  products 
  | product_id | product_name | category     | unit_price  |
  |------------|--------------|--------------|-------------|
  | 1          | Laptop       | Electronics  | 800.00      |
  | 2          | Smartphone   | Electronics  | 500.00      |
  | 3          | Coffee Maker | Appliances   | 50.00       |
  | 4          | Blender      | Appliances   | 200.00      |
  | 5          | Headphones   | Electronics  | 120.00      |
```

### Solution and working notes

```sql
SELECT
  product_id,
  product_name,
  category,
  unit_price
FROM products
ORDER BY unit_price DESC; -- Correct!
```

## Question 13: LIMIT OFFSET

### Prompt

```text
Select the employee with 3rd highest salary. Output should be only first name of the employee

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int

  Sample Input:
  employees
  | employee_id | first_name | last_name | department | salary | manager_id |
  |-------------|------------|-----------|------------|--------|------------|
  | 1           | John       | Doe       | IT         | 5000.00| 3          |
  | 2           | Jane       | Smith     | Finance    | 6000.00| 4          |
  | 3           | Mike       | Johnson   | IT         | 7000.00| 5          |
  | 4           | Emily      | Davis     | Finance    | 5500.00| 5          |
  | 5           | Robert     | Brown     | HR         | 10000.00| NULL      |
```

### Solution and working notes

```sql
SELECT
  first_name
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2; -- Correct!
```
