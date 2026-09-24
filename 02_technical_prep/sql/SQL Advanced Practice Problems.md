---
aliases:
  - "advanced_problems.sql"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Practice Problems/advanced_problems.sql"
imported: 2026-09-24
---

# SQL Advanced Practice Problems

Review: [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#Problem 1: Customer With Consistent Spending]]
- [[#Problem 2: VIP Customer]]
- [[#Problem 3: Gap Between Orders]]
- [[#Problem 4: Revenue Fluctuation]]
- [[#Problem 5: Frequency of High-Value Orders]]
- [[#Problem 6: Orders on Weekends]]
- [[#Problem 7: Orders on Holidays]]
- [[#Problem 8: Quarterly Revenue Trends]]
- [[#Problem 9: Customers With Orders in Consecutive Years and Repeated Orders]]
- [[#Problem 10: Most Valued City]]
- [[#Problem 11: Expertise in Python]]
- [[#Problem 12: Completed More Projects]]
- [[#Problem 13: Most Valued Employee]]
- [[#Problem 14: Most Experience in Data Science Department]]
- [[#Problem 15: Skills of a Data Analyst]]
- [[#Problem 16: Ph.D Employees Revenue]]
- [[#Problem 17: Months With No Orders]]
- [[#Problem 18: Average Revenue in January 2023]]
- [[#Problem 19: Employees Completed Most Projects]]
- [[#Problem 20: Average Order Amount by Month]]

## Problem 1: Customer With Consistent Spending

### Prompt

```text
Find customers who have made orders in at least two months of 2022 and list their names.

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
WITH order_stats AS (
  SELECT
      o.customer_id,
      c.name,
      MONTH(order_date) AS month
  FROM customers_prc AS c JOIN orders_prc AS o
  ON c.customerid = o.customer_id
  WHERE YEAR(order_date) = 2022
  GROUP BY o.customer_id, c.name, MONTH(order_date)
  HAVING COUNT(order_id) > 0
  ORDER BY month
)

SELECT
  name
FROM order_stats
GROUP BY customer_id
HAVING COUNT(month) >= 2; -- Correct!
-- In the subquery, we get the IDs for customers who have placed orders in at least two months of 2022 as follows:
-- We filter the result set with 'WHERE YEAR(order_date) = 2022'
-- We GROUP BY customer_id, then filter those groups according to 'COUNT(DISTINCT MONTH(order_date)) = 2'
-- This ensures that the result set only includes groups where the distinct month count is at least 2.
-- Reasoning:
-- We need name from the customers_prc table and order information (orders made in each month of 2022) from the orders_prc table.
-- Instead of joining, we use a subquery.
-- In the main query we just need the names of customers who satisfy the requirements.
-- To get the requirements we filter costumer_id as follows in the subquery:
-- First, filter the orders_prc table down to 2022 orders.
-- Next group by customers.
-- Finally, filter those groups based on those that have a distinct month count of at least 2.
```

## Problem 2: VIP Customer

### Prompt

```text
Define a VIP customer as someone who has made more than two orders. Find all VIP customers in the dataset.

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
WITH valid_customers AS (
  SELECT
    customer_id
  FROM orders_prc
  GROUP BY customer_id
  HAVING COUNT(order_id) > 2
)

SELECT
  name
FROM customers_prc
WHERE customerid IN (
  SELECT
    customer_id
  FROM valid_customers
); -- Correct!
```

## Problem 3: Gap Between Orders

### Prompt

```text
Identify the customer who had the longest gap between first and last orders. Provide their name and the duration of the gap.

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
  DATEDIFF(
    MAX(o.order_date)
    OVER (PARTITION BY o.customer_id),
    MIN(o.order_date)
    OVER (PARTITION BY o.customer_id)
  ) AS date_diff
FROM customers_prc AS c JOIN orders_prc AS o
ON c.customerid = o.customer_id
ORDER BY date_diff DESC
LIMIT 1;-- Correct!

-- Alternative:
SELECT
  c.name,
  MAX(DATEDIFF(o2.order_date, o1.order_date)) AS LongestGap
FROM customers_prc c JOIN orders_prc o1
ON c.customerid = o1.customer_id
JOIN orders_prc o2
ON o1.customer_id = o2.customer_id
WHERE o2.order_date > o1.order_date
GROUP BY c.name
ORDER BY LongestGap DESC
LIMIT 1; -- Correct!
-- As in the posted solution, self join is an alternative approach to solving the problem.
-- The above solution involving window functions is not wrong, but may take longer to implement. It's also more complicated.
-- When using self join, each row in one table is compared to all rows in the other table.
-- To limit the results to positive date differences, WHERE is used.
-- Once the tables are joined and filtered as above, you can find the max date difference across all customers.
-- Joining that self join with the customers table allows you to get the customer's name.
-- In summary, in addition to analyzing hierarchical data, self joins can also be helpful when you need to make comparisons within the same table.
-- The grouping gets the longest gap per customer, then the order by and limit gets you the max among those customers.
```

## Problem 4: Revenue Fluctuation

### Prompt

```text
Identify the customer with the most fluctuating order amounts. Calculate the difference between their highest and lowest order amounts.

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
  MAX(o2.amount - o1.amount) AS RevenueFluctuation
FROM customers_prc c JOIN orders_prc o1
ON c.customerid = o1.customer_id JOIN orders_prc o2
ON o1.customer_id = o2.customer_id
WHERE o2.amount > o1.amount
GROUP BY c.name
ORDER BY RevenueFluctuation DESC
LIMIT 1; -- Correct!

-- Alternative:
SELECT 
  c.name,
  MAX(o.amount) - MIN(o.amount) AS RevenueFluctuation
FROM customers_prc c JOIN orders_prc o
ON c.customerid = o.customer_id
GROUP BY c.name
ORDER BY RevenueFluctuation DESC
LIMIT 1;
-- This solution is simpler and avoids a self join. For each customer, it simply finds the largest and smallest order,
-- then calculates the difference using MAX and MIN and calls that RevenueFluctuation.
```

## Problem 5: Frequency of High-Value Orders

### Prompt

```text
Find the number of orders with an amount greater than $100 for each customer.

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
  COUNT(o.order_id) AS HighValueOrders
FROM customers_prc AS c JOIN orders_prc AS o
ON c.customerid = o.customer_id
WHERE o.amount > 100
GROUP BY c.customerid; -- Correct!
```

## Problem 6: Orders on Weekends

### Prompt

```text
Calculate the number of orders placed on weekends (Saturday and Sunday). Display the number of orders on weekends and total number of orders.

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
  SUM(
    CASE
      WHEN WEEKDAY(order_date) IN (5,6) THEN 1
      ELSE 0
    END
  ) AS WeekendOrders,
  COUNT(order_id) AS TotalOrders
FROM orders_prc; -- Correct!
```

## Problem 7: Orders on Holidays

### Prompt

```text
Determine if there are any orders placed on U.S. federal holidays (e.g., Independence Day, Thanksgiving) and list them.

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
  YEAR(order_date) AS order_year,
  DATE_FORMAT(order_date, '%m-%d') AS order_date_formatted,
  COUNT(order_id) AS order_count
FROM orders_prc
GROUP BY order_date, order_date_formatted
HAVING DATE_FORMAT(order_date, '%m-%d') IN (
  '01-01',  -- New Year's Day
  '07-04',  -- Independence Day
  '11-11',  -- Veterans Day
  '11-23',  -- Thanksgiving Day
  '12-25',  -- Christmas Day
  '01-20',  -- Inauguration Day (every 4 years)
  '01-16',  -- Martin Luther King Jr. Day
  '02-20',  -- Washington's Birthday
  '05-30',  -- Memorial Day
  '09-05',  -- Labor Day
  '10-10',  -- Columbus Day
  '11-11',  -- Veterans Day
  '12-31'   -- New Year's Eve (not a federal holiday, but sometimes observed)
); -- Correct!
-- You also could have done the filtering in a WHERE clause, before GROUP BY.
-- In a real interview, ask for a list of federal holidays to filter by if not given one.
```

## Problem 8: Quarterly Revenue Trends

### Prompt

```text
Analyze the quarterly revenue trends for 2022 & 2023 and provide the total revenue for each quarter.

  Table: orders_prc
  ColumnName		Datatype
  order_id 		INT
  customer_id     INT
  order_date 		DATE
  amount 			INT
```

### Solution and working notes

```sql
WITH quarters AS (
  SELECT
    *,
    (
      CASE
        WHEN MONTH(order_date) BETWEEN 1 AND 3 THEN CONCAT(YEAR(order_date), 'Q1') 
        WHEN MONTH(order_date) BETWEEN 4 AND 6 THEN CONCAT(YEAR(order_date), 'Q2') 
        WHEN MONTH(order_date) BETWEEN 7 AND 9 THEN CONCAT(YEAR(order_date), 'Q3') 
        WHEN MONTH(order_date) BETWEEN 10 AND 12 THEN CONCAT(YEAR(order_date), 'Q4') 
      END
    ) AS Quarter
  FROM orders_prc
  WHERE YEAR(order_date) IN (2022, 2023)
  ORDER BY order_date
)

SELECT
  Quarter,
  SUM(amount) AS TotalRevenue
FROM quarters
GROUP BY Quarter; -- Correct!

-- Alternative
SELECT
  CONCAT(YEAR(order_date), 'Q', QUARTER(order_date)) AS Quarter,
  SUM(amount) AS TotalRevenue
FROM orders_prc
WHERE YEAR(order_date) IN (2022,2023)
GROUP BY Quarter;
-- Didn't know about the QUARTER function. Seems a lot simpler. Don't need the CTE with that function.
-- You can use the 'Quarter' alias in GROUP BY because this specific version of SQL allows it. This is not always the case.
```

## Problem 9: Customers With Orders in Consecutive Years and Repeated Orders

### Prompt

```text
Find customers who placed orders in at least two consecutive years and had more than one order in at least one of those years.

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
WITH customer_stats AS (
  SELECT
    customer_id,
    YEAR(order_date) AS order_year,
    SUM(order_id) AS num_orders
  FROM orders_prc
  GROUP BY customer_id, order_year
),
year_over_year_stats AS (
  SELECT
    customer_id,
    order_year,
    num_orders,
    order_year - LAG(order_year, 1, order_year) OVER (PARTITION BY customer_id ORDER BY order_year) AS order_gap,
    LAG(num_orders, 1, 0) OVER (PARTITION BY customer_id ORDER BY order_year) AS prev_year_orders
  FROM customer_stats
)

SELECT
  DISTINCT name
FROM customers_prc c JOIN year_over_year_stats y
ON c.customerid = y.customer_id
WHERE y.order_gap = 1 AND (y.num_orders > 1 OR y.prev_year_orders > 1); -- Correct!
```

## Problem 10: Most Valued City

### Prompt

```text
Identify the city from which customers have placed orders the most. If there are matches select the first city in alphabetical order.

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
  c.city
FROM customers_prc c LEFT JOIN orders_prc o
ON c.customerid = o.customer_id
WHERE order_id IS NOT NULL
GROUP BY c.city
ORDER BY SUM(o.order_id) DESC
LIMIT 1; -- Correct!

-- Alternative:
SELECT
  city
FROM (
  SELECT
    SUM(o.order_id) AS total_orders,
    c.city
  FROM customers_prc c LEFT JOIN orders_prc o
  ON c.customerid = o.customer_id
  WHERE order_id IS NOT NULL
  GROUP BY c.city
  ORDER BY total_orders DESC
  LIMIT 1
) AS subquery;
-- We perform a subquery here because we don't need total_orders in the final result, so we do it in the subquery, then only select
-- city (the column/columns we need) in the main query.
```

## Problem 11: Expertise in Python

### Prompt

```text
List all employees who have expertise in Python among those who work in the "Data Science" department.

  Tables: qualificationandskills
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR


  Tables: employee_dimension
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
  e.Employee_ID AS employee_id,
  e.role,
  e.Department AS department
FROM employee_dimension e JOIN qualificationandskills q
USING(Employee_ID)
WHERE q.Technical_Skills LIKE '%Python%' AND
e.Department = 'Data Science'; -- Correct!
```

## Problem 12: Completed More Projects

### Prompt

```text
List the employees who completed more than 5 projects in January 2023, along with their roles and salaries.

  Tables: employee_dimension
  ColumnName		    Datatype
  employee_id 		INT
  role 			    VARCHAR
  department 		    VARCHAR
  experience_years 	INT
  salary 			    INT
  previousyearsalary 	INT
  location 		    VARCHAR

  Tables: employee_performance_fact 
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
  e.Employee_ID AS employee_id,
  e.Role AS role,
  e.Salary AS salary
FROM employee_dimension e JOIN employee_performance_fact p
USING(Employee_ID)
WHERE Month = 'January' AND Year = 2023 AND Projects_Completed > 5; -- Correct!
```

## Problem 13: Most Valued Employee

### Prompt

```text
Find the employees who have a Master's degree and worked more than 150 hours in January 2023, along with their roles.

  Tables: employee_performance_fact
  ColumnName		    	Datatype
  employee_id 			INT
  month 				    VARCHAR
  year 				    INT
  projects_completed 		INT
  hours_worked 			INT
  revenue_generated 		INT


  Tables: employee_dimension
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
  e.employee_id,
  e.role
FROM employee_dimension e JOIN qualificationandskills q
USING(Employee_ID) JOIN employee_performance_fact p
USING(Employee_ID)
WHERE q.education_level = "Master's Degree"
AND p.hours_worked > 150
AND p.month = 'January'
AND p.year = 2023; -- Correct!
```

## Problem 14: Most Experience in Data Science Department

### Prompt

```text
List employees who have expertise in both "Python" and "R" among those who work in the "Data Science" department.

  Tables: employee_dimension
  ColumnName		    Datatype
  employee_id 		INT
  role 			    VARCHAR
  department 		    VARCHAR
  experience_years 	INT
  salary 			    INT
  previousyearsalary 	INT
  location 		    VARCHAR

  Tables: qualificationandskills 
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR
```

### Solution and working notes

```sql
SELECT
  e.Employee_ID,
  e.role
FROM employee_dimension e JOIN qualificationandskills q
USING(Employee_ID)
WHERE e.department = 'Data Science'
AND (
  q.technical_skills LIKE '%R, %' OR
  q.technical_skills LIKE '%, R'
); -- Correct!
```

## Problem 15: Skills of a Data Analyst

### Prompt

```text
Find employees in the "Data Analyst" role who have expertise in "SQL" or "Python."

  Tables: qualificationandskills
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR

  Tables: employee_dimension
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
  e.employee_id,
  e.role
FROM employee_dimension e JOIN qualificationandskills q
USING(Employee_ID)
WHERE e.role = 'Data Analyst'
AND (
  q.Technical_Skills LIKE '%SQL%' OR q.Technical_Skills LIKE '%Python%'
); -- Correct!
```

## Problem 16: Ph.D Employees Revenue

### Prompt

```text
Identify the employees who have a Ph.D. degree and generated more than $100,000 in revenue in January 2023.
  Expected Output Columns: employee_id, role

  Tables: employee_dimension
  ColumnName		    Datatype
  employee_id 		INT
  role 			    VARCHAR
  department 		    VARCHAR
  experience_years 	INT
  salary 			    INT
  previousyearsalary 	INT
  location 		    VARCHAR

  Tables: employee_performance_fact 
  ColumnName		    	Datatype
  employee_id 			INT
  month 				    VARCHAR
  year 				    INT
  projects_completed 		INT
  hours_worked 			INT
  revenue_generated 		INT

  Tables: qualificationandskills 
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR
```

### Solution and working notes

```sql
SELECT
    e.employee_id,
    e.role
FROM employee_dimension e JOIN qualificationandskills q
USING(Employee_ID) JOIN employee_performance_fact p
USING(Employee_ID)
WHERE q.education_level = 'Ph.D.' AND p.month = 'January' AND p.year = 2023
AND p.revenue_generated > 100000; -- Correct!
```

## Problem 17: Months With No Orders

### Prompt

```text
Identify the months in 2023 with no orders and provide a list of these months.

  Table: orders_prc
  ColumnName		Datatype
  order_id 		INT
  customer_id     INT
  order_date 		DATE
  amount 			INT
```

### Solution and working notes

```sql
-- Unable to come up with solution

WITH MonthsIn2023 AS (
  SELECT '2023-01' AS YearMonth
  UNION SELECT '2023-02'
  UNION SELECT '2023-03'
  UNION SELECT '2023-04'
  UNION SELECT '2023-05'
  UNION SELECT '2023-06'
  UNION SELECT '2023-07'
  UNION SELECT '2023-08'
  UNION SELECT '2023-09'
  UNION SELECT '2023-10'
  UNION SELECT '2023-11'
  UNION SELECT '2023-12'
)

SELECT DISTINCT m.YearMonth AS MissingMonth
FROM MonthsIn2023 m
LEFT JOIN orders_prc o ON m.YearMonth = DATE_FORMAT(order_date, '%Y-%m')
WHERE o.order_id IS NULL; -- Solution
-- We create the YearMonth column by initializing as a column containing one row with '2023-01'.
-- Next, we use UNION and SELECT to add the other 11 months. We save this in a CTE.
-- In the main query, we LEFT JOIN the CTE with orders_prc on the date format.
-- The left join will only display order IDs for dates that match the provided format. All other order IDs will be null.
-- By filtering on 'o.order_id IS NULL', we only get the months with no orders (no order IDs)
```

## Problem 18: Average Revenue in January 2023

### Prompt

```text
Calculate the average revenue generated per project in January 2023 by employees with a Ph.D.

  Table: employee_performance_fact 
  ColumnName		    	Datatype
  employee_id 			INT
  month 				    VARCHAR
  year 				    INT
  projects_completed 		INT
  hours_worked 			INT
  revenue_generated 		INT

  Table: qualificationandskills 
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR
```

### Solution and working notes

```sql
SELECT
  SUM(revenue_generated)/SUM(projects_completed) AS Avg_Revenue_Per_Project
FROM employee_performance_fact
WHERE employee_id IN (
  SELECT
    employee_id
  FROM qualificationandskills
  WHERE education_level = 'Ph.D.'
) AND month = 'January' and year = 2023; -- Correct!
-- Make sure to look at all columns in the table carefully and read the question carefully.
-- First, we thought total revenue generated per project was just AVG(revenue_generated). This is just the average revenue generated.
-- The Average revenue generated PER PROJECT is the total revenue divided by the total number of projects.
```

## Problem 19: Employees Completed Most Projects

### Prompt

```text
Find the employee who completed the most projects in January 2023 and has expertise in both Python and SQL, and who also has a salary greater than 73000.

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

  Table: qualificationandskills 
  ColumnName		    	Datatype
  employee_id 			INT
  education_level 		VARCHAR
  graduation_year 		INT
  field_of_study 			VARCHAR
  technical_skills 		VARCHAR
```

### Solution and working notes

```sql
SELECT
  employee_id,
  role,
  department
FROM employee_dimension
WHERE employee_id IN (
  SELECT
    employee_id
  FROM employee_performance_fact p JOIN qualificationandskills q
  USING(Employee_ID)
  WHERE p.month = 'January' AND p.year = 2023
  AND q.technical_skills LIKE '%Python%'
  AND q.technical_skills LIKE '%SQL%'
) AND salary > 73000; -- Correct!
-- Although correct, it doesn't account for the employee who completed the most projects.

-- Alternative:
SELECT
  e.employee_id,
  e.role,
  e.department
FROM employee_performance_fact p JOIN qualificationandskills q
USING(Employee_ID) JOIN employee_dimension e
USING(Employee_ID)
WHERE p.month = 'January' AND p.year = 2023
AND q.technical_skills LIKE '%Python%'
AND q.technical_skills LIKE '%SQL%'
AND e.salary > 73000
ORDER BY p.projects_completed DESC
LIMIT 1;
```

## Problem 20: Average Order Amount by Month

### Prompt

```text
Calculate the average order amount for each month in 2022 and list the results.

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
    DATE_FORMAT(order_date, '%Y-%m') AS Month,
    AVG(amount) AS AvgOrderAmount
FROM orders_prc
WHERE YEAR(order_date) = 2022
GROUP BY MONTH(order_date); -- Correct!
```
