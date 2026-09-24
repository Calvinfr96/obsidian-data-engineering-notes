---
aliases:
  - "sql_intermediate_lessons"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Notes/sql_intermediate_lessons.md"
imported: 2026-09-24
---

# SQL Intermediate Lessons

Review: [[02_technical_prep/sql/SQL Optimization Principles|SQL Optimization Principles]] · [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#GROUP BY]]
- [[#COUNT OPERATOR]]
- [[#SUBQUERIES]]
- [[#MAX OPERATOR]]
- [[#MIN OPERATOR]]
- [[#SUM OPERATOR]]
- [[#AVG OPERATOR]]
- [[#HAVING CLAUSE]]
- [[#SUBQUERY WITH AGGREGATED FUNCTIONS]]
- [[#CASE STATEMENTS]]
- [[#LEFT JOIN]]
- [[#RIGHT JOIN]]
- [[#INNER JOIN]]
- [[#FULL OUTER JOIN OR OUTER JOIN]]
- [[#JOIN WITH WHERE]]
- [[#JOIN WITH COMPARISON OPERATOR]]
- [[#DISTINCT]]
- [[#JOIN WITH MULTIPLE KEYS]]
- [[#SELF JOIN]]
- [[#UNION]]
- [[#CURDATE & CURTIME]]
- [[#DATE ARITHMETIC FUNCTIONS]]
- [[#DATE FORMAT]]

## GROUP BY
- The SQL `GROUP BY` Clause is a tool in SQL that allows you to group rows of data based on the values in one or more columns of a table. This function comes in handy especially when performing aggregate functions on specific groups of data, such as calculating totals, averages, counts, or other statistical measures.
- The `GROUP BY` Clause is commonly used in conjunction with aggregate functions such as `SUM`, `COUNT`, and `AVG`.
- The basic syntax of the `GROUP BY` Clause is as follows:
  ```sql
  SELECT column1, column2, ..., aggregate_function(columnX) AS alias FROM table_name GROUP BY column1, column2, ...;
  ```
- The `GROUP BY` Clause can be applied to one or more columns, allowing you to group rows with similar values. The columns represent the factors by which rows are being grouped. All factors must be met for a row to be added to a group. For example:
  ```sql
  SELECT department, COUNT(*) AS employee_count FROM employees GROUP BY department;
  ```
  - This query will display the department and employee count of each department in the table.
  - The `GROUP BY` Clause groups employees (rows) by their respective department, the `COUNT(*)` Clause counts the number of employees in each department.
- The `GROUP BY` Clause is typically used in conjunction with aggregate functions such as `COUNT`, `SUM`, `AVG`, `MIN`, or `MAX`. This enables you to calculate aggregated values within each group. For example:
  ```sql
  SELECT department, AVG(salary) AS average_salary FROM employees GROUP BY department;
  ```
  - This query groups employees by department, then calculates and displays the average salary within each department.
- When using `GROUP BY`, be aware that null values are treated as a single group. To exclude null values from grouping, you can use the `WHERE` and `IS NOT NULL` Clauses before applying `GROUP BY`.

### Filtering Data Using the HAVING Clause
- You can further filter grouped data by using a `HAVING` Clause. This clause filters aggregated results, allowing you to include only groups which meet certain criteria and exclude those groups that do not. For example:
  ```sql
  SELECT department, AVG(salary) AS average_salary FROM employees GROUP BY department HAVING AVG(salary) > 50000;
  ```
  - This query groups employees by department, calculates the average salary within each department, and only includes those departments with an average salary greater than 50000 in the result set.

### Combining GROUP BY and ORDER BY
- You can use the `GROUP BY` and `ORDER BY` Clauses to sort grouped data based on specified criteria, making it easier to analyze results. For example:
  ```sql
  SELECT product_category, COUNT(*) AS product_count FROM products GROUP BY product_category ORDER BY product_count DESC;
  ```
  - This query will display the product count for each product category, sorted by product count in descending order.
  - Note the order of operations here. First, we select the columns to be displayed, applying any aggregate functions and aliases. Then we group rows according to one or more columns. Then we order by one or more columns in ascending or descending order.
## COUNT OPERATOR
- The SQL `COUNT` Function is a powerful tool that allows you to count the number of rows in either a table or group when used in conjunction with the `GROUP BY` Clause. This function can be useful for obtaining insights into the size of datasets, identifying the occurrence of specific conditions, and generating statistical summaries.
- The `COUNT` Function is most commonly used with `SELECT` and `GROUP BY` to perform aggregations and produce meaningful data analyses.
- The basic syntax of the `COUNT` Function is as follows:
  ```sql
  SELECT COUNT(column_name) AS count_result
  FROM table_name;
  ```
  - Here, count_result acts as an alias for the column that will contain the aggregation.

### Counting Rows in a Table
- The `COUNT` Function can be applied to a single column or using the asterisk (*), which applies the function to all columns in a table. In either case, the function will count the number of rows in the table. For example:
  ```sql
  SELECT COUNT(*) AS total_records FROM employees;
  ```
  - This query returns the total number of rows (employees) in the table and returns it as 'total_records'.
- You can also use the `COUNT` Function in conjunction with other conditional statements, such as a `WHERE` Clause, to count rows that meet specific criteria. For example:
  ```sql
  SELECT COUNT(*) AS active_employees FROM employees WHERE is_active = 1;
  ```
  - This query returns the total number of **active** employees in the employees table. Specifically, those with a 'is_active' attribute set to 1. The result is returned as 'active_employees'.
- The `COUNT` Function can also be used in conjunction with the `GROUP BY` Clause to count the numbers of rows within group based on a specific column's values, instead of counting the number of rows in the entire table. This can be useful for generating group-based statistics. For example:
  ```sql
  SELECT department, COUNT(*) AS employees_count FROM employees GROUP BY department;
  ```
  - This query counts the total number of employees in each department and returns it as 'employee_count'. The number of rows in the result set will depend on the number of _unique_ departments in the table.

### Handling NULL Values
- By default, the `COUNT` Function includes null values in its calculation. To exclude null values, you must use a specific column as the argument instead of an asterisk. For example:
  ```sql
  SELECT COUNT(salary) AS non_null_salaries FROM employees;
  ```
  - This query counts the number of employees with non-null salaries.
  ```sql
  SELECT COUNT(*) AS any_salary FROM employees;
  ```
  - This query will count all employees, with non and non-null salaries.
  ```sql
  SELECT COUNT(salary) AS null_salaries FROM employees WHERE salary IS NULL;
  ```
  - This query counts the number of employees without a specified salary.

### Counting Distinct Values
- To count distinct (unique) values within a column, you use the `COUNT` Function with `DISTINCT` as follows: `COUNT(DISTINCT column_name)`. This allows you to obtain unique occurrences within a column. For example:
  ```sql
  SELECT COUNT(DISTINCT department) AS distinct_departments FROM employees;
  ```
  - This query counts the number of distinct (non-null) departments in the employees table.
- `COUNT(DISTINCT)` can be computationally expensive compared to `COUNT(*)` because the database engine must sort and compare all values to find the unique ones.
- `COUNT(DISTINCT)` can also be applied to multiple columns by specifying all columns in parentheses as follows: `COUNT(DISTINCT (col1, col2))` or `COUNT(DISTINCT CONCAT(col1, col2)))` (syntax varies by database system).
## SUBQUERIES
- Subqueries, also known as nested queries, are a powerful tool in database management that allow you to embed one query within another. They allow you to retrieve data from multiple tables, perform complex calculations, and filter results based on complex conditions.
- A subquery is a query nested within another query, used to retrieve or manipulate data. They can be used in the `SELECT`, `FROM`, `WHERE`, and `HAVING` Clauses.

### Types of Subqueries
- **Single Row Subqueries:** Returns a single value (usually used for comparison).
  ```sql
  SELECT product_name, unit_price FROM products WHERE unit_price > (SELECT AVG(unit_price) FROM products);
  ```
  - Displays the product name and unite price of all products whose unit price is greater than the average unit price of all products.
- **Multiple Row Subqueries:** Returns multiple values (usually used for inclusion in a list).
  ```sql
  SELECT order_id FROM orders WHERE customer_id IN (SELECT customer_id FROM customers WHERE country = 'USA');
  ```
  - Displays order ID for all US customers (customers with a country of 'USA'). The subquery here is generating a list of US customers.
  - Note: The orders table must have a customer_id column for this query to work correctly. `WHERE customer_id` is being applied to the orders table while `SELECT customer_id` is being applied to the customers table (both tables must have the column).
- **Correlated Subqueries:** Reference values from the outer query within the subquery.
  ```sql
  SELECT employee_id, first_name, last_name FROM employees e WHERE salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id);
  ```
  - Displays the employee ID, first name, and last name of employees whose salary is greater than the average salary of employees whose department matches the employee from the outer query. In other words, it displays employees whose salary is greater than the average salary of their own department.
  - Here, `employees e` is functionally equivalent to `employees AS e`. It sets an alias for the 'employees' table as 'e'. This helps shorten the subquery, as you don't have write `employees.department_id`.
  - Although `employees e` works `employees AS` e is cleaner and enhances clarity.
  - In the subquery, `e.department_id` refers to the department of the current employee row being processed by the outer query.
- **Subqueries with `EXISTS`:** Check for the existence of certain conditions.
  ```sql
  SELECT product_name FROM products WHERE EXISTS (SELECT 1 FROM order_details WHERE product_id = products.product_id);
  ```
  - The outer query selects the names of products from the products table.
  - The subquery checks for the existence (`EXISTS`) of at least one record (`SELECT 1`) in the order_details table that corresponds to each product in the main query (products table).
    - Here, product_id refers to the product ID from the order_details table while `products.products_id` refers to the product ID from the products table in the main query. 
  - Overall, the query returns the names of all products that have been ordered at least once.
  - The `EXISTS` Operator returns true if the subquery returns one or more rows. It's efficient because it stops processing after it finds the first match.
  - By using `SELECT 1`, the query doesn’t need to retrieve actual values from the order_details table, only the existence of rows is sufficient, making it more efficient than `SELECT *` or `SELECT NULL`, which do the same thing in this context. The `SELECT 1` Clause simply indicates that you want to check for the existence of rows (it simply returns a column containing the value '1' for every row that matches the query's conditions). `SELECT 1` is primarily used in conjunction with `EXISTS`.


### Common Use Cases
- Data Filtering: Filtering results based on conditions within another table.
- Aggregations: Using subqueries within aggregate functions such as `SUM` or `AVG`.
- Data Comparison: Comparing values from different datasets.
- Conditional Checks: Checking for the existence of certain records.

### Benefits of Subqueries
- Enhanced Data Retrieval: Retrieve data from multiple tables in a single query.
- Complex Calculations: Perform calculations on multiple levels of data or multiple datasets concurrently.
- Granular Filtering: Filter data based on intricate conditions.

### Best Practices
- Understand the correlation between inner and outer queries when performing correlated subqueries.
- Optimize subqueries for performance by using appropriate indices.
## MAX OPERATOR
- The SQL `MAX` Function is a powerful tool that allows you to retrieve the largest value from a specified column in a table. The function is particularly useful when you need to find the highest value of a given attribute. The function is commonly used in conjunction with the `SELECT` Statement to perform aggregations and obtain meaningful insights from data.
- The basic syntax of the `MAX` Function is as follows:
  ```sql
  SELECT MAX(column_name) AS max_value FROM table_name;
  ```
- When applied to a specific column, the `MAX` Function returns the largest value in that column. The function can be applied to multiple data types, including numeric values, dates, or strings. For example:
  ```sql
  SELECT MAX(salary) AS max_salary FROM employees;
  ```
  - This query returns the highest salary of all employees and displays it as 'max_salary'.
- The `MAX` Function is often used with other Functions and Clauses such as `GROUP BY`. When used with `GROUP BY`, it returns the largest value from each group, instead of the entire table. For example:
  ```sql
  SELECT department, MAX(salary) AS max_salary FROM employees GROUP BY department;
  ```
  - This query returns the maximum salary of any employee in each department. In other words, the highest salary in each department.
- The `MAX` Function can also be combined with conditional statements, such as the `WHERE` Clause, to find the largest value based on specified criteria. For example:
  ```sql
  SELECT MAX(order_amount) AS largest_order_amount FROM orders WHERE order_status = 'Completed';
  ```
  - This query returns the largest order amount among completed orders, and returns the value as 'largest_order_amount'.

### Handling NULL Values
- The `MAX` Function ignores null values when determining the largest value within a given column. If all values in a specified column are null, the returned maximum value will also be null. For example:
  ```sql
  SELECT MAX(date_joined) AS latest_join_date FROM employees;
  ```
  - This query returns the most recent join date of any employee in the employees table, excluding any null values.

### Question 4: Alternative Solution
- Retrieve the employee details (employee_id, first_name, last_name, department, salary) of the employee with the highest salary in each department.
- Submitted (Correct) Solution:
  ```sql
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
    WHERE table1.department = table2.department
  );
  ```
- Alternative Solution:
  ```sql
  SELECT
    employee_id,
    first_name,
    last_name,
    department,
    salary
  FROM employees
  WHERE salary IN (
    SELECT
      MAX(salary)
    FROM employees
    GROUP BY department
  );
  ```
  - Instead using `MAX` within the subquery to calculate the maximum department salary for each row in the main query, we _can_ use the `GROUP BY` to retrieve a list of the highest department salaries. Then, we collect rows from the outer query (with the specified columns) where the salary is included in that list.
  - You _can't_ use `GROUP BY` in the subquery if you're performing a strict equality check, as in the submitted solution. This is because it will return multiple values and you can't equate a single value to a list.
  - You don't use `MAX(salary)` in `SELECT` because the other details won't necessarily be associated with the row that includes the maximum salary.
## MIN OPERATOR
- The SQL `MIN` Function that allows you to retrieve the smallest value in the specified column of a table. It is particularly useful when you need to find the minimum or smallest value of an attribute in a dataset.
- The `MIN` Function is commonly used in conjunction with `SELECT` to perform aggregations and obtain meaningful insights from data.
- The basic syntax of the `MIN` Function is as follows:
  ```sql
  SELECT MIN(column_name) AS min_value FROM table_name;
  ```
- When applied to a specific column, the `MIN` Function returns the smallest value present in that column. As with the `MAX` Function, it can be used on various data types, such as numeric values, dates, and strings. For example:
  ```sql
  SELECT MIN(salary) AS min_salary FROM employees;
  ```
  - This query will return the minimum salary of all employees in the table.
- The `MIN` Function can also be used with other SQL functions for more complex data retrieval. For example, it can be used with `GROUP BY` to find the smallest value in each group. For example:
  ```sql
  SELECT department, MIN(salary) AS min_salary FROM employees GROUP BY department;
  ```
  - This query will return the department and associated minimum salary of each department in the employees table.
  - First employees are grouped by department, then the minimum salary is calculated for employees in each department.
- The `MIN` Function can also be combined with conditional statements, such as `WHERE`, to find the smallest value of a dataset filtered based on specified criteria. For example:
  ```sql
  SELECT MIN(order_amount) AS smallest_order_amount FROM orders WHERE order_status = 'Completed';
  ```
  - This query finds the smallest order in the orders table. Specifically, the minimum order amount among completed orders.
  - The `WHERE` Clause filters orders by status, then the `MIN` Function aggregates those filtered order amounts and finds the smallest value.

### Handling NULL Values
- Like the `MAX` Function, the `MIN` Function will automatically ignore null values in a column when determining the minimum value. If all values in a column are, `MIN` will return null as well. For example:
  ```sql
  SELECT MIN(date_joined) AS earliest_join_date FROM employees;
  ```
  - This query finds the earliest join date among all employees who have a valid (non-null) join date.
## SUM OPERATOR
- The SQL `SUM` Function is a powerful tool that allows you to calculate the sum of _numeric_ values in the specified column of a table. The function is particularly useful for performing numerical aggregations on data, such as calculating total sales amounts, total expenses, or determining the sum of scores.
- The `SUM` Function is commonly used with `SELECT` to perform aggregations on data and produce meaning insights.
- The basic syntax of the `SUM` Function is as follows:
  ```sql
  SELECT SUM(numeric_column) AS sum_result FROM table_name;
  ```
- When applied to a specific _numeric_ column, `SUM` will calculate the sum of values within that column. For example
  ```sql
  SELECT SUM(sales_amount) AS total_sales FROM sales;
  ```
  - This query calculates the total of all sales amounts in the sales table.
- The `SUM` Function can also be used with conditional statements, such as `WHERE`, to calculate the sum of values that meet certain criteria. For example:
  ```sql
  SELECT SUM(quantity_sold) AS total_sold_laptops FROM products WHERE product_name = 'Laptop';
  ```
  - This query returns the total quantity sold among products whose name is 'laptop'. In other words, the total number of laptops sold.
- The `SUM` Function can also be used with `GROUP BY` to calculate the sum of values within a group of rows based on a certain column's values. For example:
  ```sql
  SELECT department, SUM(salary) AS total_salary_expense FROM employees GROUP BY department;
  ```
  - This query will return the department and sum of salaries within each department from the employees table.
  - First the rows are grouped by department. Then, the sum of salaries within each group is calculated and returned.

### Handling NULL Values
- The `SUM` Function automatically ignores null values in a column and only considers non-null values in its calculation. If an entire column is null, the sum will be null as well. For example:
  ```sql
  SELECT SUM(total_amount) AS total_revenue FROM orders;
  ```
  - This query retrieves the total revenue among orders with a non-null total_amount value.
## AVG OPERATOR
- The SQL `AVG` Function is a powerful tool that allows you to calculate the mean or average of _numeric_ values within a specified column of a table. It is particularly useful when you need to perform numerical aggregations, such as finding the average test score, calculating the average revenue, or calculating the mean salary.
- The `AVG` Function is commonly used with `SELECT` to perform aggregations and gain meaningful insights from data.
- The basic syntax of the `AVG` Function is as follows:
  ```sql
  SELECT AVG(numeric_column) AS average_result FROM table_name;
  ```
- When applied to a specific column, the function returns the average of all values within that column. For example:
  ```sql
  SELECT AVG(test_score) AS average_score FROM student_scores;
  ```
  - This query calculates the average score among all student scores.
- The `AVG` Function can also be used with conditional statements, such as `WHERE`, to perform more complex analysis based on specific criteria. For example:
  ```sql
  SELECT AVG(rating) AS average_rating FROM product_reviews WHERE product_id = 101;
  ```
  - This query returns the average rating among product reviews with a product ID of 101.
- When using `AVG` with `GROUP BY`, the average is calculated for each group, which can be helpful when generating group-level statistics. For example:
  ```sql
  SELECT department, AVG(salary) AS average_salary FROM employees GROUP BY department;
  ```
  - This query returns the department and average salary for each department in the employees table.

### Handling NULL Values
- The `AVG` Function ignores null values by default and only calculates the average among non-null values in a column. If an entire column is null, the returned average will also be null. For example:
  ```sql
  SELECT AVG(revenue) AS average_revenue FROM sales;
  ```
  - This query returns the average revenue among all sales with a non-null revenue.
## HAVING CLAUSE
- The SQL `HAVING` Clause is a powerful tool that allows you to filter the results of grouped data based on specified conditions. The function is particularly useful when applying filters to aggregated values obtained from `GROUP BY`.
- The main difference between `HAVING` and `WHERE` is that the former works on the level of groups while the latter works on the level of individual rows.
- The `HAVING` Clause is also used in conjunction with other aggregate functions such as `SUM`, `COUNT`, and `AVG` to refine data analysis and gain more precise insights.
- The basic syntax of the `HAVING` Clause is as follows:
  ```sql
  SELECT column1, column2, ..., aggregate_function(columnX) AS alias FROM table_name GROUP BY column1, column2, ... HAVING condition;
  ```
- The `HAVING` Clause is typically used to filter results obtained by `GROUP BY`. For example:
  ```sql
  SELECT department, AVG(salary) AS average_salary FROM employees GROUP BY department HAVING AVG(salary) > 50000;
  ```
  - This query returns the department and average salary among departments with an average salary greater than 50000
  - First the groups are created, including the department and average salary in each group. Then, the groups are filtered with `HAVING`, excluding groups with an average salary less than or equal to 50000 in the result set.
- The `HAVING` Clause is also used with aggregate functions such as `COUNT`, `SUM`, `AVG`, `MIN`, or `MAX`. This allows you to filter groups based on aggregated values. For example:
  ```sql
  SELECT product_category, COUNT(*) AS product_count FROM products GROUP BY product_category HAVING COUNT(*) > 100;
  ```
  - This query returns the product category and product count of those categories with more than 100 products.
- The `HAVING` Clause can also be combined with `WHERE` in addition to `GROUP BY` to create complex queries with multiple filters. For example:
  ```sql
  SELECT department, AVG(salary) AS average_salary FROM employees WHERE is_active = 1 GROUP BY department HAVING AVG(salary) > 50000;
  ```
  - This query will return the department and average salary for those departments containing only active employees with an average salary greater than 50000.
  - First the inactive employees are filtered out. Then the remaining employees are grouped by department and those groups with an average salary less than or equal to 50000 are filtered out.
- `HAVING` can also be used with Logical Operators such as `AND`, `OR`, and `NOT` to create more intricate conditions for filtering grouped data.

### Handling NULL Values
- Use caution when using the `HAVING` Clause with aggregate functions that involve null values. For accurate results, consider using the `COALESCE` or `IFNULL` functions to handle null values appropriately.
## SUBQUERY WITH AGGREGATED FUNCTIONS
- In SQL, using subqueries with aggregate functions provides a powerful way to aggregate data in multiple stages, allowing you to perform complex analyses with step-by-step calculations and comparisons.
- Using subqueries to aggregate data in stages allows you to break down intricate processes into manageable steps, gaining deeper insights and allowing more complex analysis.
- Aggregating data in stages involves breaking down complex calculations into manageable steps. Subqueries help achieve stage-by-stage data manipulation for refined analysis.

### Common Use Cases
- **Cumulative Summation:** Calculating running totals for various groups.
- **Incremental Averages:** Finding the average as new data points are added.
- **Comparative Analysis:** Comparing aggregates between different groups.
- **Stages of Aggregation: Extract, Apply, Combine**
  1. Extract relevant data for each group.
  2. Apply aggregate functions to the extracted groups.
  3. Combine the results or perform further calculations.

### Practice Examples
- Example 1 - Cumulative Sales by Month:
  ```sql
  SELECT
    month,
    SUM(amount) AS monthly_sales
  FROM (
    SELECT
      DATE_FORMAT(order_date, '%Y-%m') AS month,
      order_amount AS amount
    FROM orders
  ) AS stage1
  GROUP BY month;
  ```
  - Instead of using an actual table with `FROM`, this query uses a subquery, which returns a table based on the conditions specified in the subquery.
  - The subquery returns a table containing "month" and "amount" columns, with rows for every order (no WHERE condition is applied).
  - Then, the main query uses those results, selects the month and total amount from each month, grouping the results by month.
  - Notice how the outer query uses the column aliases from the subquery.
- Example 2 - Incremental Average Rating:
  ```sql
  SELECT
    rating_date,
    AVG(rating) OVER (ORDER BY rating_date) AS avg_rating
  FROM (
    SELECT
      rating_date,
      AVG(rating) AS rating
    FROM ratings
    GROUP BY rating_date
  ) AS stage1;
  ```
  - Here, the `OVER` Clause allows you to perform aggregations across groups (partitions) of rows without collapsing the rows as `GROUP BY` would. This would be useful, for example, if you wanted to display total category purchases for every item instead of each category.
  - The subquery returns a table with a rating_date column and a rating column containing the average rating for each rating date.
  - The main query takes this table, then forms a table with rating_date and the average rating for each rating date. The average in the main query represents the cumulative average rating (The average in the inner query represents the "rating" (average among multiple ratings) for that day. The cumulative average is the average of those averages, representing the average across _all_ dates).
  - Since there was no partition explicitly defined in the `OVER`Clause, the entire table (subquery result) acts as the default partition.
  - The `OVER` is necessary in the main query because it only calculates the average for rows up to that point.
    - For the first row, it's the just the average rating of the first row from the inner table.
    - For the second row, it's the average rating of the first two rows from the inner table.
    - For the third row, it's the average rating the the first three rows from the inner table, and so on.
    - This is what is meant by _cumulative_ average.
  - Because of the `ORDER BY` specified in the `OVER` Clause, the final output will be in chronological order.
- Example 3 - Comparative Revenue Growth:
  ```sql
  SELECT
    region,
    revenue_2022,
    revenue_2023,
    (revenue_2023 - revenue_2022) / revenue_2022 AS growth_rate
  FROM (
    SELECT
      region,
      SUM(
        CASE 
          WHEN year = 2022 THEN revenue
        END
      ) AS revenue_2022,
      SUM(
        CASE
          WHEN year = 2023 THEN revenue
        END
      ) AS revenue_2023
    FROM sales
    GROUP BY region
  ) AS stage1;
  ```
  - Here, the `CASE WHEN END` statements act on each row, applying if-then-else logic similar to a Java `switch case` statement.
    - The purpose of these statements in this query is to group revenue in each region by year, then sum each grouping (each statement creates its own group).
  - The inner query creates a table that includes the region, sum of 2022 revenue for that region, and sum of 2023 revenue for that region.
  - The main query takes this table and displays the region, 2022 revenue, 2023 revenue, and growth rate.

### Benefits of Aggregating in Stages
- Complex Analysis: Break down intricate calculations into simpler steps.
- Improved Efficiency: Optimize queries for performance in each stage.
- Granular Control: Gain a deeper understanding of each data manipulation step.

### Best Practices
- Plan your aggregation process step by step before implementing subqueries. This means writing pseudocode.
- Optimize queries for performance by using appropriate indexes.
- Test and validate the results of each stage for accuracy.
## CASE STATEMENTS
- The SQL `CASE` Statement is a powerful tool that allows you to apply conditional logic to your queries. It is particularly useful when you need to perform different actions based on specific conditions within a dataset.
- The `CASE` Statement allows you to create custom (boolean) expressions, transform data, and create different result sets based on various conditions.
- The basic syntax of the `CASE` Statement is as follows:
  ```sql
  SELECT column1, column2, ...,    
  CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2 ...        
    ELSE default_result    
  END AS alias
  FROM table_name;
  ```
  - Here, the `CASE` Statement **creates a "custom" column (named 'alias')** that is generated based on conditional logic. For each row in that column:
    - If condition 1 is true, then result 1 is selected.
    - If condition 2 is true, then result 2 is selected.
    - If neither condition is true, then the default result is selected.
    - **The result can be anything you want it to be. It doesn't have to be an existing attribute within the dataset.**
    - **Always assign an alias to your `CASE` Statements since it would be impossible to refer to the new column without it.**
- The `CASE` Statement checks a single expression against multiple conditions and returns the result **associated with the first condition that evaluates to true**. If no condition is satisfied, then the `ELSE` Clause provides a default condition. When no `ELSE` Clause is provided, it is implicitly assumed to be `ELSE NULL`. For example:
  ```sql
  SELECT product_name,       
  CASE 
    WHEN category_id = 1 THEN 'Electronics'           
    WHEN category_id = 2 THEN 'Clothing'           
    WHEN category_id = 3 THEN 'Home and Kitchen'           
    ELSE 'Other'       
  END AS category
  FROM products;
  ```
  - This query returns a table with a product name column and category column. The category column is populated by adding Electronics to the row if category ID is 1, Clothing to the row if category ID is 2, and so on.
  - In essence, the query will categorize products based on category ID and display the associated category name. If a product does not match any of the specified category IDs, then it is assigned a category of Other.

### Searched CASE Statement
- The searched `CASE` Statement evaluates multiple conditions independently and returns the result of the first condition that is true. There is no specific expression being compared in a searched `CASE` Statement. For example:
  ```sql
  SELECT product_name,       
  CASE
    WHEN price >= 1000 THEN 'Expensive'           
    WHEN price >= 500 AND price < 1000 THEN 'Moderate'           
    ELSE 'Affordable'       
  END AS price_category
  FROM products;
  ```
  - The above query selects product name and generates product category. Product category is generated by by comparing the price in each row to the specified amount in each condition, then assigning the associated value.
  - The resulting table displays the product name and associated price category for each product.

### Using CASE Statements with Aggregate Functions
- The `CASE` Statement can be used with aggregate functions to perform conditional aggregations. For example:
  ```sql
  SELECT
    department,       
    COUNT(*) AS total_employees,       
    SUM(
      CASE
        WHEN is_manager = 1 THEN 1
        ELSE 0
      END
    ) AS total_managers
  FROM employees
  GROUP BY department;
  ```
  - This query returns a table that includes the department, total employees within that department, and total managers within that department.
  - The total employees and managers per department is achieved with `GROUP BY`.
  - The `CASE` Statement here is being used with the `SUM` aggregate function to calculate the total number of managers in each department.

### Nesting CASE Statements
- `CASE` Statements can be nested to perform more complex conditional logic. For example:
  ```sql
  SELECT
    order_id,       
    CASE
      WHEN order_status = 'Cancelled' THEN 'Cancelled'           
      ELSE (
        CASE
          WHEN is_shipped = 1 THEN 'Shipped'
          ELSE 'Pending'
        END
      )   
    END AS order_status
  FROM orders;
  ```
  - This query selects order ID and order status. The status for each order is determined as follows:
    - If the order status is 'Cancelled', then the status is set to 'Cancelled'.
    - If the order is shipped, then the status is set to 'Shipped'.
    - Otherwise, the status is set to 'Pending'.

### Using CASE in UPDATE Statements
- The `CASE` Statement can be used with the `UPDATE` Statement to modify data based on specific conditions. For example:
  ```sql
  UPDATE products
  SET stock_status = (
    CASE
      WHEN stock_quantity > 0 THEN 'In Stock'
      ELSE 'Out of Stock
    END
  );
  ```
  - The above statement will update stock status based on the following conditions:
    - If the stock quantity is greater than 0, the status is set to 'In Stock'.
    - Otherwise, it is set to 'Out of Stock'.
## LEFT JOIN
- The SQL `LEFT JOIN` Function is a powerful tool that allows you to combine data from two or more tables based on a common column, even if there are incomplete matches in the **second table**.
- The function is particularly useful in cases where you need to include records from one table and include any matching records from another table.
  - Note: Here, "matching" means records (rows) that satisfy the conditions of the `ON` Clause in the function. `NULL` will be displayed for **all** values (columns) of non-matching records (rows). In other words, for non-matching rows, only columns from the "prioritized" table are preserved.
  - `LEFT JOIN` is much more commonly used than `RIGHT JOIN` because `LEFT JOIN` aligns with the left-to-right reading order of the code. `RIGHT JOIN` is considered to be less readable an "anti-pattern".
- The `LEFT JOIN` is commonly used with `SELECT` to merge data from two or more tables in order to gain comprehensive insights from your database.
- The basic syntax of the `LEFT JOIN` Function is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 LEFT JOIN table2 ON table1.columnX = table2.columnY;
  ```
  - The columns selected can be from either table in the join. It's best to be explicit when listing columns (`table.column`) to avoid ambiguity-related errors.

### Joining Tables With Complete Matches
- The `LEFT JOIN` will retrieve all records from the left table and any **matching** records from the right table based on the condition specified in the `ON` Clause.
  - Here, "retrieve" means to return the value of the row from that column. When two corresponding rows fail the `ON` check, `NULL` will be returned for values in the right table while values in the left table will be preserved.
  - The columns actually returned by the query depend on those that are specified in the `SELECT` Statement. If you use `SELECT *`, **all columns from both tables will appear in the result**. This is because the `JOIN` Function creates a transient table that includes all columns from both tables. The column being used in the `ON` Clause will appear twice in the results.
  - **It's best to avoid using `SELECT *` with `JOIN` and instead select specific columns**.
- Example `LEFT JOIN`:
  ```sql
  SELECT customers.customer_id, orders.order_id, orders.order_date FROM customers LEFT JOIN orders ON customers.customer_id = orders.customer_id;
  ```
  - This query returns the customer ID, order ID, and order date for all customers, including those that haven't placed any orders. Non-matching orders will display `NULL` for order ID and order date, since they're from the orders table.

### Retrieving Data From Multiple Tables
- You can use `LEFT JOIN` to combine data from more than two tables extending the `JOIN` Clauses and `ON` Conditions. For example:
  ```sql
  SELECT customers.customer_id, customers.customer_name, orders.order_id, order_items.product_name FROM customers LEFT JOIN orders ON customers.customer_id = orders.customer_id LEFT JOIN order_items ON orders.order_id = order_items.order_id;
  ```
  - This query returns the customer information, order details, and product name for all orders, including those that do not have related product items.
  - In this query, the joins are performed in order. The customers table is joined with the orders table, then that resultant table is joined with the order items table.
  - In the first join, all customer details are preserved and only matching order details are preserved.
  -  In the second join, all orders associated with a customer (the rows from the resultant table of the first join) are preserved and only matching order item details are preserved.
  - **Join order is never guaranteed and depends on how the database system chooses to optimize the query**.

### Combining LEFT JOIN With Other Joins
- The `LEFT JOIN` can be combined with other types of joins such as `INNER JOIN` and `RIGHT JOIN` to create more complex queries that suit your data analysis needs.

### Handling NULL Values
- When using `LEFT JOIN`, be prepared to handle null values from columns in the right table, which are associated with non-matching records. Use appropriate functions, such as `COALESCE` or `IFNULL` to handle null values when performing calculations or aggregations.

### LEFT JOIN vs INNER JOIN
- `LEFT JOIN` includes all records from the left table and matching records from the right table. Non-matching records in the right table are assigned a value of `NULL`.
- `INNER JOIN` only includes records with matching values in both tables. Non-matching records are discarded from the results, they are not assigned a value of `NULL`.
- As an example, when joining a customers table with an orders table:
  - `INNER JOIN` will only show customers who have placed at least one order. Customers without orders are hidden.
  - `LEFT JOIN` will show all customers. Order details for customers who have never placed an order will be assigned a value of `NULL`.
- `NULL` values will typically only appear in an `INNER JOIN` unless there are actual null values in the dataset.
## RIGHT JOIN
- The SQL `RIGHT JOIN` Function is a powerful tool that allows you to combine data from two or more tables based on a common column, even if there are incomplete matches in the **first table**.
- As with `LEFT JOIN`, this function is useful for merging data between two different tables based on a condition involving a common column when used with `SELECT`. It helps you perform more complex data analysis and retrieve rows from the first table regardless of matching values. Non- matching values will be set to `NULL`.
- The basic syntax of the `RIGHT JOIN` Function is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 RIGHT JOIN table2 ON table1.columnX = table2.columnY;
  ```
  - The main difference with `LEFT JOIN` is which columns are preserved (included regardless of matching). In a `RIGHT JOIN`, the second table is preserved while the first table is assigned null values for non-matching rows.

### Joining Tables With Complete Matches
- The `RIGHT JOIN` retrieves all records from the second table and matching records from the first. Non-matching records from the first are included, but assigned a value of `NULL`. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders RIGHT JOIN customers ON orders.customer_id = customers.customer_id;
  ```
  - This query retrieves order ID, order date, and customer name for customers who have placed orders. Any records for customers who have not placed orders will be assigned a null order ID and order date.

### Retrieving Data From Multiple Tables
- You can retrieve data from more than two tables using `RIGHT JOIN` by extending the join operation with additional `JOIN` Clauses and `ON` Conditions. For example:
  ```sql
  SELECT order_items.order_id, order_items.product_name, customers.customer_name, shipments.shipment_status FROM order_items RIGHT JOIN orders ON order_items.order_id = orders.order_id RIGHT JOIN customers ON orders.customer_id = customers.customer_id LEFT JOIN shipments ON orders.order_id = shipments.order_id;
  ```
  - The involved tables are: order_items, orders, customers, and shipments.
  - First join: An order's order item details. Second join: A customer's order item details. Third join: A shipment's order details.
  - Order details are always preserved in the first join, customer details are always preserved in the second join, and the customer's order item details are always preserved in the third join.
  - The query is designed to produce a comprehensive view of orders along with their associated items, customer information, and shipment statuses, prioritizing the inclusion of customers and orders (both the second table in the first two joins) even when some orders might not have items or shipments associated with them. This ensures that the data provides insight into all customers and their orders, while also accounting for items (first table in the first right join) and shipment details (second table in the last left join) whenever available.

### Combining RIGHT JOIN With Other Joins
- `RIGHT JOIN` can be combined with other joins such as `LEFT JOIN` and `INNER JOIN` to create more complex queries for more thorough data analysis.

### Handling NULL Values
- Be prepared to deal with null values, especially in columns from the left table, when working with `RIGHT JOIN`. To properly handle null values when performing calculations or aggregations, you can use `COALESCE` or `IFNULL`.

### RIGHT JOIN vs LEFT JOIN
- The primary difference between `RIGHT JOIN` and `LEFT JOIN` is which columns are preserved in the case of non-matching rows. For right joins, the columns from the right table are always preserved. For left joins, columns from the left table are always preserved.
## INNER JOIN
- The SQL `INNER JOIN` Function is a powerful tool that allows you to combine data from two or more tables using a common column, including **only matching records** from both tables.
  - You're much less likely to see null values in the result set of an `INNER JOIN` than a left or right join, unless the actual data contains null values.
- This function is particularly useful in scenarios where you need to generate results specifically for data that exists in both tables, while excluding data that does not.
- It is most commonly used with the `SELECT` Statement to merge and gain insights from data.
- The basic syntax of the `INNER JOIN` Function is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 INNER JOIN table2 ON table1.columnX = table2.columnY;
  ```
  - Non-matching records from **both** tables is **excluded** in the result set.

### Joining Tables With Complete Matches
- The `INNER JOIN` only retrieves matching records from both tables based on the conditions specified in the `ON` Clause. Any non-matching records are left out of the result set. This ensures only _perfectly_ aligned data is included in query results. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders INNER JOIN customers ON orders.customer_id = customers.customer_id;
  ```
  - This query retrieves order ID, order date, and customer name only from **customers who have placed orders**.

### Retrieving Data From Multiple Tables
- You can use `INNER JOIN` to combine data from more than two tables by using additional `JOIN` Functions and `ON` Clauses. For example:
  ```sql
  SELECT order_items.order_id, order_items.product_name, customers.customer_name, shipments.shipment_status FROM order_items INNER JOIN orders ON order_items.order_id = orders.order_id INNER JOIN customers ON orders.customer_id = customers.customer_id LEFT JOIN shipments ON orders.order_id = shipments.order_id;
  ```
  - This query selects order ID, product name, customer name, and shipment status for customers that have placed valid orders (have associated order items). Corresponding shipment information for each order, such as shipment status is included when available (only for orders that have been assigned to a shipment).

### Combining INNER JOIN With Other Joins
- `INNER JOIN` can be combined with other joins, such as `LEFT JOIN` and `RIGHT JOIN`, as in the example above. This can be helpful for obtaining more precise insights from data.

### Handling NULL Values
- There is typically no need to handle null values when working with `INNER JOIN` since it will only include data from matching records. Data quality checks should be performed to ensure the original data does not contain null values.

### Performance Considerations
- Generally speaking, `INNER JOIN` is more performant than other types of joins because it excludes non-matching records. This makes it an ideal choice for queries with strict data alignment requirements.
## FULL OUTER JOIN OR OUTER JOIN
- The SQL `FULL OUTER JOIN` Function is a powerful tool that allows you to merge data from two or more tables that share a common column, including both matching and non-matching records from **both** tables.
  - The `FULL OUTER JOIN` is one of three types (typically the default type) of `OUTER JOIN`. The other two types are `LEFT OUTER JOIN` and `RIGHT OUTER JOIN`.
  - The `LEFT OUTER JOIN` is a type of `OUTER JOIN` that only returns matched and unmatched values from the left table. It is the same thing as `LEFT JOIN`.
  - The `RIGHT OUTER JOIN` is a type of `OUTER JOIN` that only returns matched and unmatched values from the right table. It is the same thing as `RIGHT JOIN`.
- The `FULL OUTER JOIN` is particularly useful when you need to retrieve data from both tables, but also need to ensure records from both tables are included in the result set, regardless of matching.
- The `FULL OUTER JOIN` is commonly used with `SELECT` to gain meaningful insights about data spread across multiple tables.
- The basic syntax of the `FULL OUTER JOIN` is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 FULL OUTER JOIN table2 ON table1.columnX = table2.columnY;
  ```

### Joining Tables With Complete and Incomplete Matches
- The `FULL OUTER JOIN` function can be used to join two or more tables based on complete and incomplete matches. It will fetch data from the left and right table and include it in the result set, regardless of matching based on the conditions in the `ON` Clause. For example:
  ```sql
  SELECT customers.customer_id, customers.customer_name, orders.order_id, orders.order_date FROM customers FULL OUTER JOIN orders ON customers.customer_id = orders.customer_id;
  ```
  - This query will retrieve the customer ID, customer name, order ID and order date for customers who have and haven't placed orders. It will include both matched and unmatched customers and orders. However, **null values will be assigned to non-matching records** (otherwise, the `ON` Clause would be useless).

### Retrieving Data From Multiple Tables
- The `FULL OUTER JOIN` can be used to retrieve data from more than two tables by using additional `JOIN` Functions and `ON` Clauses. For example:
  ```sql
  SELECT order_items.order_id, order_items.product_name, customers.customer_name, shipments.shipment_status FROM order_items FULL OUTER JOIN orders ON order_items.order_id = orders.order_id FULL OUTER JOIN customers ON orders.customer_id = customers.customer_id FULL OUTER JOIN shipments ON orders.order_id = shipments.order_id;
  ```
  - This query will return order ID, product name, customer name, and shipment status for customers who have placed orders with assigned shipments. Null values will be assigned to the associated columns for customers who have not placed orders and shipments without orders.

### Combining FULL OUTER JOIN With Other Joins
- The `FULL OUTER JOIN` can be combined with other types of joins to perform complex data analysis.

### Handling NULL Values
- Since the `FULL OUTER JOIN` includes non-matching records from both tables, with null values assigned for non-matching records, it's imperative to properly handle null values appropriately when performing calculations and aggregations.

### Performance Considerations
- The `FULL OUTER JOIN` is not as performant as the `INNER JOIN` since it includes all columns from both tables. It should be used with caution, especially when dealing with large datasets, as it can negatively impact query execution.
## JOIN WITH WHERE
- The `WHERE` Clause can be used with `JOIN` to merge data based in two or more tables based on values in a shared column as well as filtering logic specified in the `WHERE` Clause. This is particularly useful when you need to filter data derived from merged tables based on certain criteria. This allows you to merge data while also gaining precise insights.
- The basic syntax of `JOIN` with `WHERE` is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 JOIN table2 ON table1.columnX = table2.columnY WHERE condition;
  ```
- The `JOIN` Function combines the data in the two tables based on the condition defined in the `ON` Clause. The `WHERE` Clause further filters the merged data, excluding data that does not satisfy the additional specified criteria. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders JOIN customers ON orders.customer_id = customers.customer_id WHERE orders.order_date >= '2023-01-01';
  ```
  - This query returns order ID, order date, and customer name only for customers who have placed orders, then filters those results to only include orders placed on or after 2023-01-01.
  - The `JOIN` used in this query is equivalent to `INNER JOIN`, which is why only customers who have placed orders are returned by the `JOIN` Functions.
- The `WHERE` Clause can also be applied after joining more than two tables. For example:
  ```sql
  SELECT orders.order_id, order_items.product_name, customers.customer_name FROM orders JOIN order_items ON orders.order_id = order_items.order_id JOIN customers ON orders.customer_id = customers.customer_id WHERE order_items.unit_price > 50;
  ```
  - This query returns order ID, product name, and customer name for customers who have placed valid orders (orders with order items), then filters those results to only include order items with a unit price greater than $50.

### Filtering on Non-Joined Columns
- The `WHERE` Clause can be applied to columns that were not involved in the `JOIN` Operation, allowing you to perform additional filtering as needed. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders JOIN customers ON orders.customer_id = customers.customer_id WHERE orders.order_date >= '2023-01-01' AND customers.country = 'USA';
  ```
  - This query returns order ID, order date,and customer name for customers who have placed orders, then filters those results to only include orders placed by US customers on or after 2023-01-01.

### Combining JOIN With Other Clauses
- The `JOIN` Function can also be combined with other clauses in addition to the `WHERE` Clause, such as `GROUP BY`, `ORDER BY`, and `HAVING` to perform more complex analysis. You can basically use any clause you want to manipulate the data that is returned by a join.
## JOIN WITH COMPARISON OPERATOR
- The `JOIN` Function can also be used with Comparison Operators to combine data from two or more tables based on conditions other than equality.
  - Up to this point, we've only been performing "Equi-Joins", where we perform an equality check in the `ON` Clause. You're allowed to perform other types of checks in the `ON` Clause as well. You can also apply more than one condition using Logical Operators.
- This is particularly useful when you need to merge data based on more complex comparison conditions.
- The basic syntax of `JOIN` with Comparison Operators is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 JOIN table2 ON table1.columnX = table2.columnY AND table1.columnZ > table2.columnW;
  ```
  - Note: This is just a general example. You're not limited to just '=' or '>'.

### Combining Tables With Comparison Operator
- Using the `JOIN` Function with Comparison Operators merges data from two or more tables using conditions based on Comparison Operators such as <, <=, >, >=, =, and <>. The `ON` Clause specifies the columns that should be compared using these operators. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders JOIN customers ON orders.customer_id = customers.customer_id AND orders.order_total > 1000;
  ```
  - This query returns order ID, order date, and customer name customer orders worth more than $1000 (excluding any orders with null customer details or customers with null order details).
  - Note that we still use the equality check to compare a column that is shared between the tables. This is required to effectively merge the data.

### JOIN With Multiple Tables and Comparison Operator
- You can use the `JOIN` Function to merge data across more than two tables and apply Comparison Operators on each join. For example:
  ```sql
  SELECT orders.order_id, order_items.product_name, customers.customer_name FROM orders JOIN order_items ON orders.order_id = order_items.order_id AND order_items.unit_price < 50 JOIN customers ON orders.customer_id = customers.customer_id AND customers.country = 'USA';
  ```
  - This query returns order ID, product name, and customer name for orders with items worth less than $50 that belong to US customers.

### Handling Complex Conditions
- You can perform joins on multiple tables using complex conditions by applying both Conditional and Logical Operators in the `ON` Clause to form these complex conditions. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders JOIN customers ON orders.customer_id = customers.customer_id AND (orders.order_total > 1000 OR customers.country = 'Canada');
  ```
  - This query returns order ID, order date, and customer name either for orders worth more than $1000 or orders from Canadian customers.

### Combining JOIN With Other Clauses
- The `JOIN` Function be combined with Comparison Operators, Conditional and Logical Operators, as well as other clause, such as `GROUP BY`, `ORDER BY`, and `HAVING` to create even more complex join conditions.
## DISTINCT
- In SQL, `DISTINCT` is a powerful tool that allows you to retrieve unique values from a specified column in a table. This is particularly useful when you need to eliminate duplicate records and focus on unique values. `DISTINCT` is commonly used in conjunction with `SELECT` to filter out redundant data and streamline queries.
- The basic syntax for `DISTINCT` is as follows:
  ```sql
  SELECT DISTINCT column1, column2, ... FROM table_name;
  ```
- When applied to one or more column, `DISTINCT` returns unique values from those columns, ensuring duplicate records are eliminated. For example:
  ```sql
  SELECT DISTINCT department FROM employees;
  ```
  - This query returns a unique set of departments from the employees table.
- When applied to multiple columns, `DISTINCT` returns unique _combinations_ of values across those columns. This is particularly useful when dealing with composite keys or multidimensional data. For example:
  ```sql
  SELECT DISTINCT first_name, last_name FROM customers;
  ```
  - This query returns a unique set of customers, including both their first and last name. In other words, a unique set of first-name-last-name pairs.

### Limiting Distinct Results
- You can combine `DISTINCT` with other clauses, such as `ORDER BY` and `WHERE` to obtain results that are even more refined than using just `DISTINCT`. For example:
  ```sql
  SELECT DISTINCT product_name FROM products WHERE price > 1000 ORDER BY product_name;
  ```
  - This query returns a unique set of product names whose price is greater than $1000, ordered in ascending order by product name.

### Handling NULL Values
- Be aware that `DISTINCT` will treat null values as unique. If your data contains null values, they will be included in the query results as separate entities. To properly handle null values, consider using `COALESCE` or `IFNULL` to replace null values with a default value before applying distinct.

### DISTINCT With Aggregate Functions
- `DISTINCT` can also be used with aggregate functions such as `COUNT`, `SUM`, and `AVG` to perform calculations on sets of unique data. For example:
  ```sql
  SELECT COUNT(DISTINCT department) AS total_departments FROM employees;
  ```
  - This query returns the total number of unique departments in the employees table.
  - First, `DISTINCT` returns a set of unique departments, then `COUNT` returns the number of departments in that set.
## JOIN WITH MULTIPLE KEYS
- The SQL `JOIN` Operation can be used with multiple keys, allowing you to combine two or more tables based on multiple columns. This is particularly useful when retrieving data with more complex relationships than a single foreign key. By joining data on multiple keys, you can create more precise connections and gain comprehensive insights from your data.
- The basic syntax of joining on multiple keys is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 JOIN table2 ON table1.columnX = table2.columnY AND table1.columnA = table2.columnB;
  ```
  - To join on multiple keys, we simply separate each key comparison with `AND`.
- Joining on multiple keys combines data from two or more tables based on conditions specified in the `ON` Clause. Matching multiple columns allows you to merge your data more precisely. For example:
  ```sql
  SELECT orders.order_id, orders.order_date, customers.customer_name FROM orders JOIN customers ON orders.customer_id = customers.customer_id AND orders.shipping_address = customers.address;
  ```
  - This query returns order ID, order date, and customer name for customers who have placed orders with matching addresses.
- You can join on multiple keys to connect more than two tables in order to create even more intricate connections among your data. For example:
  ```sql
  SELECT orders.order_id, order_items.product_name, customers.customer_name, shipments.shipment_status FROM orders JOIN order_items ON orders.order_id = order_items.order_id JOIN customers ON orders.customer_id = customers.customer_id AND orders.shipping_address = customers.address LEFT JOIN shipments ON orders.order_id = shipments.order_id;
  ```
  - This query returns order ID, product name, customer name, and shipment status for orders that have matching customer IDs and addresses. If a shipment doesn't exist for a given order, shipment status will be null.
- Using join on multiple keys allows you to handle more complex relationships between data, allowing you to merge data based on multiple attributes.

### Performance Considerations
- While joining on multiple keys allows for more accurate data retrieval, it can also negatively impact query performance, especially when working with large datasets.
- It's essential to optimize your database and use proper indexing to maintain efficient query execution.

### Using Other Clauses When Joining On Multiple Keys
- You can use clauses such as `WHERE`, `GROUP BY`, `HAVING`, and `ORDER BY` when joining on multiple keys to perform more complex analysis.
## SELF JOIN
- The SQL `SELF JOIN` is a powerful tool that allows you to combine data from a table with itself, creating a temporary relationship between rows within the same table. This is particularly useful when working with hierarchical data, such as organizational charts or nested categories. It allows you to compare data within the same table, providing valuable insights about hierarchical relationships.
- The basic syntax of the `SELF JOIN` is as follows:
  ```sql
  SELECT column1, column2, ... FROM table AS t1 JOIN table AS t2 ON t1.columnX = t2.columnY;
  ```
  - To perform the self join, we assign temporary aliases to different instances of the same table to differentiate between the two instances.
  - The `ON` Clause specifies conditions for combining the rows from both instances based on their common columns.

### Application in Hierarchical Data
- One common use case of the `SELF JOIN` is hierarchical data representation. For example, consider an employees table with a manager_id column that represents the employee's manager:
  ```sql
  SELECT e.employee_id, e.employee_name, m.employee_name AS manager_name FROM employees AS e JOIN employees AS m ON e.manager_id = m.employee_id;
  ```
  - This query returns the employee ID, employee name, and manager name for each employee.
  - Here, the employees table contains both employee and manager data (managers are still employees).
  - Self joining the table allows you to access the employee data and corresponding manager data in one query.
  - `employees AS e` creates an instance of the table that will be used to retrieve employee data.
  - `employees AS m` creates an instance of the table that will be used to retrieve manager data.
  - The join isn't executed by comparing the tables row-by-row (comparing the first row of table A to the first row of table B), the join is executed by looking for general matches amongst all rows in the column, then combining two rows for which there is a match.

### Finding Managerial Hierarchies
- As an example, `SELF JOIN` can be used to find employees and their respective managers at different levels of the organizational hierarchy. For example:
  ```sql
  SELECT e.employee_name, m1.employee_name AS manager, m2.employee_name AS top_manager FROM employees AS e JOIN employees AS m1 ON e.manager_id = m1.employee_id JOIN employees AS m2 ON m1.manager_id = m2.employee_id;
  ```
  - This query returns the employee name, manager name, and skip-level manager name of each employee.
  - In the first join, we combine rows by finding a match between an employee's (row's) manager ID and an employee's employee ID. When there is a match, that indicates the first row contains employee info and the second row contains their manager's info.
  - In the second join, the same thing is done as in the first join, except it's being done for an employee's manager. `m2.employee_id` represents their skip-level manager (manager's manager).
  - In the `SELECT`, we retrieve the employee name from the first instance of the table, their manager's name from the second instance, and their skip-level manager's name from the third instance.

### Handling Circular References
- When working hierarchical data using `SELF JOIN`, be cautious of circular references, where an employee is their own manager. Avoid circular references, infinite loops, and unexpected results by using proper conditionals or filters 

### Combining Self JOIN With Other Clauses
- `SELF JOIN` can be combined with other clauses, such as `GROUP BY`, `HAVING`, `ORDER BY`, and `WHERE` to perform more comprehensive analysis on hierarchical data.
## UNION
- The SQL `UNION` Operator is a powerful tool that allows you to combine corresponding **rows** from multiple `SELECT` queries into a single result set. This particularly useful for collecting data from multiple tables and presenting it as a unified output.
- The `UNION` Operator allows you to merge datasets with the same number of columns and compatible data types, providing valuable insights for data analysis and reporting.
  - **`UNION` can only be used when the `SELECT` Statements have the same number of columns _and_ compatible data types.** If data types differ, consider using type conversion functions to ensure compatibility
- The `UNION` Operator is applicable when:
  - Retrieving data from multiple tables that have similar structures.
  - Consolidating info from different conditions into a single result set.
  - Displaying aggregated data from multiple sources.
- The basic syntax of the `UNION` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table1 WHERE condition UNION SELECT column1, column2, ... FROM table2 WHERE condition;
  ```
- The `UNION` Operator will combine data from two or more sets _while also removing duplicate rows_ by default.
  - Duplicates are those rows where _every single column value_ is identical.
  - `UNION ALL` can be used in use cases where duplicate rows need to be included.
- Since the `UNION` Operator does not guarantee any specific ordering of combined rows, use `ORDER BY` after `UNION` to ensure specific ordering is applied.

### Best Practices
- Ensure that the `SELECT` queries in a `UNION` have the same number of columns and compatible data types.
- Use `UNION ALL` when you need to include all rows from both data sets.
- Consider using column aliases to provide meaningful names in the final result set.
- Use `ORDER BY` after `UNION` to apply specific ordering where necessary.

### Practice Examples
- Example 1 - **Consolidating Data from Multiple Conditions:** We have two tables, "orders" and "invoices," with similar structures, and we want to consolidate the order IDs and amounts from both tables for a specific customer.
  ```sql
  SELECT order_id, amount FROM orders WHERE customer_id = 101 UNION SELECT invoice_id AS order_id, total_amount AS amount FROM invoices WHERE customer_id = 101;
  ```
  - This query returns order IDs and amounts for orders where the customer ID is 101. Data is collected and merged from the orders table and invoices table.
- Example 2 - **Adding Duplicates with UNION ALL:** By default, SQL UNION removes duplicate rows from the result set. If you want to include duplicate rows, use the UNION ALL operator.
  ```sql
  SELECT order_id, amount FROM orders UNION ALL SELECT invoice_id AS order_id, total_amount AS amount FROM invoices
  ```
  - This query returns all order IDs and amounts from both the orders and invoices table.
## CURDATE & CURTIME
- Date and Time Functions in SQL allow you to work with dates, times, and timestamps within a database. They are particularly useful for tasks such as filtering data based on a specific time period, calculating time intervals, or formatting date outputs for reports. They help you analyze trends, track activity, and create time-based summaries.
- The most widely used Date and Time Functions in SQL are those that are used to retrieve the current date, current time, and current date and time. These are particularly useful for logging events and running reports.
- The basic syntax varies based on the type of SQL you're using, but the overall concept remains the same. The functions are called without arguments to retrieve the current date/time.
  ```sql
  SELECT CURRENT_DATE;     -- Current date
  SELECT CURRENT_TIME;     -- Current time
  SELECT CURRENT_TIMESTAMP;-- Current date and time
  ```
### Getting Current Date
- MySQL:
  ```sql
  SELECT CURDATE() AS current_date; -- This returns the current date (e.g., 2025-08-09).
  ```
- PostgreSQL:
  ```sql
  SELECT CURRENT_DATE AS current_date;
  ```
- SQL Server:
  ```sql
  SELECT CAST(GETDATE() AS DATE) AS current_date;
  ```

### Getting Current Time
- MySQL:
  ```sql
  SELECT CURTIME() AS current_time; -- This returns the current time in HH:MM:SS format from the UTC timezone by default.
  ```
- PostgreSQL:
  ```sql
  SELECT CURRENT_TIME AS current_time;
  ```
- SQL Server:
  ```sql
  SELECT CAST(GETDATE() AS TIME) AS current_time;
  ```

### Getting Both Date and Time
- MySQL:
  ```sql
  SELECT NOW() AS current_datetime; -- Uses UTC timezone by default.
  ```
- PostgreSQL:
  ```sql
  SELECT CURRENT_TIMESTAMP AS current_datetime;
  ```
- SQL Server:
  ```sql
  SELECT GETDATE() AS current_datetime;
  ```

### Practice Example
- Imagine you have a logins table and want to find all users who logged in today:
  ```sql
  SELECT user_id, login_time
  FROM logins
  WHERE DATE(login_time) = CURDATE();
  ```
  - Here, the `DATE` Function converts the login_time into a date.
## DATE ARITHMETIC FUNCTIONS
- SQL Date Arithmetic Functions allow you to perform calculations on datetime values. They are essential for tasks such as finding the difference between two dates/timestamps, adding or subtracting days, months, or years, and manipulating timestamps to analyze time intervals.

### Basic Syntax and Functions
- `DATEDIFF` - Calculate the difference between two dates:
  - MySQL:
    ```sql
    SELECT DATEDIFF('2025-08-15', '2025-08-09') AS days_diff; -- Returns 6 (Because each date is assumed to be at midnight).
    ```
  - SQL Server:
    ```sql
    SELECT DATEDIFF(day, '2025-08-09', '2025-08-15') AS days_diff; -- Returns 6.
    ```
  - PostgreSQL:
    ```sql
    SELECT '2025-08-15'::date - '2025-08-09'::date AS days_diff; -- Returns 6.
    ```

- `DATE_ADD()` / `DATEADD()` – Add interval (days, months, years, or other intervals) to a date:
  - MySQL:
    ```sql
    SELECT DATE_ADD('2025-08-09', INTERVAL 10 DAY) AS new_date; -- Returns 2025-08-19.
    ```
  - SQL Server:
    ```sql
    SELECT DATEADD(day, 10, '2025-08-09') AS new_date; -- Returns 2025-08-19.
    ```
  - PostgreSQL:
    ```sql
    SELECT '2025-08-09'::date + INTERVAL '10 days' AS new_date; -- Returns 2025-08-19.
    ```
- `DATE_SUB()` / `DATEADD()` with negative values – Subtract interval from a date:
  - MySQL:
  ```sql
  SELECT DATE_SUB('2025-08-09', INTERVAL 5 DAY) AS new_date; -- Returns 2025-08-04.
  ```
  - SQL Server:
  ```sql
  SELECT DATEADD(day, -5, '2025-08-09') AS new_date; -- Returns 2025-08-04.
  ```
  - PostgreSQL:
  ```sql
  SELECT '2025-08-09'::date - INTERVAL '5 days' AS new_date; -- Returns 2025-08-04.
  ```
## DATE FORMAT
- The SQL `DATE_FORMAT` Function allows you to display date and time values in a human-readable manner. Instead of showing dates in the default `YYYY-MM-DD` format, you can format them to match reporting needs, cultural preferences, or business requirements.
- The function is particularly useful when:
  - You want to display a date in the `day/month/year` format.
  - You need to show weekday names, such as Monday or Tuesday.
  - You want to extract parts of a date.
- The basic syntax of the `DATE_FORMAT` Function is as follows:
  ```sql
  DATE_FORMAT(date, format)
  ```
  - date is the date or timestamp value you want to format.
  - format is the string containing specifiers for how the date should be displayed.
- Common Format Specifiers:
  ```sql
  | Specifier | Meaning                  | Example Output |
  | --------- | ------------------------ | -------------- |
  | `%Y`      | 4-digit year             | 2025           |
  | `%y`      | 2-digit year             | 25             |
  | `%M`      | Full month name          | August         |
  | `%m`      | Month number (01–12)     | 08             |
  | `%d`      | Day of month (01–31)     | 11             |
  | `%b`      | Abbreviated month name   | Aug            |
  | `%W`      | Full weekday name        | Monday         |
  | `%a`      | Abbreviated weekday name | Mon            |
  | `%H`      | Hour (00–23)             | 14             |
  | `%h`      | Hour (01–12)             | 02             |
  | `%i`      | Minutes                  | 05             |
  | `%s`      | Seconds                  | 09             |
  | `%p`      | AM/PM                    | PM             |
  ```
- Example:
  ```sql
  SELECT
      order_id,
      order_date,
      DATE_FORMAT(order_date, '%M %d, %Y') AS formatted_date,
      DATE_FORMAT(order_date, '%W at %h:%i %p') AS friendly_time
  FROM orders;
  ```
  - Selects the order ID and order date, then formats the order date as both 'Month dd, yyyy' and 'Weekday at hh:mm AM/PM'.
