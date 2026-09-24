---
aliases:
  - "sql_advanced_lessons.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Practice Code/sql_advanced_lessons.sql"
imported: 2026-09-24
---

# SQL Advanced Lesson Exercises

Review: [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#Question 1: DATA TYPES]]
- [[#Question 2: CONCAT]]
- [[#Question 3: CAST]]
- [[#Question 4: LENGTH]]
- [[#Question 5: SUBSTRING]]
- [[#Question 6: CHARINDEX OR SUBSTRING_INDEX]]
- [[#Question 7: TRIM]]
- [[#Question 8: LEFT & RIGHT]]
- [[#Question 9: UPPER & LOWER]]
- [[#Question 10: EXTRACT]]
- [[#Question 11: COALESCE]]
- [[#Question 12: SUBQUERY IN CONDITION]]
- [[#Question 13: WITH STATEMENTS (CTEs)]]
- [[#Question 14: WINDOW FUNCTIONS]]
- [[#Question 15: WINDOW FUNCTION WITH AGGREGATE FUNCTION]]
- [[#Question 16: ROW NUMBER]]
- [[#Question 17: RANK]]
- [[#Question 18: LEAD]]
- [[#Question 19: LAG]]

## Question 1: DATA TYPES

### Prompt

```text
Calculate the number of days between each order_date and '2023-06-01' as "days_since_order". Also display the customer_id, customer_name, order_id and order_date for customers only who have placed orders.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
SELECT
  c.customer_id,
  c.customer_name,
  o.order_id,
  o.order_date,
  DATEDIFF('2023-06-01', o.order_date) AS days_since_order
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id; -- Correct!
```

## Question 2: CONCAT

### Prompt

```text
Retrieve customer_name, customer_id and customer address as "full address" as a column using address and city combined one after the other.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar
```

### Solution and working notes

```sql
SELECT
  customer_id,
  customer_name,
  CONCAT(address, ', ', city) AS "full address"
FROM customers; -- Correct!
```

## Question 3: CAST

### Prompt

```text
Calculate total amount for orders with order status = 'Completed' and cast as string with '$' prefix. Display column name as "total_amount_string".

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
SELECT
  SUM(order_amount) AS total_amount,
  CAST(total_amount AS CONCAT('$', total_amount))
FROM orders
WHERE order_status = 'Completed'; -- Incorrect

SELECT
  CONCAT('$', CAST(SUM(order_amount) AS CHAR)) AS total_amount_string
FROM orders
WHERE order_status = 'Completed'; -- Correct!
-- Previously thought that they weren't, but nested functions are allowed in SQL.
-- The final result we want is a composite string, so the outermost function should be CONCAT.
-- Next, we need to CAST the total order amount as a CHAR. To get the total, we use SUM within CAST.
-- When using CAST, the data type specified after AS and needs to be a supported SQL data type, not a custom value.
```

## Question 4: LENGTH

### Prompt

```text
Retrieve order_id, order_date and order_amount along with customer_name and customer_id where customer name is of odd length for all customers who ordered.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
SELECT
  c.customer_id,
  c.customer_name,
  o.order_id,
  o.order_date,
  o.order_amount,
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id
WHERE MOD(LENGTH(customer_name), 2) <> 0; -- Correct!
-- You could also use the % operator as follows: LENGTH(customer_name) % 2 <> 0.
```

## Question 5: SUBSTRING

### Prompt

```text
Retrieve the first 5 characters of categories for all products. Display product_id, product_name and a new column with first 5 characters AS first_5_chars. 

  Table: products 
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal
```

### Solution and working notes

```sql
SELECT
  product_id,
  product_name,
  SUBSTRING(category, 1, 5) AS first_5_chars
FROM products; -- Correct!
```

## Question 6: CHARINDEX OR SUBSTRING_INDEX

### Prompt

```text
Exercise: Retrieve the customer_name along with their email as "EmailDomain".
  Note: You may use 'SUBSTRING_INDEX' here to solve the question which is MYSQL compatible.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar
```

### Solution and working notes

```sql
SELECT
  customer_name,
  SUBSTRING(
    email,
    CHARINDEX('@', email) + 1,
    LENGTH(email)
  )AS email_domain
FROM customers; -- Incorrect

SELECT
  customer_name,
  SUBSTRING_INDEX(email, '@', -1) AS email_domain
FROM customers; -- Solution (CHARINDEX doesn't work in the code editor on the DEA website).
-- If CHARINDEX did work, we could have used SUBSTRING with CHARINDEX to get the email domain as follows:
-- SUBSTRING(email, CHARINDEX('@') + 1, LENGTH(email));
```

## Question 7: TRIM

### Prompt

```text
Retrieve customer information including customer_id, customer_name as "FullName", trimming the address as TrimmedAddress and City.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar
```

### Solution and working notes

```sql
SELECT
  customer_id,
  customer_name AS FullName,
  TRIM(address) AS TrimmedAddress,
  city
FROM customers; -- Correct!
-- 'TRIM(TRAILING ' ' FROM address) AS TrimmedAddress' also works. The question doesn't specify whether to trim leading, trailing or both.
-- TRIM(string) assumes you want to trim spaces from both ends of the string.
```

## Question 8: LEFT & RIGHT

### Prompt

```text
Retrieve three letter short form of all departments starting from first letter as "ShortDeptName", alongside display employee_id and first_name & last_name combined as "FullName".

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int
```

### Solution and working notes

```sql
SELECT
  employee_id,
  CONCAT(first_name, ' ', last_name) AS FullName,
  LEFT(department, 3) AS ShortDeptName
FROM employees; -- Correct!
```

## Question 9: UPPER & LOWER

### Prompt

```text
Retrieve product_id, converting product_name to uppercase as "ProductNameUpper", category to lowercase as "CategoryLower".

  Input Format: The products table with the following columns:
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal
```

### Solution and working notes

```sql
SELECT
  product_id,
  UPPER(product_name) AS ProductNameUpper,
  LOWER(category) AS CategoryLower
FROM products; -- Correct!
```

## Question 10: EXTRACT

### Prompt

```text
Retrieve all orders displaying order_id, order_status, order_amount and month of order_date as "MonthInWords". Also fetch and display customer_id and customer_name who have ordered.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
SELECT
  o.order_id,
  o.order_amount,
  MONTHNAME(o.order_date) AS MonthInWords,
  -- DATE_FORMAT(STR_TO_DATE(EXTRACT(MONTH FROM o.order_date), '%m'), '%M') AS MonthInWords, (alternative)
  o.order_status,
  c.customer_id,
  c.customer_name
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id; -- Correct!
-- We also could have extracted the numerical month from the order_date and used a CASE Statement to get the string month from that.
-- For example: CASE extracted_numerical_month WHEN 1 THEN 'January' END;
-- You could also use the MONTH function to get the numerical month from a date, instead of extract.
```

## Question 11: COALESCE

### Prompt

```text
Find out all the employee_id, employee first_name as "employee_first_name", their manager first_name as "manager_name" and manager_id. Employees who do not have a manager or they are highest in hierarchy, assign 'Manager' as the columnar value for "manager_name" and "X" as 'manager_id'.

  Input Format: The employees table with the following columns:
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int
```

### Solution and working notes

```sql
SELECT
  e.employee_id,
  e.first_name AS employee_first_name,
  COALESCE(m.employee_id, 'X') AS manager_id,
  COALESCE(m.first_name, 'Manager') AS manager_name
FROM employees e LEFT JOIN employees m
ON e.manager_id = m.employee_id; -- Correct!
-- We join employees with managers (instead of managers with employees) because an employee is much less likely to have a manager than a manager to have no employees.
```

## Question 12: SUBQUERY IN CONDITION

### Prompt

```text
Retrieve product_id, product_name, unit_price and category that have a price higher than the average price of products within their respective categories.

  Table: products 
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal
```

### Solution and working notes

```sql
SELECT
  product_id,
  product_name,
  unit_price,
  category
FROM products
WHERE product_id IN (
  SELECT
    product_id
  FROM products AS allProducts
  WHERE unit_price > (
    SELECT
      AVG(unit_price)
    FROM products AS categoryProducts
    WHERE allProducts.category = categoryProducts.category -- We don't use GROUP BY here because that would return an average for each category.
  )
); -- Correct!

SELECT
  product_id,
  product_name,
  unit_price,
  category
FROM products AS allProducts
WHERE unit_price > (
  SELECT
    AVG(unit_price)
  FROM products categoryProducts
  WHERE allProducts.category = categoryProducts.category
); -- Alternative (less complicated) solution.
```

## Question 13: WITH STATEMENTS (CTEs)

### Prompt

```text
Retrieve department names, the total number of employees in each department, and the average salary, but include only those departments where the average salary exceeds the overall average salary across all employees.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int
```

### Solution and working notes

```sql
WITH OverallSalary AS (
  SELECT
    AVG(salary) AS average_overall_salary
  FROM employees
)

SELECT
  department,
  COUNT(*) AS total_employees,
  AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING average_salary > average_overall_salary; -- Incorrect. We need to reference the CTE somehow in the main query in order to use average_overall_salary.

WITH DepartmentStats AS (
  SELECT
    department,
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary
  FROM employees
  GROUP BY department
),
OverallStats AS (
  SELECT
    AVG(salary) AS average_overall_salary
  FROM employees
)

SELECT
  ds.department,
  ds.total_employees,
  ds.average_salary
FROM DepartmentStats AS ds JOIN OverallStats AS os
ON ds.average_salary > os.average_overall_salary; -- Solution
-- This solution uses CTEs to perform the necessary calculations in stages.
-- In the first stage, employee count and average salary per department is calculated and saved in a CTE.
-- In the second stage, the overall average salary is calculated for all employees and saved in a CTE.
-- Lastly, we select the needed columns from each CTE and join the CTEs on the following condition: ds.average_salary > os.average_overall_salary;.
-- Joining on this condition ensures only records that meet the necessary criteria are returned.
```

## Question 14: WINDOW FUNCTIONS

### Prompt

```text
Calculate the rank of customers as "rank" based on their total order_amount (from highest to lowest). Display customer_id, customer_name, order_amount and rank for all customers who ordered.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address           varchar

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
-- Unable to formulate solution

WITH GroupedCustomers AS (
  SELECT
    c.customer_id AS customer_id,
    c.customer_name AS customer_name,
    SUM(o.order_amount) AS order_amount
  FROM customers c JOIN orders o
  ON c.customer_id = o.customer_id
  GROUP BY o.customer_id
)

SELECT
  customer_id,
  customer_name,
  order_amount,
  RANK() OVER(ORDER BY order_amount DESC) AS 'rank'
FROM GroupedCustomers; -- Solution
```

## Question 15: WINDOW FUNCTION WITH AGGREGATE FUNCTION

### Prompt

```text
Retrieve the first_name, last_name, salary, average departmental salary as "avg_department_salary", and salary difference as "salary_difference" from the average for each employee.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int
```

### Solution and working notes

```sql
WITH AverageDepartmentSalary AS (
  SELECT
    department,
    AVG(salary) AS average_department_salary
  FROM employees
  GROUP BY department
)

SELECT
  first_name,
  last_name,
  salary,
  average_department_salary,
  (salary - average_department_salary) AS salary_difference
FROM employees AS e JOIN AverageDepartmentSalary AS a
ON e.department = a.department; -- Correct!
-- This solution uses a CTE without utilizing window functions.
-- You could also use the 'AVG(salary) OVER(PARTITION BY department)' window function here instead of GROUP BY.
-- This would allow you to retrieve all of the fields in the CTE and avoid joining the CTE in the main query.

SELECT
  first_name,
  last_name,
  salary,
  AVG(salary) OVER(PARTITION BY department) AS average_department_salary,
  (salary - AVG(salary) OVER(PARTITION BY department)) AS salary_difference
FROM employees; -- Correct!
-- The CTE solution is probably more efficient because you need to calculate the average twice.
-- Calculating the average twice is required because the alias can't be referenced in the same SELECT statement where it was defined.
```

## Question 16: ROW NUMBER

### Prompt

```text
Retrieve the top 3 orders with the highest order_amount values, along with their order_id, customer_id, order_date and order_amount. Additionally, assign a unique row number as "row_num"  to each of these orders based on the descending order of order_amount values. 

  Hint: Use LIMIT to restrict results to 3 rows

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
SELECT
  ROW_NUMBER() OVER (ORDER BY order_amount DESC) AS row_num,
  order_id,
  customer_id,
  order_date,
  order_amount
FROM orders
ORDER BY order_amount DESC
LIMIT 3; -- Correct!
```

## Question 17: RANK

### Prompt

```text
Retrieve the first_name, last_name, department and salary for each department, including their rank as "rank" within the department based on salary.

  Table: employees
  ColumnNames		DataType		
  employee_id		int		
  first_name		varchar
  last_name		varchar  
  department		varchar 
  salary			decimal
  manager_id		int
```

### Solution and working notes

```sql
SELECT
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank,
  first_name,
  last_name,
  salary,
  department
FROM employees; -- Correct!
```

## Question 18: LEAD

### Prompt

```text
Retrieve a list of products along with their product_name, category and unit_price, including the name as "next_product_name" and price as "next_product_price" of the next higher-priced product within the same category. Additionally add 'X' to name in case where it is the last product in the list as per higher price logic and '0' for the price.

  Table: products 

  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal
```

### Solution and working notes

```sql
SELECT
  product_name,
  unit_price,
  LEAD(product_name, 1, 'X') OVER (PARTITION BY category ORDER BY unit_price) AS next_product_name,
  LEAD(unit_price, 1, 0) OVER (PARTITION BY category ORDER BY unit_price) AS next_product_price,
  category
FROM products; -- Correct!
-- You could also only specify the first argument in LEAD, then use COALESCE to handle null values.
```

## Question 19: LAG

### Prompt

```text
Retrieves a list of orders along with their order_id, order_date, order_amount and customer_id, and includes the order_date as "previous_order_date" and order_amount as "previous_total_amount" of the previous order made by the same customer. 

  Table: orders

  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar
```

### Solution and working notes

```sql
SELECT
  order_id,
  order_date,
  order_amount,
  LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS previous_order_date,
  LAG(order_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS previous_total_amount,
  customer_id
FROM orders; -- Correct!
```
