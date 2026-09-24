---
aliases:
  - "sql_advanced_lessons"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Notes/sql_advanced_lessons.md"
imported: 2026-09-24
---

# SQL Advanced Lessons

Review: [[02_technical_prep/sql/SQL Optimization Principles|SQL Optimization Principles]] · [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]]

## Contents

- [[#DATA TYPES]]
- [[#CONCAT]]
- [[#CAST]]
- [[#LENGTH]]
- [[#SUBSTRING]]
- [[#CHARINDEX OR SUBSTRING_INDEX]]
- [[#TRIM]]
- [[#LEFT & RIGHT]]
- [[#UPPER & LOWER]]
- [[#EXTRACT]]
- [[#COALESCE]]
- [[#SUBQUERY IN CONDITION]]
- [[#WITH STATEMENTS (CTEs)]]
- [[#WINDOW FUNCTIONS]]
- [[#WINDOW FUNCTION WITH AGGREGATE FUNCTION]]
- [[#ROW NUMBER]]
- [[#RANK]]
- [[#LEAD]]
- [[#LAG]]

## DATA TYPES
- SQL Data Types play a crucial role in determining how data is stored, manipulated, and retrieved in a database. Each data type serves a specific purpose, ensuring data integrity, optimizing storage, and facilitating query processing.
- SQL Data Types are broadly classified into the following categories:
  - Numeric Types: Integer and Floating Point Numbers (decimals).
  - Character Strings: Variable and Fixed-Length Strings.
  - Date and Time: Dates, Times, and Intervals.
  - Booleans: True or False Values.
  - Binary Data: Storage for binary objects, such as images or files.
  - Miscellaneous: Data Types such as UUID, JSON, XML, etc.
- Common Number Types:
  - INT: Whole numbers.
  - DECIMAL(p, s): Exact number type with defined precision and scale.
    - Ideal for financial and monetary data, **where minor variations are unacceptable**.
    - `DECIMAL(10, 2)` means 10 total digits with 2 after the decimal point.
    - Represented in base 10.
    - No inherent rounding and comparison errors.
    - Can lead to slower performance.
  - FLOAT: Approximate number type for floating-point numbers.
    - Ideal for scientific or measurement-based calculations, **where minor variations are acceptable**.
    - Provides a range of values with a certain number of significant figures. **Cannot represent decimal values exactly**.
    - Represented in base 2.
    - Prone to minor rounding and comparison errors.
    - Provides better performance when compared to decimal.
- Common String Types:
  - `CHAR(n)`: Fixed-length character string with defined length.
  - `VARCHAR(n)`: Variable-length character string with maximum length.
- Common Date and Time Types:
  - `DATE`: Stores dates (year, month, day). Example: `DATE` represents 'YYYY-MM-DD'.
  - `TIME`: Represents time of day. Example: `TIME` or `TIME(0)` represents 'hh:mm:ss' and `TIME(3)` represents 'hh:mm:ss.nnn'.
  - `TIMESTAMP`: Stores date and time. Example: `TIMESTAMP` represents 'YYYY-MM-DD HH:MM:SS'.
- Boolean Data Type:
  - `BOOLEAN`: Represents true or false values.
- Binary Data Type:
  - `BLOB`: Stores binary large objects, such as images and documents.
- Considerations:
  - Choose data types based on the nature of data and storage requirements.
  - Be mindful of memory consumption and storage efficiency.
  - Maintain data consistency by selecting appropriate data types.
- Advanced Data Types:
  - Data Types such as JSON, XML, and arrays are suitable for more complex data storage needs.
## CONCAT
- The SQL `CONCAT` Function is a powerful tool used to combine two or more strings into a single string. This function is useful for generating composite strings, enhancing data representation, and enriching data manipulation capabilities.
- The basic syntax of the `CONCAT` Function is as follows:
  ```sql
  CONCAT(string1, string2, ...);
  ```
- Common Use Cases:
  - Data Representation: Creating informative and well-structured text.
  - Displaying Full Names: Combining first, middle, and last names into one string.
  - Generating Custom Messages: Constructing personalized messages.

### Best Practices
- Use `CONCAT` for creating informative and personalized text.
- Test `CONCAT` with various string combinations to achieve the desired output.
- Consider null handling to ensure accurate results.

### Handling NULL Values
- `CONCAT` treats null values as empty strings. To handle null values while using `CONCAT`, use `ISNULL` or `COALESCE`.

### Practice Examples
- Example 1 - Concatenating Names:
  ```sql
  SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM students;
  ```
  - Combines the first and last names of each student in the students table to generate a full_name column.
  - Uses a string representing a space to separate the two values in the composite string.
- Example 2 - Creating Custom Messages:
  ```sql
  SELECT CONCAT('Hello, ', first_name, '! Your order #', order_id, ' is confirmed.') AS confirmation_message FROM orders;
  ```
  - Creates a custom confirmation message using a customer's first name and order ID from the orders table.
- Example 3 - Displaying Address:
  ```sql
  SELECT CONCAT(street_address, ', ', city, ', ', state, ' ', postal_code) AS full_address FROM customers;
  ```
  - Creates a composite full_address string using a customer's street address, city, state, and postal code from the customers table.
## CAST
- The SQL `CAST` Function allows you to convert one data type to another, ensuring compatibility for operations that involve mixed data types. This is particularly useful for performing calculations, comparisons, or aggregations involving data of different types.
- The basic syntax of the `CAST` Function is as follows:
  ```sql
  SELECT CAST(expression AS target_data_type) AS alias_name;
  ```
- Common Use Cases:
  - Data Type Conversion: Converting data types to perform accurate calculations.
  - Data Presentation: Formatting data for more meaningful presentation.
  - Comparison Operations: Facilitating comparisons between different data types.
  - Aggregations: Ensuring consistency in aggregated results.

### Implicit vs. Explicit Conversion
- **Implicit Conversion:** Some database systems perform automatic type conversions. However performing explicit type conversions using `CAST` provides better control and clarity.
- **Explicit Conversion:** Using `CAST` for intentional data type conversion reduces ambiguity and ensures desired outcomes.
- **Handling Conversion Errors:** `CAST` may fail when attempting to convert one data type to an incompatible data type. Consider using `TRY_CAST` or exception handling to manage potential errors.

### Best Practices
- Specify the target data type explicitly for clarity and control.
- Be cautious when performing type conversions as it may impact query performance.
- Test conversions and validate results to ensure accuracy.

### Practice Examples
- Example 1 - Data Type Conversion:
  ```sql
  SELECT CAST('123.45' AS DECIMAL(5, 2)) AS converted_value;
  ```
  - Casts a string to a decimal that uses the same precision and scale.
- Example 2 - Comparison of Different Data Types:
  ```sql
  SELECT product_name, unit_price FROM products WHERE CAST(unit_price AS DECIMAL(8, 2)) > 50.00;
  ```
  - Casts unit price, which might be represented as a string, to a decimal for comparison purposes.
- Example 3 - Presenting Data with CAST:
  ```sql
  SELECT order_id, CAST(order_date AS DATE) AS formatted_date FROM orders;
  ```
## LENGTH
- The SQL `LENGTH` Function allows you to determine the length (number of characters) of a given string. This is especially useful for assessing the size of text data and making informed decisions based on string length.
  - Spaces are included in the returned character count.
  - Depending on database and character encoding, `LENGTH` might behave differently when used with multibyte characters.
- The basic syntax of the `LENGTH` Function is as follows:
  ```sql
  LENGTH(string_expression);
  ```
- Common Use Cases:
  - Data Validation: Ensuring text fields meet length requirements.
  - Data Analysis: Assessing the length of textual data.
  - Formatting: Identifying overlong content for formatting adjustments.

### Best Practices
- Utilize `LENGTH` to validate data length effectively.
- Combine `LENGTH` with other functions for sophisticated data analysis.
- Consider the impact of multibyte characters on the result.

### Practice Examples
- Example 1 - Data Validation:
  ```sql
  SELECT product_name FROM products WHERE LENGTH(product_description) <= 100;
  ```
  - Selects product names for products with a product description of at most 100 characters.
- Example 2 - Data Analysis:
  ```sql
  SELECT article_title, LENGTH(article_content) AS content_length FROM articles ORDER BY content_length DESC;
  ```
  - Selects the article title and content length from the articles table, ordering the results in descending order based on content length.
  - Content length is calculated by applying `LENGTH` to the article_content field.
- Example 3 - Formatting Adjustment:
```sql
SELECT CASE WHEN LENGTH(username) > 15 THEN LEFT(username, 15) + '...' ELSE username END AS adjusted_username FROM users;
```
- Selects adjusted user name for users based on the following criteria:
  - Users with usernames greater than 15 characters, the first 15 characters are displayed, followed by '...'.
  - For other users, their username is displayed as-is.
## SUBSTRING
- The SQL `SUBSTRING` Function allows you to extract specific portions of text or character data from a given string. This is especially useful for data manipulation tasks that involve isolating specific information from a larger text.
- The basic syntax of the `SUBSTRING` Function is as follows:
  ```sql
  SUBSTRING(string_expression, start_position, length);
  ```
  - The start_position parameter indicates where the extraction should begin and based on a 1-based index.
  - The length parameter determines the _maximum_ number of characters to be extracted.
- Common Use Cases:
  - Extracting Names: Retrieve the first or last name from a full name composite string.
  - Data Cleansing: Remove unnecessary characters or spaces.
  - Parsing Information: Separating data with delimiters for easier analysis.
- When handling variable-length text data, you can use `CHARINDEX` to find the index of a delimiter or space within a string dynamically.

### Best Practices
- Consider using the `LENGTH` function to determine the length parameter dynamically.
- Validate the start_position and length parameters to avoid errors.

### Practice Examples
- Example 1 - Extracting First Names:
  ```sql
  SELECT SUBSTRING(full_name, 1, CHARINDEX(' ', full_name) - 1) AS first_name FROM employees;
  ```
  - Extracts the first name from the full name by dynamically calculating the length of the first name using `CHARINDEX`.
- Example 2 - Removing Prefixes:
  ```sql
  SELECT SUBSTRING(product_name, 4, LEN(product_name)) AS cleaned_name FROM products;
  ```
  - **Note: The length parameter in the `SUBSTRING` Function indicates the _maximum_ number of characters to extract, starting at the start_position. If the number of remaining characters from this position in the string is less than that length, the function will simply return the rest of the string without throwing an error.**
- Example 3 - Parsing Dates:
  ```sql
  SELECT SUBSTRING(order_date, 6, 2) AS month FROM orders;
  ```
  - Selects the 6th and 7th characters of the order date. If the order date is formatted correctly, this represents the month.
## CHARINDEX OR SUBSTRING_INDEX
- The SQL `CHARINDEX` Function allows you to determine the position _of the first occurrence_ of a specific substring within a given string. This is very useful for the determining the presence/location of a specific text segment within larger content.
  - If the substring is not found, an index of 0 is returned.
  - `CHARINDEX` is **case-insensitive**. Use `COLLATE` for case-sensitive searches.
- The basic syntax of the `CHARINDEX` Function is as follows:
  ```sql
  CHARINDEX(substring, string_expression, starting_point);
  ```
  - Returns the **position** of a substring according to a 1-based index.
  - The third parameter is **optional** and allows you to specify a starting point instead of starting from the beginning of the string.
- The basic syntax of the `SUBSTRING_INDEX` function is as follows:
  ```sql
  SUBSTRING_INDEX(string, delimiter, count);
  ```
  - Returns a **substring** based on a delimiter's occurrence count.
  - The string parameter is the string or text you want to work with.
  - The delimiter parameter is the character or string that marks the split points of the string parameter.
  - The count parameter determines how many times the delimiter should be counted.
    - A positive count returns characters _before_ the nth occurrence from the left.
    - A negative count returns characters _after_ the nth occurrence from the right.
- Common Use Cases:
  - Searching for Keywords: Identifying specific keywords or terms within text data.
  - Validating Data: Ensuring the presence of required information in a dataset.
  - Extracting Information: Using the identified position for data extraction, using a function like `SUBSTRING`.

### Best Practices
- Use `CHARINDEX` to dynamically and efficiently locate substrings.
- Combine `CHARINDEX` with other functions for more complex data analysis.
- Consider handling cases where the substring is not found (0 is returned).

### Practice Examples
- Example 1 - Searching for Keywords:
  ```sql
  SELECT product_name FROM products WHERE CHARINDEX('premium', product_description) > 0;
  ```
  - Selects the product name for products whose description contains the word premium.
- Example 2 - Validating Data:
  ```sql
  SELECT order_id FROM orders WHERE CHARINDEX('@', customer_email) > 0;
  ```
  - Selects the ID of orders with a valid email (one that contains the '@' character).
- Example 3 - Extracting Information:
  ```sql
  SELECT SUBSTRING(
    text_data,
    CHARINDEX('[', text_data) + 1,
    CHARINDEX(']', text_data) - CHARINDEX('[', text_data) - 1
  ) AS extracted_data FROM content;

  SELECT SUBSTRING_INDEX(
    SUBSTRING_INDEX(text_data, '[', -1),
    ']',
    1
  ) AS extracted_data FROM content;
  ```
  - The first query returns the text within the first set of square brackets.
  - The second query returns the text within the last set of square brackets.
## TRIM
- The SQL `TRIM` Function allows you to remove leading or trailing spaces (or other specified characters) from a string. This is especially useful for tasks involving cleaning and formatting data and ensuring data integrity and consistency.
- The basic syntax of the `TRIM` Function is as follows:
  ```sql
  TRIM([characters FROM] string_expression);
  ```
  - The `FROM` keyword is used to specify the characters you want to remove. For example: `TRIM('.,?!' FROM text)` removes periods, commas, question marks, and exclamation marks from the string expression (it treats each character from the string_expression _individually_).
  - Depending on database and character encoding, `TRIM` might behave differently with multibyte characters.
- Common Use Cases:
  - Data Cleansing: Removing spaces that may cause data inconsistency.
  - Data Formatting: Ensuring consistent formatting of text data.
  - User Input Processing: Removing spaces from user-provided input.

### Best Practices
- Use `TRIM` to ensure consistent data formatting and integrity.
- Use caution when specifying characters to remove to avoid unintended data loss.
- Test the behavior of `TRIM` with multibyte characters if your data involves them.

### Practice Example
- Example 1 - Removing Leading and Trailing Spaces:
  ```sql
  SELECT TRIM(' ' FROM product_name) AS cleaned_name FROM products;
  ```
  - Removes spaces from the product name in the products table.
- Example 2 - Removing Trailing Characters:
  ```sql
  SELECT product_id, TRIM(TRAILING ' ' FROM product_name) AS trimmed_product_name, unit_price FROM products;
  ```
  - The `TRAILING` keyword here specifically removes the specified character(' ') from the **trailing** (right) side of the string. Leading spaces and spaces between words remain unaffected.
  - There are also `LEADING` and `BOTH` keywords that can be used to specify removal from the left side and both sides. **In any case, spaces between words are preserved**.
  - Using `TRAILING` is equivalent to using `RTRIM(string)` and using `LEADING` is equivalent to using `LTRIM(string)`.
## LEFT & RIGHT
- The SQL `LEFT` and `RIGHT` Functions allow you to extract a specific number of characters from the beginning (`LEFT`) or ending (`RIGHT`) of a string. This is particularly useful for tasks that involve isolating portions of text data.
- The basic syntax of the `LEFT` Function is as follows:
  ```sql
  LEFT(string_expression, length);
  ```
- The basic syntax of the `RIGHT` Function is as follows:
  ```sql
  RIGHT(string_expression, length);
  ```
- In both functions, the length parameter specifies the total number of characters to extract, starting from the beginning or end. The `LENGTH` Function can be used to calculate this parameter dynamically.
- Common Use Cases:
  - Data Extraction: Isolate prefixes or suffixes in text data.
  - Data Analysis: Analyze the beginning or end of text for insights.
  - Formatting: Prepare data for presentation by truncating long text.

### Best Practices
- Use `LEFT` and `RIGHT` to effectively extract relevant data.
- Combine `LEFT` and `RIGHT` with other functions for more sophisticated data manipulation.
- Consider handling cases where the specified length exceeds the actual string length.

### Practice Examples
- Example 1 - Extracting Prefixes:
  ```sql
  SELECT LEFT(full_name, 10) AS short_name FROM employees;
  ```
- Example 2 - Extracting Suffixes:
  ```sql
  SELECT RIGHT(product_code, 4) AS last_digits FROM products;
  ```
- Example 3 - Data Presentation:
  ```sql
  SELECT LEFT(article_content, 100) + '...' AS summarized_content FROM articles;
  ```
## UPPER & LOWER
- The SQL `UPPER` and `LOWER` Functions allow you to convert text to uppercase and lowercase respectively. This is particularly useful for ensuring consistent text formatting. These functions can also be very helpful when performing case-insensitive comparisons.
- The basic syntax of the `UPPER` Function is as follows:
  ```sql
  UPPER(string_expression);
  ```
- The basic syntax of the `LOWER` Function is as follows:
  ```sql
  LOWER(string_expression);
  ```
- Common Use Cases:
  - Data Consistency: Ensuring uniform text formatting.
  - Data Analysis: Making case-insensitive comparisons.
  - Display Formatting: Presenting data in a consistent manner.

### Best Practice
- Utilize `UPPER` and `LOWER` for consistent data formatting.
- Use `UPPER` and `LOWER` in comparison queries for accurate results.
- Use caution when applying `UPPER` and `LOWER` to data that will be used for case-sensitive operations.
- **Always store original data, without case modifications, separately for future reference.**

### Practice Examples
- Example 1 - Uppercase Formatting:
  ```sql
  SELECT UPPER(product_name) AS capitalized_name FROM products;
  ```
- Example 2 - Case-Insensitive Comparison:
  ```sql
  SELECT product_name FROM products WHERE LOWER(product_name) = 'smartphone';
  ```
- Example 3 - Display Formatting:
  ```sql
  SELECT CONCAT(UPPER(first_name), ' ', LOWER(last_name)) AS formatted_name FROM customers;
  ```
## EXTRACT
- The SQL `EXTRACT` Function allows you to retrieve specific components (year, month, day, hour, minute, second), as a number, from a given date or time value. This is especially useful for analyzing and manipulating temporal data.
- The basic syntax of the `EXTRACT` Function is as follows:
  ```sql
  EXTRACT(field FROM source);
  ```
  - Here, the field parameter refers to the part of the date (year, month, day, etc.) and source is the date or time value.
  - Common fields include `YEAR`, `MONTH`, `DAY`, `DAYOFWEEK`, `HOUR`, `MINUTE`, and `SECOND`.
  - Always consider time zone differences when working with Date and Time Functions in SQL. The default time zone for these functions is typically UTC.
- Common Use Cases:
  - Data Analysis: Extracting the year, month, or day for trend analysis.
  - Time Analysis: Retrieving hours or minutes for time-based insights.
  - Date Calculations: Performing calculations using the extracted components.

### Best Practices
- Utilize `EXTRACT` for analyzing and manipulating date and time data.
- Combine `EXTRACT` with other functions to perform more sophisticated analysis.
- Verify data consistency before performing date-based operations.

### Practice Examples
- Example 1 - Extracting Year:
  ```sql
  SELECT EXTRACT(YEAR FROM order_date) AS order_year FROM orders;
  ```
- Example 2 - Extracting Month:
  ```sql
  SELECT EXTRACT(MONTH FROM hire_date) AS hire_month FROM employees;
  ```
- Example 3 - Time-based Calculations:
  ```sql
  SELECT order_id, EXTRACT(HOUR FROM order_time) AS order_hour FROM orders;
  ```
## COALESCE
- The SQL `COALESCE` Function allows you to handle null values effectively by providing a fallback value. This is especially useful for tasks that involve data validation, calculation, and presentation, ensuring reliable data processing and analysis.
- The basic syntax of the `COALESCE` Function is as follows:
  ```sql
  COALESCE(expression1, expression2, ...);
  ```
  - The function will return the first non-null value among the provided expressions. The expressions will be evaluated in order until the first non-null value is found.
- Common Use Cases:
  - Data Validation: Replacing null values with meaningful defaults.
  - Calculation: Performing operations with potentially null values. **When you try to perform a computation such as `SUM` with a null value, the value will be ignored as if it were zero. If all values are null, `SUM` will return null.**
  - Data Presentation: Ensuring data consistency in reports.

### Best Practices
- Use `COALESCE` to handle null values and ensure data reliability.
- Combine `COALESCE` with other functions for more complex data analysis.
- Choose meaningful defaults that align with your data context.

### Practice Examples
- Example 1 - Data Validation:
  ```sql
  SELECT product_name, COALESCE(unit_price, 0.00) AS adjusted_price FROM products;
  ```
  - Sets a default price by explicitly providing 0.00 as an expression (which will never be null).
- Example 2 - Calculation:
  ```sql
  SELECT order_id, COALESCE(quantity * unit_price, 0) AS total_amount FROM order_details;
  ```
- Example 3 - Data Presentation:
  ```sql
  SELECT CONCAT(first_name, ' ', COALESCE(last_name, '')) AS full_name FROM customers;
  ```
## SUBQUERY IN CONDITION
- In SQL, using subqueries within conditional statements can be very useful, especially when retrieving information from related tables, comparing values, and applying filters based on specific conditions.
- Using subqueries in conditional logic allows for dynamic decision making based on data relationships and values. Subqueries are typically used within the `WHERE`, `FROM`, and `HAVING` Clauses, and many other clauses, for tailored data retrieval.
- Subqueries can also be used within aggregate functions like `SUM`, `COUNT`, and `MAX`.
- Common Use Cases:
  - Filtered Selection: Retrieving rows from a table based on related conditions.
  - Subquery Comparisons: Comparing values with results from another query.
  - Subqueries in `CASE` Statements: Using subqueries to generate conditional results.

### Best Practices
- Plan your queries carefully to ensure that your subqueries provide accurate results.
- Optimize queries for performance by using appropriate indexes.
- Verify subquery behavior with simple data before using it in production.

### Practice Examples
- Example 1 - Subquery in `WHERE` Clause:
  ```sql
  SELECT product_name, unit_price FROM products WHERE unit_price > (SELECT AVG(unit_price) FROM products);
  ```
  - Selects product name and unit price for products with a higher-than-average unit price.
- Example 2 - Subquery with `IN` Operator:
  ```sql
  SELECT customer_name FROM customers WHERE customer_id IN (SELECT customer_id FROM orders WHERE order_status = 'Shipped');
  ```
  - Selects the name of customers who have placed orders that have shipped.
- Example 3 - Subquery in `CASE` Statement:
  ```sql
  SELECT
    product_name,
    CASE
      WHEN unit_price > (SELECT AVG(unit_price)FROM products) THEN 'Above Avg'
      ELSE 'Below Avg'
    END AS price_category
  FROM products;
  ```
  - Selects product name and price category, where price category is 'Above Avg' for products with an above average unit price and 'Below Avg' otherwise.
## WITH STATEMENTS (CTEs)
- The SQL Common Table Expression (CTE) provides a powerful and readable mechanism to structure complex queries. This enables the breakdown of complex calculations and operations into more manageable and logical steps. CTEs are also crucial for maintaining clarity and efficiency in SQL code.
- A CTE is a temporary result set (table) defined within the execution scope of a `WITH` Statement. CTEs help to simplify complex queries by dividing them into smaller, reuseable components.
  - CTEs are usually preferred over subqueries because they run once and the result set is stored in working memory. A subquery is executed for each row that the main query acts on, which can be less efficient if that subquery is returning the same result each time.
  - Generally speaking, there is no case where you would _have_ to use a subquery over a CTE, so it's always better to use a CTE for readability and maintainability.
- The basic syntax of a CTE is as follows:
  ```sql
  WITH cte_name AS (
      -- Query defining the CTE
      SELECT ...
  )
  SELECT ...
  FROM cte_name;
  ```
- Benefits of CTEs:
  - Improved Readability: Simplifies complex queries by breaking them down into logical steps.
  - Reuseable Code: Avoids repeating the execution of code by referencing the same CTE multiple times.
  - Reduced Temporary Table Usage: Eliminates the need for temporary tables in certain scenarios.
- Common Use Cases:
  - Simplifying Complex Queries: Breaking down complex queries into logical, reuseable parts.
  - Reusability: Reusing the same logic multiple times in a single query.
  - Hierarchical Data: Handling parent-child relationships with recursive CTEs.
  - Data Aggregation: Performing aggregations and joining the results with other data.
- Stages of Aggregation With CTEs:
  1. Extract relevant data for each group.
  2. Perform aggregate functions on the extracted data.
  3. Combine the results and perform further calculations.

### Best Practices
- Use meaningful aliases for CTEs to improve clarity.
- Limit the scope of CTEs to a single query to avoid unnecessary complication.
- Optimize the CTE by ensuring it only retrieves necessary data.

### Practice Examples
- Example 1 - Cumulative Sales by Month:
  ```sql
  WITH Stage1 AS (
      SELECT
          DATE_FORMAT(order_date, '%Y-%m') AS month,
          order_amount AS amount
      FROM orders
  ),
  CumulativeSales AS (
      SELECT
          month,
          SUM(amount) AS monthly_sales
      FROM Stage1
      GROUP BY month
  )
  SELECT *
  FROM CumulativeSales;
  ```
  - Collects the formatted month and order amount from orders, then uses that to calculate the cumulative sales per month.
  - **Note: A CTE can only be referenced by the statement immediately following it.**
- Example 2 - Incremental Average Rating:
  ```sql
  WITH Stage1 AS (
      SELECT
          rating_date,
          AVG(rating) AS daily_avg_rating
      FROM ratings
      GROUP BY rating_date
  ),
  IncrementalAverage AS (
      SELECT
          rating_date,
          AVG(daily_avg_rating) OVER (ORDER BY rating_date) AS cumulative_avg_rating
      FROM Stage1
  )
  SELECT *
  FROM IncrementalAverage;
  ```
  - `OVER (ORDER BY rating_date)` provides the incremental, cumulative daily average for each day.
- Example 3 - Comparative Revenue Growth:
  ```sql
  WITH Stage1 AS (
      SELECT
          region,
          SUM(CASE WHEN year = 2022 THEN revenue END) AS revenue_2022,
          SUM(CASE WHEN year = 2023 THEN revenue END) AS revenue_2023
      FROM sales
      GROUP BY region
  ),
  GrowthCalculation AS (
      SELECT
          region,
          revenue_2022,
          revenue_2023,
          (revenue_2023 - revenue_2022) / revenue_2022 AS growth_rate
      FROM Stage1
  )
  SELECT *
  FROM GrowthCalculation;
  ```
## WINDOW FUNCTIONS
- The SQL Window Function allows you to perform calculations across a "window" of rows related to the current row. They offer a level of flexibility and insight that traditional aggregate functions cannot achieve.
- Window functions operate within a specified "window" of rows related to the current row. They can calculate results without changing the result set. Aggregate and Ranking Functions are examples of Window Functions in SQL.
- The basic syntax of a Window Function is as follows:
  ```sql
  function_name() OVER (PARTITION BY partition_expression ORDER BY order_expression ROWS/RANGE frame_specification);
  ```
  - The `OVER` Clause defines the "window" or set of related rows within a query result set on which a calculation is performed. **Unlike the `GROUP BY` Clause, it performs calculations without collapsing the individual rows of the result.** This is beneficial because it allows aggregate or ranking information to be displayed alongside individual row details.
  - The `PARTITION BY` Clause divides the query result set into partitions (groups of rows) to which the window function is **independently applied** (applied to each row in the table). The function calculation resets when the partition boundary is crossed. The entire result set (up to the current row in the table) is treated as a default partition when omitted.
    - The default window frame for a window function (when no partition is specified) **with `ORDER BY`** is as follows: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This goes from the first row of the partition up to and including the current row.
    - The default window frame for a window function (when no partition is specified) **without `ORDER BY`** is as follows: `RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`. This includes all rows in the partition.
    - **Note: The _partition_ for the window function is the entire result set by default, but the _window frame_ is as explained above.**
  - The `ORDER BY` Clause defines the logical order of rows within each partition. **This does not affect the final order of the query output**; a separate ORDER BY must be used at the end of the SQL query for that.
  - The `ROWS` or `RANGE` Clauses further limits the rows within a partition that constitute the _window frame_ for the current row's calculation, based on physical row position (`ROWS`) or logical value range (`RANGE`). For example, `ROWS BETWEEN 1 PRECEDING AND CURRENT ROW` would sum values from the previous row up to and including the current row.
    - The primary difference between `ROWS` and `RANGE` is whether the frame is determined by physical row position or logical value range.
    - The `ROWS` clause treats every row as a distinct entity based on its position in the sorted list, regardless of its actual value. In other words, it counts exactly the number of rows you specify.
    - It is ideal, for example, when you need to calculate strict moving averages or sums, where the value increases row-by-row, even if there are rows with duplicate values.
    - The `RANGE` clause groups rows together if they have the same value in the `ORDER BY` column. These rows are considered **peers**. In other words, it looks at the _value_ of the `ORDER BY` column, rather than the physical count of rows. If multiple rows have the same value (like the same date), they are all processed together in the same frame.
    - It is ideal, for example, for time-series analysis, where you want to include all transactions from a specific period (e.g., "all sales on this date").
    - Duplicates are treated as follows: If you have 10 sales on 2026-01-01, a `RANGE` cumulative sum will show the total for all 10 sales on every single row for that date. In other words, they are processed as a single group.
    - **`ROWS` is considered to be more ideal than `RANGE` for enhanced performance and precision.**
- Common Use Cases:
  - Ranking and Ordering: Determining rankings based on specific criteria.
  - Aggregations: Computing running totals, moving (incremental) averages, cumulative sums.
  - Analytical Insights: Identifying trends and patterns in a dataset.
- Common Types of Window Functions:
  - Ranking Functions: `RANK()`, `DENSE_RANK()`, `NTILE()`.
  - Aggregate Functions: `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()`.
  - Analytical Functions: `LEAD()`, `LAG()`, `FIRST_VALUE()`, `LAST_VALUE()`.

### Best Practices
- Understand the logic and purpose of each Window Function.
- Use appropriate ordering and partitioning to get the desired results.
- Optimize queries for performance when using Window Functions.

### Practice Examples
- Example 1 - Calculating Row Number:
  ```sql
  SELECT product_name, unit_price, ROW_NUMBER() OVER (ORDER BY unit_price DESC) AS row_num FROM products;
  ```
  - `ROW_NUMBER` assigns a unique sequential integer to rows within the partition of a result set, starting at 1 for the first row. In this case, it will number the rows based on the specified order.
  - `OVER (ORDER BY unit_price DESC)` causes the rows within the partition (the entire table by default) to be numbered according to unit price in descending order.
  - In this case, since we've ordered by unit price, the row_num indicates the rank of each product based on its price.
- Example 2 - Calculating Running Total:
  ```sql
  SELECT order_date, order_amount, SUM(order_amount) OVER (ORDER BY order_date) AS running_total FROM orders;
  ```
  - No partition was specified in the `OVER` Clause, so the entire table, is the partition by default.
  - The default _window frame_ for this function ends up being `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` since `ORDER BY` was specified in the `OVER` Clause.
  - `ORDER BY order_date` specifies that the rows should be processed in the order of order_date. Essentially, it means that for each row, the sum will include all previous rows (in terms of order date), plus the current row.
  - For each row in the table, the query takes the order_amount of that row and adds it to the cumulative total of all previous rows (ordered by order_date).
  - For rows with duplicate dates, all rows for that date will be included in the running total. This would cause the running total to "jump", then "level off" for all rows with that duplicate date.
- Example 3 - Calculating Moving Average:
  ```sql
  SELECT date, revenue, AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING) AS moving_avg FROM sales;
  ```
  - Calculates the moving average for each order.
  - `ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING` creates a dynamic sliding window for every row in your result set. This window will contain 5 rows, consisting of the two previous rows, the current row, and the two following rows.
  - The window will shrink to include only rows that exist for rows at the beginning and end of the table, where there aren't 2 preceding rows or two following rows. For example, at the first row, the window will include the current row and two following rows.
## WINDOW FUNCTION WITH AGGREGATE FUNCTION
- The SQL Window Function can be combined with various Aggregate Functions to perform more advanced data retrieval and analysis. When used within Window Functions, the aggregate function is applied across the specific "window" of rows defined by the CTE. This facilitates gaining insights into trends, rankings, and comparisons within the defined window.
- Common Use Cases:
  - Running Totals: Calculate cumulative totals within a specific window of rows.
  - Moving Averages: Compute averages for a range of rows around each row to which the Window Function is applied.
- The basic syntax of using Aggregate Functions within Window Functions is as follows:
  ```sql
  aggregate_function(column) OVER (PARTITION BY partition_expression ORDER BY order_expression ROWS/RANGE frame_specification);
  ```
  - `SUM()`, `AVG()`, `COUNT()`, `MIN()`, and `MAX()` are common aggregate functions used with Window Functions.

### Best Practices
- Understand the specific purpose of each aggregate function used with a Window Function.
- Choose the appropriate window frame that suits analysis requirements.
- Test and validate results with sample data to ensure accuracy before using in production.

### Practice Examples
- Example 1 - Running Total of Sales:
  ```sql
  SELECT date, revenue, SUM(revenue) OVER (ORDER BY date) AS running_total FROM sales;
  ```
  - Calculates the running total of revenue (sales), ordered by order date. Additionally includes the date.
  - Note, since `ORDER BY` was specified, but the partition was not, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This means duplicates (rows with the same date) will be grouped together in the window frame for each row.
- Example 2 - Moving Average of Orders:
  ```sql
  SELECT order_date, order_amount, AVG(order_amount) OVER (ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING) AS moving_avg FROM orders;
  ```
  - Selects the order date, order amount, and sliding average for a window that includes the two previous rows, the current row, and the next two rows, ordered by order date.
- Example 3 - Comparative Ranking:
  ```sql
  SELECT product_name, unit_price, RANK() OVER (PARTITION BY category_id ORDER BY unit_price) AS price_rank FROM products;
  ```
  - Selects the product name, unit price, and rank of that product for its category, based on unit price.
  - `PARTITION BY category_id` groups products based on their category. The ranking calculation starts over at 1 for each new category.
  - `RANK()` assigns a rank to each product, based on a competition-style ranking. If two products in the same category have the same price, they receive the same rank.
  - However, when there is a tie, the next ranking is skipped. For example, when two products tie for first place, the next product (ordered by price) receives a rank of 3 (there is no second place). `DENSE_RANK()` should be used to avoid these gaps.
## ROW NUMBER
- The SQL `ROW_NUMBER()` Window Function allows you to assign a unique, sequential integer to each row in a partitioned and ordered result set, without impacting original data. This is particularly useful for tasks such as ranking, pagination, and general unique identification.
- The basic syntax of the `ROW_NUMBER()` Function is as follows:
  ```sql
  ROW_NUMBER() OVER (PARTITION BY partition_expression ORDER BY order_expression);
  ```
  - The `PARTITION BY` Clause is generally considered **optional** while the `ORDER BY` Clause is considered **mandatory** for consistent results.
  - This function **cannot be used** in a `WHERE` Clause because Window Functions are logically processed after this clause is applied.
- Common Use Cases:
  - Ranking: Assigning a unique rank to each row based on a specified ordering.
  - Pagination: Selecting a specific range of rows for display or analysis.
  - Generating Unique Identifiers: Creating unique identifiers for records.
- Benefits:
  - Sequential Ranking: Assign unique, sequential numbers to rows.
  - Efficient Pagination: Easily select specific result set ranges.
  - Creating Unique Identifiers: Generate temporary unique identifiers.

### Best Practices
- Understand the purpose and requirements of `ROW_NUMBER()`.
- Combine `ROW_NUMBER()` with other functions for advanced analysis.
- Verify results by testing sample data before implementing.

### Practice Examples
- Example 1 - Ranking Products by Price:
  ```sql
  SELECT product_name, unit_price, ROW_NUMBER() OVER (ORDER BY unit_price DESC) AS price_rank FROM products;
  ```
  - Selects product name, unit price, and price rank from products. Price rank is calculated by applying `ROW_NUMBER()`, then specifying a descending order based on unit price for the window.
- Example 2 - Paginating Employee Records:
  ```sql
  SELECT employee_id, first_name, last_name, ROW_NUMBER() OVER (ORDER BY last_name) AS row_num FROM employees WHERE row_num BETWEEN 11 AND 20;
  ```
  - **This query is invalid for several reasons:**
    - You cannot reference the row_num alias inside the `WHERE` Clause of the same query in which it was defined.
    - Since the conditions in a `WHERE` Clause are logically processed before the Window Function, you couldn't reference the results of the function inside the `WHERE` Clause either way.
    - To fix this, you must use a CTE to calculate the row number, then use that CTE in the main query:
      ```sql
      WITH RankedEmployees AS (
          SELECT 
              employee_id, 
              first_name, 
              last_name, 
              ROW_NUMBER() OVER (ORDER BY last_name) AS row_num 
          FROM employees
      )
      SELECT * 
      FROM RankedEmployees 
      WHERE row_num BETWEEN 11 AND 20;
      ```
  - If the query was valid, it would employee ID, first name, last name, and row_num from employees. row_num is calculated by applying `ROW_NUMBER()`, then specifying an ascending order based on last name for the window. Finally, it filters the results to only include employees ranked 11 through 20, inclusive.
- Example 3 - Generating Unique IDs:
  ```sql
  SELECT order_id, ROW_NUMBER() OVER (ORDER BY order_date) AS unique_order_id FROM orders;
  ```
  - Selects order ID, then a unique order ID calculated based on ascending order date.
## RANK
- The SQL `RANK()` Window Function allows you to assign a unique rank to each row in a partitioned and ordered result set based on specified criteria. This is especially useful for creating ordered rankings while handling ties gracefully.
  - `RANK()` handles ties as follows: When two or more rows in a column have the same value, they are assigned the same value and the subsequent ranking is skipped. For example, if two rows were assigned a rank of one, the next row would be assigned a rank of 3 instead of 2. If three rows had a rank of one, the next row would be assigned a rank of 4.
- The basic syntax of the `RANK()` Window Function is as follows:
  ```sql
  RANK() OVER (PARTITION BY partition_expression ORDER BY order_expression);
  ```
  - The `PARTITION BY` Clause is generally considered **optional** while the `ORDER BY` Clause is considered **mandatory** for consistent results.
- Common Use Cases:
  - Competitive Analysis: Ranking data based on specific attributes.
  - Leaderboards: Creating leaderboards based on performance metrics.
  - Partitioned Ranking: Assigning ranking to partitioned data.
- Benefits:
  - Accurate Ranking: Assign ranking while considering ties and missing ranks.
  - Handling Partitioned Data: Rank within specific groups or partitions of data.
  - Verify results by testing with sample data before applying in production.

### Best Practices
- Understand the purpose and significance of `RANK()`.
- Utilize `RANK()` in conjunction with other functions to facilitate more advanced analysis.
- Use `DENSE_RANK()` or `ROW_NUMBER()` when different tie-handling behavior is required.

### Practice Examples
- Example 1 - Ranking Products by Sales:
  ```sql
  SELECT product_name, unit_price, RANK() OVER (ORDER BY unit_price DESC) AS price_rank FROM products;
  ```
  - Selects the product name, unit price and price rank from the products table. Products are ranked by unit price in descending order.
- Example 2 - Creating a Leaderboard:
  ```sql
  SELECT player_name, score, RANK() OVER (ORDER BY score DESC) AS player_rank FROM leaderboard;
  ```
  - Selects the player name, score, and player rank from the leaderboard table. Players are ranked based on score in descending order.
- Example 3 - Partitioned Ranking:
  ```sql
  SELECT department_id, employee_name, salary, RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dept_rank FROM employees;
  ```
  - Selects department ID, employee name, salary, and dept_rank from the employees table. Each department receives a separate ranking of employees based on salary in descending order.
## LEAD
- The SQL `LEAD()` Window Function allows you to access data from a subsequent row within a result set, allowing you to gain insights into the next rows without performing complex self joins or subqueries. This is particularly useful for comparing values, tracking trends, and performing time-series analysis.
- The basic syntax of the `LEAD()` Function is as follows:
  ```sql
  LEAD(column, offset, default_value) OVER (ORDER BY order_expression);
  ```
  - The offset parameter is used to specify how many hows ahead to look for the row that should be returned. When not specified, the function uses a default value of 1.
  - The default_value parameter determines the value if there is no subsequent value. When not specified, the function returns null.
  - `ORDER BY` is required in the `OVER` Clause while `PARTITION BY` is optional and divides the rows into groups.
- Common Use Cases:
  - Trend Analysis: Analyzing change in values over time.
  - Comparative Analysis: Comparing current and future values for insights.
  - Lagging Metrics: Identifying trends in data that evolve over time.
- Benefits:
  - Streamlined Analysis: Accessing subsequent rows of data without complex queries.
  - Simplified Trend Detection: Easily identify trends and changes in data.
  - Enhanced Decision-Making: Make informed decisions based on future values.

### Best Practices
- Understand the purpose and applications of the `LEAD()` Function.
- Combine the `LEAD()` Function with other Window Functions for more complex analysis.
- Verify results by testing with representative data samples before applying in production.

### Practice Examples
- Example 1 - Analyzing Sales Growth:
  ```sql
  SELECT date, sales_amount, LEAD(sales_amount, 1, 0) OVER (ORDER BY date) AS next_day_sales FROM sales;
  ```
  - Selects date, sales amount, and next day's sales for each row in the sales table. Next day's sales are determined by using `LEAD()` and passing 1 as an offset to look only at the next day (row), then applying a default value of 0. This is done in chronological order by specifying `ORDER BY date`.
- Example 2 - Comparing Monthly Revenues:
  ```sql
  SELECT month, revenue, LEAD(revenue) OVER (ORDER BY month) AS next_month_revenue FROM monthly_reports;
  ```
  - Selects month, revenue, and next_month_revenue for each monthly report. Next month's revenue is determined by using `LEAD()` with the default values of 1 and NULL for the offset and default_value. The results are calculated in chronological order by specifying `ORDER BY month`.
- Example 3 - Detecting Changes in Stock Price:
  ```sql
  SELECT date, stock_price, LEAD(stock_price) OVER (ORDER BY date) AS next_day_price_change FROM stock_prices;
  ```
  - Selects date, stock price, and next_day_price_change from the stock_prices table. The next day's price change is determined by checking the next row's stock price, ordered by date in ascending order.
## LAG
- The SQL `LAG()` Window Function allows you to access data from a preceding row within a dataset, allowing you to gain insights into historical values without performing complex self joins or subqueries. This particularly useful for tracking trends, comparing values, and conducting time-series analysis.
- The basic syntax of the `LAG()` Function is as follows:
  ```sql
  LAG(column, offset, default_value) OVER (ORDER BY order_expression);
  ```
  - The offset parameter is used to specify how many hows behind to look for the row that should be returned. When not specified, the function uses a default value of 1.
  - The default_value parameter determines the value if there is no preceding value. When not specified, the function returns null.
  - `ORDER BY` is required in the `OVER` Clause while `PARTITION BY` is optional and divides the rows into groups.
- Common Use Cases:
  - Historical Analysis: Analyzing changes in data values over time.
  - Comparative Analysis: Comparing current and past values for insights.
  - Leading Metrics: Identifying trends that evolve over time.
- Benefits:
  - Streamlined Analysis: Access data in past rows without performing complex queries.
  - Simplified Trend Analysis: Easily identify changes and trends in data.
  - Improved Decision-Making: Make informed decisions based on historical values.

### Best Practices
- Understand the purpose and applications of the `LAG()` Function.
- Combine the `LAG()` Function with other Window Function for complex analysis and comprehensive insights.
- Validate results by testing with relevant data samples.

### Practice Examples
- Example 1 - Monitoring Stock Price Trends:
  ```sql
  SELECT date, stock_price, LAG(stock_price, 1, 0) OVER (ORDER BY date) AS previous_day_price FROM stock_prices;
  ```
  - Selects the date, stock price, and previous_day_price from the stock_prices table. The previous day's price calculated by using `LAG` with an offset of 1 and default_value of 0. The partition is ordered by date in ascending order.
- Example 2 - Tracking Quarterly Revenue Changes:
  ```sql
  SELECT quarter, revenue, LAG(revenue) OVER (ORDER BY quarter) AS previous_quarter_revenue FROM quarterly_reports;
  ```
  - Selects the quarter, revenue, and previous_quarter_revenue from the quarterly_reports table. The previous quarter's revenue is calculated using the `LAG` Function with an offset of 1 (default) and a default_value of NULL (default). The partition is ordered by quarter in ascending order.
- Example 3 - Analyzing Customer Behavior:
  ```sql
  SELECT purchase_date, purchase_amount, LAG(purchase_amount) OVER (ORDER BY purchase_date) AS previous_purchase_amount FROM customer_purchases;
  ```
  - Selects the purchase date, purchase amount, and previous_purchase_amount from the customer_purchases table. The previous purchase amount is calculated with the `LAG` Function using an offset of 1 and a default_value of null. The partition is ordered in chronological order by purchase_date.
