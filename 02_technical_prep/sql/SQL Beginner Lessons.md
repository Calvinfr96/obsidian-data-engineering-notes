---
aliases:
  - "sql_beginner_lessons"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Notes/sql_beginner_lessons.md"
imported: 2026-09-24
---

# SQL Beginner Lessons

Review: [[02_technical_prep/sql/SQL Optimization Principles|SQL Optimization Principles]] · [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#SELECT FROM]]
- [[#WHERE Condition]]
- [[#COMPARISON OPERATORS]]
- [[#LOGICAL OPERATORS]]
- [[#LIKE OPERATOR]]
- [[#IN OPERATOR]]
- [[#BETWEEN OPERATOR]]
- [[#IS NULL OPERATOR]]
- [[#AND OPERATOR]]
- [[#OR OPERATOR]]
- [[#NOT Operator]]
- [[#ORDER BY]]
- [[#LIMIT OFFSET]]

## SELECT FROM
- `SELECT FROM` is the main statement that is used for querying database tables in SQL. The general syntax for the statement is `SELECT {columns} FROM {table};`:
  - Example `SELECT FROM` Statement (selecting specific columns):
    ```sql
    SELECT
        column1,
        column2,
        column3
    FROM table_name;
    ```
    - When specifying columns to display, they are displayed _in the specified order,_ not in the order defined in the table schema.
    - Selecting only the necessary columns can optimize data retrieval, reduce network traffic, and improve query performance.
  - Example `SELECT FROM` Statement (selecting specific columns and specifying column aliases):
    ```sql
    SELECT
        column1 AS alias1,
        column2 AS alias2,
        column3 AS alias3
    FROM table_name;
    ```
    - The `AS` Keyword is used to assign **temporary** names to columns in the result set. Aliases are typically assigned to columns to improve readability.
    - When specifying columns in this manner, the will be displayed in the specified order _with their aliases_, not the name specified in the table schema.
    - Column aliases can be specified with or without quotes. They are typically specified **without** quotes (assuming they're single-word aliases).
  - Example `SELECT FROM` Statement (selecting all columns):
    ```sql
    SELECT * FROM table_name;
    ```
    - Keep in mind that while retrieving all columns might be useful in some scenarios, it can lead to inefficiencies when dealing with large tables or unnecessary data.

### Practice Examples
- Example 1 - Simple `SELECT`:
  ```sql
  SELECT * FROM employees;
  ```
  - Selects all columns from the employees table, in the order specified in the table schema.
- Example 2 - Selecting Specific Columns:
  ```sql
  SELECT
    employee_id,
    first_name,
    last_name
  FROM employees;
  ```
  - Selects the employee ID, first name, and last name of each employee in the employees table.
- Example 3 - Aliasing Columns:
  ```sql
  SELECT
    product_name AS "Product",
    unit_price AS "Price"
  FROM products;
  ```
  OR
  ```sql
  SELECT
    product_name AS Product,
    unit_price AS Price
  FROM products;
  ```
  - Selects the product name (displayed as "Product") and unit price of each product in the products table.
## WHERE Condition
- The `WHERE` Clause is used to filter data based on specific conditions in an SQL query. Combining the `WHERE` Clause with `SELECT FROM` allows you to perform precise and targeted retrieval of data (rows) within a database table based on certain conditions.
- The basic syntax of using a `WHERE` Clause within a `SELECT FROM` Statement is as follows:
  ```sql
  SELECT
    column1,
    column2,
    column3
  FROM table_name
  WHERE condition;
  ```
  - The `WHERE` Clause filters data by applying the specified conditions to the rows within a table. It allows you to specify one or more conditions that must true for the row to be included in the result set.
  - The order of keywords is important here. `SELECT` always comes first, then `FROM`, then `WHERE`.
- Both Comparison and Logical Operators can be used within the condition of a `WHERE` clause.
- Logical Operators include `AND`, `OR`, and `NOT`. They are used to create more complex conditions within a `WHERE` clause. For example:
  ```sql
  SELECT * FROM employees WHERE salary > 50000 AND department = 'Sales';
  ```
  - Instead of selecting all rows from the employees table, this query selects specific employees (rows). Namely, those with a salary greater than 50000 who are work in the Sales department.
- Comparison Operators (`=`, `<>`, `<`, `>`, `<=`, `>=`) are used to compare values within the condition of a `WHERE` Clause:
  ```sql
  SELECT * FROM employees WHERE salary > 50000;
  ```
  - This query retrieves employees whose salary is greater than 50000.
  - The `<>` operator is functionally equivalent to `!=`. However, `<>` follows the ISO standard for SQL and **is the generally recommended syntax for better compatibility** across different database systems.
- The `WHERE` Clause can also be used with Wildcards (`%`) for pattern matching. For example, the following query will retrieve all rows (customers) whose customer_name starts with "Joh":
  ```sql
  SELECT * FROM customers WHERE customer_name LIKE 'Joh%';
  ```
- Logical expressions can be grouped with parentheses to create more complex conditions. For example, the following query will retrieve products with a unit price less than 50 or a category of 'Electronics':
  ```sql
  SELECT
    product_name,
    unit_price,
    category
  FROM products
  WHERE (unit_price < 50 OR category = 'Electronics');
  ```
  - Note that the parentheses aren't required in this query, but would help if you wanted to attach additional conditions, such as `AND product_name = 'Macbook'`. This would return Macbooks whose unit price is less than 50 or whose category is electronics.
- To filter rows based on `NULL` values, the `IS NULL` or `IS NOT NULL` Operators can be used. For example:
  ```sql
  SELECT * FROM employees WHERE manager_id IS NULL;
  ```
  - This query returns all columns for employees who have not been assigned a manager.
  - In SQL, `NULL` means the column in a specific row **has not been assigned a value**, which is why you can't use the `=` or `<>` operators to compare with `NULL`.
## COMPARISON OPERATORS
- SQL Comparison Operators are essential tools used for making data comparisons in SQL queries. They enable you to evaluate conditions and filter data based on specific comparisons between values.
- Comparison Operators are used in the `WHERE` Clause to compare values. The basic syntax is as follows:
    ```sql
    SELECT column1, column2, ... FROM table_name WHERE column_name operator value;
    ```
    For example:
    ```sql
    SELECT
      product_id,
      product_name,
      category,
      price
    FROM products
    WHERE category = 'Electronics';
    ```
- Commonly used Comparison Operators include: `=`, `!=`, `<>`, `>`, `<`, `>=`, and `<=`.
- Comparison Operators can be combined with the Logical Operators `AND`, `OR`, or `NOT` to create more complex queries for more targeted data retrieval. For example:
  ```sql
  SELECT * FROM orders WHERE order_amount > 1000 AND order_status = 'Pending';
  ```
  - This query will return every detail or all pending orders worth more than $1000.
## LOGICAL OPERATORS
- Logical Operators are powerful components in SQL queries which allow you to apply several conditions to create powerful and precise data retrieval statements.
- By using the Logical Operators (`AND`, `OR`, `NOT`), you can perform complex filtering and make more complex comparisons between data elements.
- Logical Operators are most commonly used in the `WHERE` clause to combine several conditions. The basic syntax is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE condition1 {logical_operator} condition2;
  ```
  For example:
  ```sql
  SELECT * FROM employees WHERE department = 'Sales' AND salary > 50000;
  ```
  - `AND`: Data is only returned when both conditions are met.
  - `OR`: Data is only returned when either (at least one) condition is met.
  - `NOT`: Negates the condition and only returns data if it evaluates to false.
- Logical Operators can be combined to create more complex conditions in the `WHERE` clause. For example:
  ```sql
  SELECT * FROM orders WHERE (order_amount > 1000 AND order_status = 'Pending') OR (order_status = 'Processing');
  ```
  - This query will only return orders that are still processing (worth any amount) or pending orders worth more than $1000.
  - When combining multiple logical operators, it is important to consider operator precedence. The order of evaluation is typically: `NOT`, `AND`, `OR`. However, parentheses can be used to control the order of evaluation explicitly.
  - Since order precedence is not guaranteed across database systems, it's probably best practice to use parentheses to ensure the desired order is followed.
## LIKE OPERATOR
- The SQL `LIKE` Operator is a powerful tool that allows you to perform pattern matching on text data in SQL. It enables you to retrieve data based on specific patterns or substrings that are present in a column's values.
- Whether you need to search for specific words, find similar names, or filter data based on partial information, the `LIKE` Operator is an essential tool to master.
- The basic syntax of the `LIKE` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE column_name LIKE pattern;
  ```
- The true power of the `LIKE` Operator comes from its ability to use wildcard characters for pattern matching. The two most commonly used wildcards are `%` and `_`. For example:
  ```sql
  SELECT * FROM customers WHERE customer_name LIKE 'Joh%';
  ```
  - This query will retrieve all customers with names starting with "Joh," followed by any number of characters.
  ```sql
  SELECT * FROM employees WHERE last_name LIKE 'Sm_th';
  ```
  - This query will retrieve employees with last names starting with "Sm", followed by any single character, followed by "th".
  - You can use multiple `_` in one comparison, where each `_` represents one (any) character. For example "Smith" would also match the pattern `Sm__h` or `S_i_h`.
- **You do not need to use `*` when using wildcards to make comparisons. You can select specific columns within the table.**

### Practice Examples
- Example 1 -  **Finding Specific Words:** To find all products with names containing the word "phone," you can use the `LIKE` operator as follows:
  ```sql
  SELECT * FROM products WHERE product_name LIKE '%phone%';
  ```
- Example 2 - **Similar Names:** To find customers with similar names, such as "Smith" and "Smyth," you can use the LIKE Operator with the underscore wildcard:
  ```sql
  SELECT * FROM customers WHERE last_name LIKE 'Sm_th';
  ```
  - Note that the `_` only represents one character while the `%` represents any number of characters.
- Example 3 - **Combining LIKE with Other Operators:** The `LIKE` Operator can be combined with other operators and Logical Operators to create more complex conditions:
  ```sql
  SELECT * FROM products WHERE product_name LIKE 'A%' OR (unit_price > 100 AND product_name LIKE '%phone%');
  ```
  - This query will retrieve products with names starting with 'A' or products with unit prices greater than 100 and names containing the word "phone."
## IN OPERATOR
- The SQL `IN` Operator simplifies data filtering in queries when dealing with multiple values. It allows you to specify a list of values and the query will return rows that match any of the values in the list.
- The `IN` Operator is particularly useful when you need to filter data based on multiple options or want to avoid using multiple `OR` conditions.
- The basic syntax of the `IN` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE column_name IN (value1, value2, value3, ...);
  ```
  - This avoids using multiple `OR` statements to perform the same filtering:
  ```sql
  SELECT
    column1,
    column2,
    ...
  FROM table_name
  WHERE column_name = value1 OR
  column_name = value3 OR
  column_name = value3 OR
  ...;
  ```
- The `IN` Operator checks if the specified column contains any rows with values that match those in the specified list. For example:
  ```sql
  SELECT * FROM products WHERE category IN ('Electronics', 'Appliances', 'Office Supplies');
  ```
  - Selects all columns for products whose category is 'Elections', 'Appliances', or 'Office Supplies'.

### Combining the IN Operator
- The `IN` Operator can be combined with other Logical Operators to form complex queries that can be used for more targeted data retrieval. For example:
  ```sql
  SELECT * FROM products WHERE (unit_price > 100 AND category IN ('Electronics', 'Appliances')) OR (category = 'Office Supplies');
  ```
  - Retrieves all columns for products that cost more than $100 and belong to the 'Electronics' or 'Appliances' category, or products that belong to the 'Office Supplies' category.
  - Note the usage of parentheses to group conditions for precise querying.

### Practice Examples
- Example 1 - **Filtering Data with Multiple Options:** Suppose you have a table of employees and you want to retrieve employees from specific departments. Instead of using multiple `OR` conditions, you can use the `IN` Operator as follows:
  ```sql
  SELECT * FROM employees WHERE department IN ('Sales', 'Marketing', 'Customer Support');
  ```
  - Selects employees in the 'Sales', 'Marketing', or 'Customer Support' department.
- Example 2 - **Using Subqueries with IN:** The `IN` Operator can also be used with subqueries to retrieve data based on values from another table. For example:
  ```sql
  SELECT * FROM orders WHERE customer_id IN (SELECT customer_id FROM customers WHERE country = 'USA');
  ```
  - Selects orders from US customers. The subquery prevents having to list out all of the customer IDs for US customers.
## BETWEEN OPERATOR
- The SQL `BETWEEN` Operator simplifies data filtering in queries that deal with range-based conditions. It allows you to specify a range of values and the query will retrieve rows that fall within that range. This is particularly useful when filtering data based on numeric or date ranges.
- The basic syntax of the `BETWEEN` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE column_name BETWEEN value1 AND value2;
  ```
- The `BETWEEN` Operator will check if the values in the specified column falls within the specified range. For example:
  ```sql
  SELECT * FROM products WHERE unit_price BETWEEN 50 AND 100;
  ```
  - Retrieves products that cost between $50 and $100.
- The `NOT BETWEEN` Operator will check if the values in the specified column fall _outside_ the specified range. For example:
  ```sql
  SELECT * FROM employees WHERE salary NOT BETWEEN 40000 AND 60000;
  ```
  - Retrieves employees whose salary is less than $40000 or greater than $60000.

### Combining the BETWEEN Operator
- The `BETWEEN` Operator can be combined with other Logical Operators to form complex queries that can be used for more targeted data retrieval. For example:
  ```sql
  SELECT * FROM products WHERE (unit_price BETWEEN 50 AND 100) AND (category = 'Electronics');
  ```
  - Retrieves products whose category is 'Electronics' and unit price falls between $50 and $100.

### Important Caveats
- The SQL `BETWEEN` Operator in **inclusive**, meaning that columns containing values equal to the beginning and ending values of the range will be included in the result set. It is functionally equivalent to `WHERE column_name >= value1 AND column_name <= value2`.
- However, when dealing with time-sensitive data, such as `datetime` or `timestamp` data types, you must be careful if you only specify date literals.
  - For example, if you write '1996-07-31', the database often interprets this as '1996-07-31 00:00:00' (midnight at the beginning of the day). This means any records from later in the day on July 31st (e.g., '1996-07-31 10:30:00') would be excluded from your results, which is often not the intended behavior.
  - For full-day date range queries with time-sensitive data, it is recommended to **use explicit comparison operators** (`>=` and `<`) to specify an open-ended upper boundary:
    ```sql
    SELECT * FROM Orders
    WHERE OrderDate >= '1996-07-01' AND OrderDate < '1996-08-01';
    ```

### Practice Examples
- Example 1 - **Filtering Data with Numeric Ranges:** Suppose you have a table of employees, and you want to retrieve employees with salaries between $40,000 and $60,000. You can use the `BETWEEN` Operator as follows:
  ```sql
  SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;
  ```
- Example 2 - **Using BETWEEN with Dates:** The `BETWEEN` Operator is also helpful for filtering data based on date ranges. For example:
  ```sql
  SELECT * FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-03-31';
  ```
## IS NULL OPERATOR
- The SQL `IS NULL` Operator is an important tool used to identify null or missing values in SQL queries. In databases, null represents the absence of data or an unknown value. The `IS NULL` Operator allows you to check if a column contains null values, helping you properly handle missing data and perform data quality checks.
- The basic syntax of the  `IS NULL` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE column_name IS NULL;
  ```
- The `IS NULL` Operator checks if a column contains any missing or unknown values. For example:
  ```sql
  SELECT * FROM customers WHERE email IS NULL;
  ```
  - This query will return all details for customers that are missing an email address.
- The `IS NOT NULL` Operator checks if a column does not contain any missing or unknown values. For example:
  ```sql
  SELECT * FROM employees WHERE manager_id IS NOT NULL;
  ```
  - This query will return all details for employees assigned to a manager (containing a manager ID).

### Combining the IS NULL Operator
- The `IS NULL` Operator can be combined with other Logical Operators to form more complex queries for more targeted data retrieval. For example:
  ```sql
  SELECT * FROM orders WHERE (order_amount > 1000 AND customer_id IS NULL) OR order_status = 'Pending';
  ```
  - This query will return all details for orders whose order amount is greater than $1000 and are missing a customer ID, or orders whose order status is pending.

### Practice Examples
- Example 1 - **Identifying Missing Information:** Suppose you have a table of products, and you want to find products without a description. You can use the `IS NULL` Operator as follows:
  ```sql
  SELECT * FROM products WHERE description IS NULL;
  ```
- Example 2 - **Data Quality Check:** The `IS NULL` Operator is also useful for performing data quality checks. For instance, if you have a table of employees and you want to ensure that all employees have an assigned manager, you can use the following query:
  ```sql
  SELECT * FROM employees WHERE manager_id IS NULL;
  ```
## AND OPERATOR
- The SQL `AND` Operator is a powerful tool used to combine multiple conditions in queries to perform more precise and targeted data retrieval. It is typically used in the `WHERE` clause to specify that both or all (in the case of multiple `AND` statements) conditions must be true for a value to be included in the result set.
- The basic syntax of the `AND` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE condition1 AND condition2 AND condition3 ...;
  ```
- The `AND` Operator is used to combine multiple conditions. All conditions connected by the `AND` Operator must be true for a row to be included in the result set. For example:
  ```sql
  SELECT * FROM employees WHERE department = 'Sales' AND salary > 50000;
  ```
  - Only employees earning more than $50000 in the Sales department will be included in the result set.
- When combining multiple Logical Operators like `AND` and `OR`, it's essential to consider operator precedence. Parentheses can and should be used to control the order of evaluation explicitly.

### Combining the AND Operator
- The `AND` Operator can be combined with other Logical Operators to create even more complex conditions. For example:
  ```sql
  SELECT * FROM customers WHERE (age >= 18 AND age <= 30) AND (city = 'New York' OR city = 'Los Angeles');
  ```
  - This query returns all details for customers from New York or Los Angeles who are aged between 18 and 30 (inclusive).
  - The first condition could be replaced with `age BETWEEN 18 AND 30`, but the way it's written here is more precise.

### Practice Examples
- Example 1 - **Combining Multiple Conditions:** Suppose you have a table of products, and you want to retrieve products from the 'Electronics' category with a unit price greater than 100. You can use the `AND` Operator as follows:
  ```sql
  SELECT * FROM products WHERE category = 'Electronics' AND unit_price > 100;
  ```
- Example 2 - **Complex Conditions:** The `AND` Operator can be used to create complex conditions involving multiple attributes. For example:
  ```sql
  SELECT * FROM orders WHERE (order_date >= '2023-01-01' AND order_date <= '2023-03-31') AND order_amount > 500;
  ```
  - This query returns orders worth more than $500 that were placed between January 1 and March 31, 2023 (inclusive).
## OR OPERATOR
- The SQL `OR` Operator is an important tool that allows you to combine multiple conditions in SQL queries for more _flexible_ data retrieval. It is typically used in the `WHERE` Clause to specify that **at least one** of the conditions must be true for a row to be included in the result set.
- The `OR` Operator plays a crucial role in broadening data filtering, providing alternative options and handling various scenarios.
- The basic syntax of the `OR` Operator is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name WHERE condition1 OR condition2 OR condition3 ...;
  ```
- The `OR` Operator is used to combine multiple conditions, specifying that at least one of the connected conditions must be true for a row to be included in the result set. For example:
  ```sql
  SELECT * FROM employees WHERE department = 'Sales' OR salary > 50000;
  ```
  - This query returns employees in the Sales department or those earning more than $50000.
- When combining multiple Logical Operators like `AND` and `OR`, it's essential to consider operator precedence. Parentheses can and should be used to control the order of evaluation explicitly.
  - Recall that the typical order of evaluation is  `NOT`, `AND`, then `OR`.

### Combining the OR Operator
- The `OR` Operator can be combined with other Logical Operators to create more complex conditions. For example:
  ```sql
  SELECT * FROM orders WHERE (order_amount > 1000 OR order_status = 'Pending') AND customer_id IS NOT NULL;
  ```
  - This query returns orders that are worth more than $1000 or are pending, and are assigned to a customer.

### Practice Examples
- Example 1 - **Combining Multiple Conditions:** Suppose you have a table of products, and you want to retrieve products either from the 'Electronics' category or with a unit price greater than 1000. You can use the `OR` Operator as follows:
  ```sql
  SELECT * FROM products WHERE category = 'Electronics' OR unit_price > 1000;
  ```
- Example 2 - **Handling Different Scenarios:** The `OR` Operator can be used to handle various scenarios. For instance, if you have a table of customers, and you want to retrieve customers from either 'New York' or 'Los Angeles,' you can use the following query:
  ```sql
  SELECT * FROM customers WHERE city = 'New York' OR city = 'Los Angeles';
  ```
## NOT Operator
- The SQL `NOT` Operator is an important tool that allows you to negate a condition in an SQL query. It's typically used in combination with other Comparison and Logical Operators in the `WHERE` Clause to retrieve rows that do not meet a certain condition.
  - The `NOT` Operator is used to exclude certain records from a result set, which is important for fine-tuning data retrieval.
- The basic syntax of the `NOT` Operator is as follows:
  ```sql
  SELECT * FROM employees WHERE NOT department = 'Sales'
  ```
  - This selects employees that do not belong to the Sales department and is functionally equivalent to `department != 'Sales'`.

### Practice Examples
- Example 1 - **Negating Equality:** Suppose you have a table of products, and you want to retrieve products that are not from the 'Electronics' category. You can use the `NOT` operator as follows:
  ```sql
  SELECT * FROM products WHERE NOT category = 'Electronics';
  ```
  Functionally equivalent to:
  ```sql
  SELECT * FROM products WHERE category != 'Electronics';
  ```
- Example 2 - **Combining NOT with Other Operators:** The `NOT` operator can be combined with other Comparison and Logical Operators to create more complex conditions. For instance:
  ```sql
  SELECT * FROM orders WHERE NOT (order_amount > 1000 OR order_status = 'Pending');
  ```
  - This query returns all details for orders that are not worth more than $1000 or are not Pending.
## ORDER BY
- The SQL `ORDER BY` Clause is an essential tool that allows you to sort the result set of an SQL query in a specified order. It is used in conjunction with the `SELECT` statement to arrange the rows of a result set according to a one or more columns' values. Sorting data is essential to presenting it in a meaningful and organized manner.
- The basic syntax of the `ORDER BY` Clause is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name ORDER BY column1 [ASC|DESC], column2 [ASC|DESC], ...;
  ```
  - `ORDER BY` is the last clause specified in the query, whereby the results are sorted in a certain manner.
- The `ORDER BY` Clause can sort data in either ascending (`ASC`) or descending (`DEC`) order. The default sorting order is `ASC` if one is not explicitly specified. For example:
  ```sql
  SELECT first_name, last_name FROM employees ORDER BY first_name ASC, last_name DESC;
  ```
  - This query will sort employees in ascending order according to their first name and descending order according to their last name.
- When using `ORDER BY` to sort by multiple columns, as in the example above, data is sorted by the first column specified. Then, if two or more rows have the same value in that column, the secondary column is used to "break the tie". The same methodology applies to all subsequent columns specified in the `ORDER BY` Clause. For example:
  ```sql
  SELECT product_name, unit_price, category FROM products ORDER BY category ASC, unit_price DESC;
  ```
  - This query orders products according to their category, in ascending order. Then, if two products share the same category, they are ordered according to their unit price, in descending order.

### Sorting with Numeric and Text Columns
- When sorting numeric columns, rows are sorted based on their numeric magnitude.
- When sorting text columns, rows are sorted based on the character sequence following the collation (sorting) rules of the database. For example:
  ```sql
  SELECT product_name, unit_price FROM products ORDER BY unit_price DESC;
  ```
  - This query sorts products according to their unit price in descending (numerical) order, assuming the `unit_price` column is of a numeric data type.

### Sorting with Dates
- Date columns or sorted based on the chronological order of the dates. For example:
  ```sql
  SELECT customer_name, order_date FROM orders ORDER BY order_date ASC;
  ```
  - This query will word orders according to their order date in ascending, chronological order.

### Limiting Sorting Results
- You can combine `ORDER BY` with the `LIMIT` Clause to restrict the number of sorted rows returned in the result set. For example:
  ```sql
  SELECT product_name, unit_price FROM products ORDER BY unit_price DESC LIMIT 10;
  ```
  - This query returns 10 products sorted in descending order according to their unit price. In other words, it will return the 10 most expensive products, sorted in descending order.
## LIMIT OFFSET
- The SQL `LIMIT` Clause is a powerful way to limit the size of a result set returned by an SQL query. It's particularly useful when dealing with large datasets, as it allows you to retrieve a specified number of rows from the table, starting from the beginning of the result set.
- The basic syntax of the `LIMIT` Clause is as follows:
  ```sql
  SELECT column1, column2, ... FROM table_name LIMIT number_of_rows;
  ```
- The `LIMIT` Clause allows you to specify the maximum number of rows that can be returned by the query. For example:
  ```sql
  SELECT * FROM employees LIMIT 10;
  ```
  - This query retrieves the first 10 (assuming there are more than 10 rows) employees from the employees table.

### OFFSET with LIMIT
- The `OFFSET` Clause can be used in conjunction with the `LIMIT` Clause to skip a certain number of rows **in the result set** and retrieve the subsequent rows. For example:
  ```sql
  SELECT * FROM customers LIMIT 5 OFFSET 10;
  ```
  - The result set is generated based on `SELECT * FROM customers`. The `LIMIT` and `OFFSET` result in the first 5 customers being returned, starting from the 11th customer in the result set.
  - `OFFSET` should **always** be used with `ORDER BY` for consistent results. Without it, runs of the same query may skip different rows.

### Compatibility with Different SQL Databases
- Although the `LIMIT` Clause is widely supported in many SQL database systems (MySQL, PostgreSQL, SQLite), it's important to note that some of these systems use different syntax for limiting results. For example `TOP` is used in Microsoft SQL and `FETCH FIRST` is used in IBM Db2. Always reference the documentation of the system you're using to ensure compatibility.

### Practice Examples
- Example 1 - **Limiting Result Set:** Suppose you have a table called "products" with a large number of records, and you only want to see the first 5 products in the result set. You can use the LIMIT clause as follows:
  ```sql
  SELECT * FROM products LIMIT 5;
  ```
- Example 2 - **Combined with `ORDER BY`:** Often, you may want to retrieve a limited number of rows based on a specific ordering criterion. In this case, you can combine the LIMIT clause with the ORDER BY clause. For instance:
  ```sql
  SELECT product_name, unit_price FROM products ORDER BY unit_price DESC LIMIT 3;
  ```
  - This query returns the first three products ordered by unit price in descending order (the 3 most expensive items).
