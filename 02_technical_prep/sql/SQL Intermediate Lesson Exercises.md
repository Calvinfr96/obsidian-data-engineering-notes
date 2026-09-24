---
aliases:
  - "sql_intermediate_lessons.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Practice Code/sql_intermediate_lessons.sql"
imported: 2026-09-24
---

# SQL Intermediate Lesson Exercises

Review: [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#Question 1: GROUP BY]]
- [[#Question 2: COUNT OPERATOR]]
- [[#Question 3: SUBQUERIES]]
- [[#Question 4: MAX OPERATOR]]
- [[#Question 5: MIN OPERATOR]]
- [[#Question 6: SUM OPERATOR]]
- [[#Question 7: AVG OPERATOR]]
- [[#Question 8: HAVING CLAUSE]]
- [[#Question 9: SUBQUERY WITH AGGREGATED FUNCTIONS]]
- [[#Question 10: CASE STATEMENT]]
- [[#Question 11: LEFT JOIN]]
- [[#Question 12: RIGHT JOIN]]
- [[#Question 13: INNER JOIN]]
- [[#Question 14: FULL OUTER JOIN OR OUTER JOIN]]
- [[#Question 15: JOIN WITH WHERE]]
- [[#Question 16: JOIN WITH COMPARISON OPERATOR]]
- [[#Question 17: DISTINCT]]
- [[#Question 18: JOIN WITH MULTIPLE KEYS]]
- [[#Question 19: SELF JOIN]]
- [[#Question 20: UNION]]
- [[#Question 21: CURDATE & CURTIME]]
- [[#Question 22: DATE ARITHMETIC FUNCTIONS]]
- [[#Question 23: DATE FORMAT]]

## Question 1: GROUP BY

### Prompt

```text
Retrieve the department and the total salary expenses (sum of salaries) as "TotalSalaryExpenses" for each department. Display the results in descending order based on the "TotalSalaryExpenses".

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
  | 6           | Maria      | Garcia    | Finance    | 78000.00| 4         |
  | 7           | Alex       | Lee       | HR         | 85000.00| 5         |
```

### Solution and working notes

```sql
SELECT
  department,
  SUM(salary) AS TotalSalaryExpenses
FROM employees
GROUP BY department
ORDER BY TotalSalaryExpenses DESC; -- Correct!
-- In this question, "for each" is the key phrase indicating you will need to use GROUP BY in the query.
-- Without the GROUP BY Clause, the SUM aggregate function would apply to the entire salary column, returning just one total. This is why it's important to use GROUP BY with aggregate functions.
```

## Question 2: COUNT OPERATOR

### Prompt

```text
Retrieve the department and the count of employees as "EmployeeCount" in each department. Display the results in descending order based on the "EmployeeCount".

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
  | 6           | Maria      | Garcia    | Finance    | 78000.00| 4         |
  | 7           | Alex       | Lee       | HR         | 85000.00| 5         |
```

### Solution and working notes

```sql
SELECT
  department,
  COUNT(*) AS EmployeeCount
FROM employees
GROUP BY department
ORDER BY EmployeeCount DESC; -- Correct!
-- Here, "in each departments" signifies the need for GROUP BY and "count of employees" signifies the need for COUNT.
-- To exclude null values from the count, you could also use a column with unique values, such as the primary key column (employee_id). The above query counts all values, both null and non-null.
```

## Question 3: SUBQUERIES

### Prompt

```text
Retrieve employee_id, first_name, last_name and salary who have a higher salary than the average salary across all employees.

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
  | employee_id | first_name | last_name | department | salary  | manager_id |
  |-------------|------------|-----------|------------|---------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3          |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4          |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5          |
  | 4           | Emily      | Davis     | IT         | 70000.00| 5          |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL       |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name,
  salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees); -- Correct!
```

## Question 4: MAX OPERATOR

### Prompt

```text
Retrieve the employee details (employee_id, first_name, last_name, department, salary) of the employee with the highest salary in each department.

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
  | employee_id | first_name | last_name | department | salary  | manager_id |
  |-------------|------------|-----------|------------|---------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3          |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4          |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5          |
  | 4           | Emily      | Davis     | IT         | 70000.00| 5          |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL       |
  | 6           | Maria      | Garcia    | Finance    | 78000.00| 4          |
  | 7           | Alex       | Lee       | HR         | 85000.00| 5          |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name,
  department,
  MAX(salary)
FROM employees
GROUP BY department; -- Incorrect
-- Tried to use GROUP BY in the subquery to obtain the max salary per department.
-- The main query needs to retrieve the requested rows from the employee table for employees with the max department salary.
-- As the main query is assessing rows, it needs to compare the salary of the current employee to that max salary.
-- To retrieve the max department salary, the subquery needs to use the where clause to only include rows where the department
-- from the outer query matches the department from the inner query (they have the same department), then apply MAX(salary) to that subset of data.

SELECT
  employee_id,
  first_name,
  last_name,
  department,
  salary
FROM employees AS table1
WHERE table1.salary = (
  SELECT
    MAX(salary)
  FROM employees as table2
  WHERE table1.department = table2.department -- Solution
);
-- We don't use GROUP BY here because that clause is typically used with aggregate functions such as SUM, AVG, or COUNT. In this case, we need the highest salary and MAX is not an aggregate function.
-- MAX retrieves the maximum value in a column, it doesn't perform a calculation based on the total value of rows in the column, such as sum, average or total count.
-- You don't use MAX in the main select clause because you're trying to find the details (including salary) of the employee with the max salary in each department.
-- This means you first need to find the max salary per department with a subquery, then use WHERE to compare that to the salary of the employees in the main query.
-- The most you could do by combining MAX and GROUP BY is display the department and highest salary per department, you wouldn't be able to list other employee details.
-- With the outer query, we're going row by row and asking, does this employee's salary match the maximum salary for the department (derived from the inner query)? If so, include in the result set.
```

## Question 5: MIN OPERATOR

### Prompt

```text
Retrieve the employee details (employee_id, first_name, last_name, department, salary) of the employee with the lowest salary.

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
  | employee_id | first_name | last_name | department | salary  | manager_id |
  |-------------|------------|-----------|------------|---------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3          |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4          |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5          |
  | 4           | Emily      | Davis     | IT         | 70000.00| 5          |
  | 5           | Robert     | Brown     | HR         | 60000.00| NULL       |
```

### Solution and working notes

```sql
SELECT
  employee_id,
  first_name,
  last_name,
  department,
  salary
FROM employees e1
WHERE salary = (
  SELECT
    MIN(salary)
  FROM employees
); -- Correct!
-- We need to collect the details of the employee with the lowest salary.
-- We can't use GROUP BY because we're not being asked to find the lowest salary within a certain grouping of employees.
-- We could only really use GROUP BY to do that if the question was only asking for the department and associated lowest salary.
-- Because the question asks for non-aggregated columns, we would need to use WHERE with a subquery query.
-- Here, the question just asks for the details of the overall lowest paid employee, so the subquery just needs to find the overall minimum salary.
-- You could also use IN instead of =, but since the subquery only returns one row, it's probably more appropriate to use =.
```

## Question 6: SUM OPERATOR

### Prompt

```text
Retrieve the department and the total salary (sum of salaries) as "TotalSalary" for each department. Display the results in descending order based on the total salary expenses.

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
  | employee_id | first_name | last_name | department | salary  | manager_id |
  |-------------|------------|-----------|------------|---------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3          |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4          |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5          |
  | 4           | Emily      | Davis     | IT         | 70000.00| 5          |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL       |
  | 6           | Maria      | Garcia    | Finance    | 78000.00| 4          |
  | 7           | Alex       | Lee       | HR         | 85000.00| 5          |
```

### Solution and working notes

```sql
SELECT
  department,
  SUM(salary) AS TotalSalary
FROM employees
GROUP BY department
ORDER BY TotalSalary DESC; -- Correct!
```

## Question 7: AVG OPERATOR

### Prompt

```text
Retrieve the year of the order_date as "order_year" along with the average order_amount (average of total amounts) as "AverageOrderAmount" for each year. Display the results in chronological order of the "order_year".

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
  | 1        | 2022-05-15 | 100.00       | 101         | Completed    | Address1          |
  | 2        | 2023-01-20 | 150.00       | 102         | Completed    | Address2          |
  | 3        | 2022-05-15 | 120.00       | 103         | Pending      | Address3          |
  | 4        | 2023-02-10 | 200.00       | 104         | Completed    | Address4          |
  | 5        | 2022-10-05 | 180.00       | 105         | Pending      | Address5          |
  | 6        | 2023-01-20 | 220.00       | 106         | Completed    | Address6          |
  | 7        | 2022-10-05 | 250.00       | 107         | Completed    | Address7          |
```

### Solution and working notes

```sql
SELECT
  YEAR(order_date) AS order_year,
  AVG(order_amount) AS AverageOrderAmount
FROM orders
GROUP BY order_year
ORDER BY order_year ASC; -- Correct!
-- "for each" was the key phrase indicating the need for GROUP BY
```

## Question 8: HAVING CLAUSE

### Prompt

```text
Retrieve the department and the average salary (average of salaries) as "AverageSalary" for departments with an average salary greater than 60000. Display the results in descending order based on the "AverageSalary".

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
  | employee_id | first_name | last_name | department | salary  | manager_id |
  |-------------|------------|-----------|------------|---------|------------|
  | 1           | John       | Doe       | Sales      | 65000.00| 3          |
  | 2           | Jane       | Smith     | Finance    | 75000.00| 4          |
  | 3           | Mike       | Johnson   | Sales      | 80000.00| 5          |
  | 4           | Emily      | Davis     | IT         | 70000.00| 5          |
  | 5           | Robert     | Brown     | HR         | 90000.00| NULL       |
  | 6           | Maria      | Garcia    | Finance    | 78000.00| 4          |
  | 7           | Alex       | Lee       | HR         | 85000.00| 5          |
```

### Solution and working notes

```sql
SELECT
  department,
  AVG(salary) AS AverageSalary
FROM employees
GROUP BY department
HAVING AverageSalary > 60000
ORDER BY AverageSalary DESC; -- Correct!
```

## Question 9: SUBQUERY WITH AGGREGATED FUNCTIONS

### Prompt

```text
Retrieve the customer_name of customers who have placed orders with a total amount greater than the average amount of all orders.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address             varchar

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar

  Sample Input:
  customers
  | customer_id | customer_name | age | city| email        | address            |
  |-------------|---------------|-----|-----|--------------|--------------------|
  | 101         | Alice         | 30  | NYC | alice@email  | 123 Main Street    |
  | 102         | Bob           | 35  | LA  | bob@email    | 456 Oak Avenue     |
  | 103         | Charlie       | 28  | SF  | charlie@email| 789 Pine Boulevard |
  | 104         | David         | 40  | NYC | david@email  | 101 Elm Lane       |
  | 105         | Emily         | 25  | LA  | emily@email  | 202 Maple Street   |

  orders
  | order_id | order_date | order_amount | customer_id | order_status | shipping_address  |
  |----------|------------|--------------|-------------|--------------|-------------------|
  | 1        | 2023-03-15 | 100.00       | 101         | Pending      | 123 Main Street   |
  | 2        | 2023-05-20 | 150.00       | 102         | Completed    | 456 Oak Avenue    |
  | 3        | 2023-02-10 | 200.00       | 103         | Shipped      | 789 Pine Boulevard|
  | 4        | 2023-06-25 | 80.00        | 104         | Pending      | 101 Elm Lane      |
  | 5        | 2023-04-05 | 120.00       | 105         | Completed    | 202 Maple Street  |
```

### Solution and working notes

```sql
-- Unable to come up with solution =(

SELECT
  customer_name
FROM customers
WHERE customer_id IN (
  SELECT
    customer_id
  FROM orders
  GROUP BY customer_id
  HAVING
    SUM(order_amount) > (
      SELECT
        AVG(order_amount)
      FROM orders
    )
); -- Solution
-- In the main query, we're getting customer_name from the customers table.
-- In the first subquery, we're retrieving a list of customer IDs from the orders table (to be evaluated by the IN operator).
-- This list is produced by grouping the rows in the orders table by customer ID, then eliminating groups (customer IDs) where the total order amount is less than average order amount of all orders.
-- The second subquery is what determines the average order amount amongst all orders.
-- Without a HAVING clause in the first subquery, it would simply return a list of unique customer IDs.
```

## Question 10: CASE STATEMENT

### Prompt

```text
Retrieve the product_name, unit_price and a column named "PriceCategory" that categorizes products into three groups: "Low", "Medium", and "High" based on their price ranges. Use the following criteria:
  1. "Low" for products with price less than 200.
  2. "Medium" for products with price between 200 and 500.
  3. "High" for products with price greater than 500.

  Table: products 
  ColumnName          DataType
  product_id          int
  product_name        varchar 
  category            varchar
  unit_price          decimal


  Sample Input:
  products
  | product_id | product_name | category    | unit_price |
  |------------|--------------|-------------|------------|
  | 1          | Laptop       | Electronics | 800.00     |
  | 2          | Smartphone   | Electronics | 300.00     |
  | 3          | Headphones   | Electronics | 120.00     |
  | 4          | Microwave    | Appliances  | 250.00     |
  | 5          | Refrigerator | Appliances  | 600.00     |
  | 6          | Blender      | Appliances  | 50.00      |
  | 7          | Desk Chair   | Furniture   | 180.00     |
```

### Solution and working notes

```sql
SELECT
  product_name,
  unit_price,
  (
    CASE
      WHEN unit_price < 200 THEN "Low"
      WHEN unit_price BETWEEN 200 AND 500 THEN "Medium"
      ELSE "High"
    END
  ) AS PriceCategory
FROM products; -- Correct!
-- The CASE Statement here categorizes the product in each row based on unit price.
```

## Question 11: LEFT JOIN

### Prompt

```text
Retrieve a list of customer_id and customer_name along with the total number of orders as "order_count" they have placed and the total amount as "total_amount" they have spent.

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


  Sample Input:
  customers
  | customer_id | customer_name  | age | city | email               | address               |
  |-------------|----------------|-----|------|---------------------|-----------------------|
  | 1           | John Doe       | 30  | NY   | john.doe@email.com. | 123 Main Street       |
  | 2           | Jane Smith     | 25  | LA   | jane.smith@email.com| 456 Oak Avenue        |
  | 3           | Mike Johnson   | 35  | SF   | mike.j@email.com    | 789 Pine Boulevard    |
  | 4           | Emily Davis    | 28  | CHI  | emily.d@email.com   | 101 Elm Street        |

  orders
  | order_id | order_date | order_amount  | customer_id | order_status | shipping_address      |
  |----------|------------|---------------|-------------|--------------|-----------------------|
  | 101      | 2023-01-15 | 120.00        | 1           | Completed    | 123 Main Street       |
  | 102      | 2023-02-20 | 250.00        | 1           | Shipped      | 123 Main Street       |
  | 103      | 2023-03-10 | 180.00        | 2           | Completed    | 456 Oak Avenue        |
  | 104      | 2023-04-05 | 300.00        | 3           | Completed    | 789 Pine Boulevard    |
  | 105      | 2023-05-12 | 150.00        | 2           | Shipped      | 456 Oak Avenue        |
  | 106      | 2023-06-08 | 200.00        | 4           | Completed    | 101 Elm Street        |
```

### Solution and working notes

```sql
-- Unable to come up with solution =(

SELECT
 c.customer_id,
 c.customer_name,
 COUNT(o.order_id) AS order_count,
 SUM(order_amount) AS total_amount
FROM
  customers AS c LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name; -- Solution
-- "order_count" indicates we need to count the number of valid rows per customer in the orders table. "total_amount" indicates we need to sum order amount per customer.
-- Since this question is related to LEFT JOIN, it should have explicitly stated to include customers that haven't placed any orders.
-- Instead of using 'ON c.customer_id = o.customer_id', you could also use 'USING(customer_id)'.
-- We group by customer ID and customer name because it is best practice to GROUP BY all non-aggregated columns added to the SELECT Statement.
-- The solution joins customers, then orders, because you're much less likely to find orders without customers than you are customers without orders.
```

## Question 12: RIGHT JOIN

### Prompt

```text
Exercise: Retrieve a list of orders with their order_id, order_date, order_amount  and the corresponding customer information (customer_name and email). Also include customers who have not made any orders.

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


  Sample Input:
  customers
  | customer_id | customer_name  | age | city | email               | address               |
  |-------------|----------------|-----|------|---------------------|-----------------------|
  | 1           | John Doe       | 30  | NY   | john.doe@email.com  | 123 Main Street       |
  | 2           | Jane Smith     | 25  | LA   | jane.smith@email.com| 456 Oak Avenue        |
  | 3           | Mike Johnson   | 35  | SF   | mike.j@email.com    | 789 Pine Boulevard    |

  orders
  | order_id | order_date | order_amount  | customer_id | order_status | shipping_address      |
  |----------|------------|---------------|-------------|--------------|-----------------------|
  | 101      | 2023-01-15 | 120.00        | 1           | Completed    | 123 Main Street       |
  | 102      | 2023-02-20 | 250.00        | 1           | Shipped      | 123 Main Street       |
  | 103      | 2023-03-10 | 180.00        | 2           | Completed    | 456 Oak Avenue        |
  | 104      | 2023-04-05 | 300.00        | 3           | Completed    | 789 Pine Boulevard    |
  | 105      | 2023-05-12 | 150.00        | 2           | Shipped      | 456 Oak Avenue        |
  | 106      | 2023-06-08 | 200.00        | 1           | Completed    | 123 Main Street       |
```

### Solution and working notes

```sql
SELECT
  o.order_id,
  o.order_date,
  o.order_amount,
  c.customer_name,
  c.email
FROM orders AS o RIGHT JOIN customers AS c
ON o.customer_id = c.customer_id; -- Correct!
-- Specifying to include customers who have not made any orders was a signal to use RIGHT JOIN with customers as the second table.
```

## Question 13: INNER JOIN

### Prompt

```text
Exercise: Retrieve a list of customer_id, customer_name who have placed orders along with information about their orders, including the order_id, order_date and order_amount.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address             varchar

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar


  Sample Input:
  customers
  | customer_id | customer_name  | age | city | email               | address               |
  |-------------|----------------|-----|------|---------------------|-----------------------|
  | 1           | John Doe       | 30  | NY   | john.doe@email.com. | 123 Main Street       |
  | 2           | Jane Smith     | 25  | LA   | jane.smith@email.com| 456 Oak Avenue        |
  | 3           | Mike Johnson   | 35  | SF   | mike.j@email.com    | 789 Pine Boulevard    |
  | 4           | Emily Davis    | 28  | CHI  | emily.d@email.com   | 101 Elm Street        |

  orders
  | order_id | order_date | order_amount  | customer_id | order_status | shipping_address      |
  |----------|------------|---------------|-------------|--------------|-----------------------|
  | 101      | 2023-01-15 | 120.00        | 1           | Completed    | 123 Main Street       |
  | 102      | 2023-02-20 | 250.00        | 1           | Shipped      | 123 Main Street       |
  | 103      | 2023-03-10 | 180.00        | 2           | Completed    | 456 Oak Avenue        |
  | 104      | 2023-04-05 | 300.00        | 3           | Completed    | 789 Pine Boulevard    |
  | 105      | 2023-05-12 | 150.00        | 2           | Shipped      | 456 Oak Avenue        |
  | 106      | 2023-06-08 | 200.00        | 4           | Completed    | 101 Elm Street        |
```

### Solution and working notes

```sql
SELECT
  c.customer_id,
  c.customer_name,
  o.order_id,
  o.order_date,
  o.order_amount
FROM customers AS c INNER JOIN orders AS o
ON c.customer_id = o.customer_id; -- Correct!
```

## Question 14: FULL OUTER JOIN OR OUTER JOIN

### Prompt

```text
Retrieve a list of customer_id, customer_name and their corresponding order_id, order_date and order_amount for customers who have placed an order. For other customers order details can be null as they haven't placed an order.

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


  Sample Input:
  customers
  | customer_id | customer_name  | age | city | email               | address               |
  |-------------|----------------|-----|------|---------------------|-----------------------|
  | 1           | John Doe       | 30  | NY   | john.doe@email.com. | 123 Main Street       |
  | 2           | Jane Smith     | 25  | LA   | jane.smith@email.com| 456 Oak Avenue        |
  | 3           | Mike Johnson   | 35  | SF   | mike.j@email.com    | 789 Pine Boulevard    |
  | 4           | Emily Davis    | 28  | CHI  | emily.d@email.com   | 101 Elm Street        |

  orders
  | order_id | order_date | order_amount  | customer_id | order_status | shipping_address      |
  |----------|------------|---------------|-------------|--------------|-----------------------|
  | 101      | 2023-01-15 | 120.00        | 1           | Completed    | 123 Main Street       |
  | 102      | 2023-02-20 | 250.00        | 1           | Shipped      | 123 Main Street       |
  | 103      | 2023-03-10 | 180.00        | 2           | Completed    | 456 Oak Avenue        |
  | 104      | 2023-04-05 | 300.00        | 3           | Completed    | 789 Pine Boulevard    |
  | 105      | 2023-05-12 | 150.00        | 2           | Shipped      | 456 Oak Avenue        |
  | 106      | 2023-06-08 | 200.00        | 1           | Completed    | 123 Main Street       |
```

### Solution and working notes

```sql
SELECT
  c.customer_id,
  c.customer_name,
  o.order_id,
  o.order_date,
  o.order_amount
FROM customers AS c LEFT JOIN orders AS o
ON c.customer_id = o.customer_id; -- Correct!
-- We need to use LEFT JOIN here instead of FULL OUTER JOIN because the version of MySQL used by the DEA website does not support this function.
```

## Question 15: JOIN WITH WHERE

### Prompt

```text
Retrieve a list of customer_id and customer_name and their corresponding order_id, order_date and order_amount, but only for orders placed between '2023-01-01' and '2023-06-30'.

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


  Sample Input:
  customers
  | customer_id | customer_name  | age | city | email               | address               |
  |-------------|----------------|-----|------|---------------------|-----------------------|
  | 1           | John Doe       | 30  | NY   | john.doe@email.com. | 123 Main Street       |
  | 2           | Jane Smith     | 25  | LA   | jane.smith@email.com| 456 Oak Avenue        |
  | 3           | Mike Johnson   | 35  | SF   | mike.j@email.com    | 789 Pine Boulevard    |

  orders
  | order_id | order_date | order_amount  | customer_id | order_status | shipping_address      |
  |----------|------------|---------------|-------------|--------------|-----------------------|
  | 101      | 2023-01-15 | 120.00        | 1           | Completed    | 123 Main Street       |
  | 102      | 2023-02-20 | 250.00        | 1           | Shipped      | 123 Main Street       |
  | 103      | 2023-03-10 | 180.00        | 2           | Completed    | 456 Oak Avenue        |
  | 104      | 2023-04-05 | 300.00        | 3           | Completed    | 789 Pine Boulevard    |
  | 105      | 2023-05-12 | 150.00        | 2           | Shipped      | 456 Oak Avenue        |
  | 106      | 2023-06-08 | 200.00        | 1           | Completed    | 123 Main Street       |
  | 107      | 2023-07-15 | 220.00        | 2           | Shipped      | 456 Oak Avenue        |
```

### Solution and working notes

```sql
SELECT
  c.customer_id,
  c.customer_name,
  o.order_id,
  o.order_date,
  o.order_amount
FROM customers AS c INNER JOIN orders AS o
ON c.customer_id = o.customer_id
WHERE o.order_date BETWEEN '2023-01-01' AND '2023-06-30'; -- Correct!
-- Although this is the correct answer, remember to use caution when using BETWEEN with date literals.
```

## Question 16: JOIN WITH COMPARISON OPERATOR

### Prompt

```text
Retrieve a list of customer_id and customer_name and their corresponding order_id, order_date and order_amount, but only for orders with a total amount at least 500.

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


  Sample Input:
  customers
  | customer_id | customer_name  | age | city | email               | address               |
  |-------------|----------------|-----|------|---------------------|-----------------------|
  | 1           | John Doe       | 30  | NY   | john.doe@email.com  | 123 Main Street       |
  | 2           | Jane Smith     | 25  | LA   | jane.smith@email.com| 456 Oak Avenue        |
  | 3           | Mike Johnson   | 35  | SF   | mike.j@email.com    | 789 Pine Boulevard    |

  orders
  | order_id | order_date | order_amount  | customer_id | order_status | shipping_address      |
  |----------|------------|---------------|-------------|--------------|-----------------------|
  | 101      | 2023-01-15 | 120.00        | 1           | Completed    | 123 Main Street       |
  | 102      | 2023-02-20 | 150.00        | 1           | Shipped      | 123 Main Street       |
  | 103      | 2023-03-10 | 180.00        | 2           | Completed    | 456 Oak Avenue        |
  | 104      | 2023-04-05 | 600.00        | 3           | Completed    | 789 Pine Boulevard    |
  | 105      | 2023-05-12 | 520.00        | 2           | Shipped      | 456 Oak Avenue        |
  | 106      | 2023-06-08 | 200.00        | 1           | Completed    | 123 Main Street       |
  | 107      | 2023-07-15 | 800.00        | 2           | Shipped      | 456 Oak Avenue        |
```

### Solution and working notes

```sql
SELECT
  c.customer_id,
  c.customer_name,
  o.order_id,
  o.order_date,
  o.order_amount
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id AND o.order_amount >= 500; -- Correct!
```

## Question 17: DISTINCT

### Prompt

```text
Retrieve all unique departments. Display department.

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
  | 1           | John       | Doe       | Sales      | 60000  | 3          |
  | 2           | Jane       | Smith     | Marketing  | 55000  | 4          |
  | 3           | Mike       | Johnson   | IT         | 70000  | 5          |
  | 4           | Emily      | Davis     | Finance    | 80000  | 6          |
  | 5           | Chris      | Evans     | Sales      | 65000  | 3          |
  | 6           | Emma       | Wilson    | Marketing  | 60000  | 4          |
  | 7           | David      | Lee       | IT         | 75000  | 5          |
```

### Solution and working notes

```sql
SELECT
  DISTINCT department AS department
FROM employees; -- Correct!
-- The alias isn't necessarily required, but it doesn't hurt to add it.
```

## Question 18: JOIN WITH MULTIPLE KEYS

### Prompt

```text
Retrieve customer_id, customer_name and address along with their order_id, order_date, order_amount, order_status and shipping_address of customers whose address and shipping address match.

  Table: customers
  ColumnName          Datatype           
  customer_id         int 
  customer_name       varchar
  age                 int 
  city                varchar
  email               varchar
  address             varchar

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
  c.address,
  o.order_id,
  o.order_date,
  o.order_amount,
  o.order_status,
  o.shipping_address
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id
AND c.address = o.shipping_address; -- Correct!
```

## Question 19: SELF JOIN

### Prompt

```text
Retrieve pairs of employees who share the same manager. Display first employee_id AS employee1_id, first employee's first_name & last_name combined as employee1_name, second employee_id AS employee2_id, second employee's first_name & last_name combined as as employee2_name and manager_id. Use CONCAT(e1.first_name, ' ', e1.last_name) to get the full name.

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
  | 1           | John       | Doe       | Sales      | 60000  | 3          |
  | 2           | Jane       | Smith     | Marketing  | 55000  | 4          |
  | 3           | Mike       | Johnson   | IT         | 70000  | 5          |
  | 4           | Emily      | Davis     | Finance    | 80000  | 6          |
  | 5           | Chris      | Evans     | Sales      | 65000  | 3          |
  | 6           | Emma       | Wilson    | Marketing  | 60000  | 4          |
  | 7           | David      | Lee       | IT         | 75000  | 5          |
```

### Solution and working notes

```sql
SELECT
  e1.employee_id AS employee_id,
  CONCAT(e1.first_name, ' ', e1.last_name) AS employee1_name,
  e2.employee_id AS employee2_id,
  CONCAT(e2.first_name, ' ', e2.last_name) AS employee2_name,
  e1.manager_id AS manager_id
FROM employees AS e1 JOIN employees AS e2
ON e1.manager_id = e2.manager_id
AND e1.employee_id <> e2.employee_id; -- Incorrect

SELECT
  e1.employee_id AS employee_id,
  CONCAT(e1.first_name, ' ', e1.last_name) AS employee1_name,
  e2.employee_id AS employee2_id,
  CONCAT(e2.first_name, ' ', e2.last_name) AS employee2_name,
  e1.manager_id AS manager_id
FROM employees AS e1 JOIN employees AS e2
ON e1.manager_id = e2.manager_id
AND e1.employee_id < e2.employee_id
WHERE e1.manager_id IS NOT NULL; -- Solution
-- We need a self join here because a manager is an employee (the hierarchy). The manager_id column refers to a value from the employee_id column.
-- 'AND e1.employee_id < e2.employee_id' is required to avoid redundant pairs (i.e paring employee 101 and 103, then pairing employee 103 and 101)
  -- We use < (you can also use >) instead of != because != doesn't actually avoid redundant pairs. Two employees would never share the same ID because it's a primary key.
  -- Using < limits the scope of the comparison that the join performs. When comparing a row in table A to the rows in table B, it won't look at all of the rows.
  -- This results in pairs where e1.employee_id is ALWAYS less than e2.employee_id
-- 'WHERE e1.manager_id IS NOT NULL' is required to exclude employees who don't have managers.
```

## Question 20: UNION

### Prompt

```text
Create a consolidated list of transactions from both the Orders and Invoices tables. Your result must contain customer_id, order_date as 'transaction_date' named column and the order_amount & total_amount from both tables as 'transaction_amount' named column.

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar

  Table: invoices
  ColumnName      DataType
  invoice_id      int
  customer_id     int
  invoice_date    date
  total_amount    decimal
```

### Solution and working notes

```sql
SELECT
  customer_id,
  order_date AS transaction_date,
  order_amount AS transaction_amount,
FROM orders
UNION
SELECT
  customer_id,
  invoice_date AS transaction_date,
  total_amount AS transaction_amount,
FROM invoices; -- Correct!
```

## Question 21: CURDATE & CURTIME

### Prompt

```text
Retrieve all orders and display the following (after displaying current dates and times delete them from select to get the question passed marker):
  order_id
  order_status
  order_amount
  customer_id
  customer_name
  The current date as ReportDate
  The current time as ReportTime
  The current date and time as ReportGeneratedAt

  Table: orders
  ColumnName      DataType
  order_id        int
  order_date      date
  order_amount    decimal
  customer_id     int
  order_status    varchar
  shipping_address   varchar

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
  o.order_id,
  o.order_status,
  o.order_amount,
  c.customer_id,
  c.customer_name,
  CURDATE() AS ReportDate,
  CURTIME() AS ReportTime, -- UTC
  NOW() AS ReportGeneratedAt -- UTC
FROM orders AS o JOIN customers AS c
ON o.customer_id = c.customer_id; -- Correct!
```

## Question 22: DATE ARITHMETIC FUNCTIONS

### Prompt

```text
Retrieve all orders displaying order_id, order_status, order_amount, and the original order_date. Additionally, calculate and display:
  The expected delivery date by adding 7 days to the order_date using DATE_ADD.
  The last cancellation date by subtracting 3 days from the order_date using DATE_SUB.
  Also include the customer_id and customer_name for each order.

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
  o.order_status,
  o.order_amount,
  o.order_date,
  DATE_ADD(DATE(order_date), INTERVAL 7 DAY) AS expected_delivery_date,
  DATE_SUB(DATE(order_date), INTERVAL 3 DAY) AS last_cancellation_date,
  c.customer_id,
  c.customer_name
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id; -- Correct!
```

## Question 23: DATE FORMAT

### Prompt

```text
Write a query to display each customer’s name, their order date in the format DD/MM/YYYY, and the weekday name of the order.

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
  c.customer_name,
  DATE_FORMAT(o.order_date, '%d/%m/%Y') AS formatted_date,
  DAYNAME(o.order_date) AS weekday
FROM customers AS c JOIN orders AS o
ON c.customer_id = o.customer_id; -- Correct!
-- You could also use 'DATE_FORMAT(o.order_date, '%W') AS weekday' to get the weekday instead of using DAYNAME.
```
