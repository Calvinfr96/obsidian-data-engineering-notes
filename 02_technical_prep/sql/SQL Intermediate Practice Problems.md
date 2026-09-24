---
aliases:
  - "intermediate_problems.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Practice Problems/intermediate_problems.sql"
imported: 2026-09-24
---

# SQL Intermediate Practice Problems

Review: [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#Problem 1: Employee Bonus Calculation]]
- [[#Problem 2: Customer Demographics Analysis]]
- [[#Problem 3: Average Years of Experience]]
- [[#Problem 4: Multiple Aggregations]]
- [[#Problem 5: Age Range Counts]]
- [[#Problem 6: Highest Salary above Average]]
- [[#Problem 7: Total Sales by Product Category]]
- [[#Problem 8: Total Revenue by Customer]]
- [[#Problem 9: Customers with High Spending]]
- [[#Problem 10: Average Order Amount by Customer]]

## Problem 1: Employee Bonus Calculation

### Prompt

```text
Calculate and display 5% bonus amount for employees in the employee_dimension table who have generated more than $75000 in revenue. Round the employee bonus to 2 decimals.

  Tables: employee_dimension
  ColumnName		    Datatype
  employee_id 		INT
  role 			    VARCHAR
  department 		    VARCHAR
  experience_years 	INT
  salary 			    INT
  previousyearsalary 	INT
  location 		    VARCHAR

  Table: employee_performance_fact 
  ColumnName		    	Datatype
  employee_id 			INT
  month 				    VARCHAR
  year 				    INT
  projects_completed 		INT
  hours_worked 			INT
  revenue_generated 		INT
```

### Solution and working notes

```sql
SELECT
  Employee_ID,
  Salary,
  PreviousYearSalary,
  ROUND(employee.salary * 0.05, 2) AS employee_bonus
FROM employee_dimension AS employee JOIN employee_performance_fact AS performance USING(Employee_ID)
WHERE revenue_generated > 75000; -- Correct!
-- Note: Question is not specific about which columns from the table should be included, but the employee_bonus column was correct when it was the only column I included.
-- Instead of joining, we also could have done:
select employee_id, salary, previousyearsalary, (salary * 0.05) as bonus
from employee_dimension
WHERE employee_id IN (
    SELECT employee_id
    FROM employee_performance_fact
    WHERE revenue_generated > 75000)
-- This might be better since joining wasn't necessary (we didn't need to include columns from both tables in the result set).
-- Consider using subqueries when joining isn't absolutely necessary.
-- We also could have added 'revenue_generated > 75000' condition to the ON Clause instead of putting it in a separate WHERE.
```

## Problem 2: Customer Demographics Analysis

### Prompt

```text
Find the average age of customers in each city from the customers_prc table to understand the age demographics of different locations.

  Tables: customers_prc
  ColumnName		Datatype
  customerid 		INT
  name 			VARCHAR
  age 			INT
  city 			VARCHAR
```

### Solution and working notes

```sql
SELECT
  city,
  AVG(age) AS average_age
FROM customers_prc
GROUP BY city; -- Correct!
```

## Problem 3: Average Years of Experience

### Prompt

```text
Calculate the average experience (in years) of data scientists who have expertise in "TensorFlow."

  Table: qualificationandskills
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR


  Table: employee_dimension
  ColumnName		    Datatype
  employee_id 		INT
  role 			    VARCHAR
  department 		    VARCHAR
  experience_years 	INT
  salary 			    INT
  previousyearsalary 	INT
  location 		    VARCHAR
```

### Solution and working notes

```sql
SELECT
  AVG(employees.Experience_Years)
FROM employee_dimension AS employees
WHERE employees.Employee_ID IN (
  SELECT
    qualifications.Employee_ID
  FROM qualificationandskills AS qualifications
  WHERE qualifications.Technical_Skills LIKE '%TensorFlow%'
) AND employees.Role = 'Data Scientist'; -- Correct!
-- This solution purposefully avoids joins

SELECT
  AVG(employees.Experience_Years)
FROM employee_dimension AS employees JOIN qualificationandskills AS qualifications
ON (
  employees.Employee_ID = qualifications.Employee_ID AND
  employees.Role = 'Data Scientist' AND
  qualifications.Technical_Skills LIKE '%TensorFlow%'
); -- Correct!
-- This solution uses joins with complex join criteria.
-- You could also just join on employee ID, the filter the remaining criteria using WHERE.
```

## Problem 4: Multiple Aggregations

### Prompt

```text
Calculate the total quantity sold, average price, and the maximum price for each product category in the sales_prc and products_prc tables.

  Tables: sales_prc
  ColumnName		    	Datatype
  saleid 				    INT
  product 			    VARCHAR
  quantitysold 			INT

  Table: products_prc
  ColumnName		    	Datatype
  productid 			    INT
  productname 			VARCHAR
  price 				    DECIMAL
  category 			    VARCHAR
  quantityinstock 		INT
```

### Solution and working notes

```sql
SELECT
  category,
  (
    SELECT
      SUM(quantitysold)
    FROM sales_prc
    WHERE product = productname
  ) AS total_quantity_sold,
  AVG(price) AS average_price,
  MAX(price) AS maximum_price,
FROM products_prc
GROUP BY category; -- Incorrect
-- This solution is wrong because the two tables need to joined on product/productname before being grouped by category

SELECT
  p.category,
  SUM(s.quantitysold) AS total_quantity_sold,
  AVG(p.price) AS average_price,
  MAX(p.price) AS maximum_price
FROM sales_prc AS s JOIN products_prc AS p
ON s.product = p.productname
GROUP BY p.category; -- Solution
-- Even though the columns in the products and sales tables had different names, the productname and product columns were the common columns among the two.
-- Always look for instances like this and clarify whether the columns have the same data and can be used to join two tables.
```

## Problem 5: Age Range Counts

### Prompt

```text
Write an SQL query to count the number of customers in three age ranges: under 30, 30 to 40, and over 40, in the customers_prc table.

  Tables: customers_prc
  ColumnName		Datatype
  customerid 		INT
  name 			VARCHAR
  age 			INT
  city 			VARCHAR
```

### Solution and working notes

```sql
SELECT
  SUM(
    CASE
      WHEN age < 30 THEN 1
      ELSE 0
    END
  ) AS customers_aged_under_30,
  SUM(
    CASE
      WHEN age BETWEEN 30 AND 40 THEN 1
      ELSE 0
    END
  ) AS customers_aged_30_to_40,
  SUM(
    CASE
      WHEN age > 40 THEN 1
      ELSE 0
    END
  ) AS customers_aged_over_40
FROM customers_prc; -- Correct!
```

## Problem 6: Highest Salary above Average

### Prompt

```text
Write an SQL query to identify the employee with the highest salary in the employees_src table, but only if their salary is above the average salary of all employees.

  Tables: employees_prc
  ColumnName		    	Datatype
  employeeid 			INT
  name 				VARCHAR
  salary 				DECIMAL
  department 			VARCHAR
```

### Solution and working notes

```sql
SELECT
  employeeid,
  name,
  salary
FROM employees_prc
WHERE salary = (
  SELECT
    MAX(salary)
  FROM employees_prc
); -- Correct!
-- This solution is correct because statistically, the maximum is always greater than the average.
-- Using AVG instead of MAX in the subquery returns two employees, to get the maximum from that result set, use ORDER BY and LIMIT:
SELECT
  employeeid,
  name,
  salary
FROM employees_prc
WHERE salary > (
  SELECT
    AVG(salary)
  FROM employees_prc
);
ORDER BY salary DESC
LIMIT 1;
```

## Problem 7: Total Sales by Product Category

### Prompt

```text
Write an SQL query to calculate the total sales amount for each product category in the sales_prc and products_prc tables.

  Table: sales_prc
  ColumnName		    	Datatype
  saleid 				    INT
  product 			    VARCHAR
  quantitysold 			INT

  Table: products_prc
  ColumnName		    	Datatype
  productid 			    INT
  productname 			VARCHAR
  price 				    DECIMAL
  category 			    VARCHAR
  quantityinstock 		INT
```

### Solution and working notes

```sql
SELECT
  p.category,
  SUM(p.price * s.quantitysold) AS total_sales_amount
FROM sales_prc AS s JOIN products_prc AS p
ON s.product = p.productname
GROUP BY category; -- Correct!
```

## Problem 8: Total Revenue by Customer

### Prompt

```text
Calculate the total revenue for each customer.

  Tables: orders_prc
  ColumnName		Datatype
  order_id 		INT
  customer_id     INT
  order_date 		DATE
  amount 			INT

  Table: customers_prc
  ColumnName		Datatype
  customerid 		INT
  name 			VARCHAR
  email 			VARCHAR
  age 			INT
  city 			VARCHAR
```

### Solution and working notes

```sql
SELECT
  SUM(amount) AS total_customer_revenue
FROM orders_prc
GROUP BY customer_id; -- Correct!
-- This solution is technically correct, but the posted solution also requires customer name in the result set, meaning we need to join.

SELECT
  c.name AS customer_name,
  SUM(o.amount) AS total_customer_revenue
FROM orders_prc AS o JOIN customers_prc AS c
ON o.customer_id = c.customerid
GROUP BY o.customer_id; -- Correct!
-- We also could have grouped by customer name.
```

## Problem 9: Customers with High Spending

### Prompt

```text
Find the customers who have a total spending (sum of "amount") greater than $250.

  Tables: customers_prc
  ColumnName		Datatype
  customerid 		INT
  name 			VARCHAR
  age 			INT
  city 			VARCHAR

  Table: orders_prc
  ColumnName		Datatype
  order_id 		INT
  customer_id     INT
  order_date 		DATE
  amount 			INT
```

### Solution and working notes

```sql
SELECT
  c.name AS customer_name,
  SUM(o.amount) AS total_spending
FROM customers_prc AS c JOIN orders_prc AS o
ON c.customerid = o.customer_id
GROUP BY o.customer_id
HAVING total_spending > 250; -- Correct!
-- This answer is correct. However, the solution only wants the result set to have customer name.
-- To get rid of the total_spending column, we could simply move SUM(o.amount) to the HAVING Clause instead of using the alias.

SELECT
  result.customer_name
FROM (
  SELECT
    c.name AS customer_name,
    SUM(o.amount) AS total_spending
  FROM customers_prc AS c JOIN orders_prc AS o
  ON c.customerid = o.customer_id
  GROUP BY o.customer_id
  HAVING total_spending > 250
) AS result; -- Correct!
-- This is a more convoluted approach that allows us to keep the alias.
```

## Problem 10: Average Order Amount by Customer

### Prompt

```text
Calculate the average order amount for each customer.

  Tables: customers_prc
  ColumnName		Datatype
  customerid 		INT
  name 			VARCHAR
  age 			INT
  city 			VARCHAR

  Table: orders_prc
  ColumnName		Datatype
  order_id 		INT
  customer_id     INT
  order_date 		DATE
  amount 			INT
```

### Solution and working notes

```sql
SELECT
  c.name,
  AVG(o.amount) AS average_order_amount
FROM customers_prc AS c JOIN orders_prc AS o
ON c.customerid = o.customer_id
GROUP BY o.customer_id; -- Correct!
```
