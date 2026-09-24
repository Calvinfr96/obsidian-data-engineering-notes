---
aliases:
  - "solving_sql_problems"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-structured-query-language-learning/SQL Notes/solving_sql_problems.md"
imported: 2026-09-24
---

# SQL Problem-Solving Walkthroughs

Review: [[02_technical_prep/sql/SQL Algorithms Practice|SQL Algorithms Practice]] · [[02_technical_prep/sql/SQL Optimization Principles|SQL Optimization Principles]]

## Extracting Keywords
- Aggregation: Total, Sum, Count, Average, Minimum, Maximum, etc.
  - Available Functions: `SUM`, `COUNT`, `AVG`, `MIN`, `MAX`, etc.
  - Don't mix up `SUM` and `COUNT`. `SUM` is for finding the total value of cells by adding. `COUNT` is for counting the occurrence of null and/or non-null values within cells.
- Grouping: Per (i.e. per customer, per year, per product, etc.).
  - Available Functions: `GROUP BY`
- Ranked Aggregations: Most, Top, Least, Highest, Lowest, etc.
  - Available Functions: `ORDER BY`, `LIMIT`, `RANK`, `DENSE_RANK`, etc.
- User Behavior/Events:
  - Count Users Over Time By Behavior: "Active Users", "Returning Users", etc.
    - Available Functions: `COUNT`, `DISTINCT`, `JOIN`, etc.
  - Negate or Exclusion: "Did not", "Never", "Missing", etc.
    - Available Functions: `NOT IN`, `LEFT JOIN`, `IS NULL`, `EXCEPT`, etc.
  - Filter by Count or Threshold: "Only", "Exactly", "At least", "More than", etc.
    - Available Functions: `HAVING`, `COUNT`, `WHERE`, `OVER`, etc.
- Logic and Conditions:
  - Condition Logic: If, When, Otherwise, etc.
    - Available Functions: `CASE`, `WHEN`, etc.
  - Set Comparisons: All, None, Any, Some, etc.
    - Available Functions: `ALL`, `ANY`, `EXISTS`, `NOT EXISTS`, etc. (**Not commonly used**)
  - Filtering Logic: Include, Exclude, etc.
    - Available Functions: `WHERE`, `NOT IN`, `JOIN`, etc.
- Joins and Relationships:
  - Tables Are Linked: "Related", "Mapped", "Belongs to", etc.
    - Available Functions" `JOIN` (using foreign keys), etc.
  - Unmatched Rows: "Missing", "No match", "Don't have, but include" etc.
    - Available Functions: `LEFT JOIN`, `WHERE`, `IS NULL`, etc.
  - Multiple Tables Involved: "Each", "Every", "Multiple entities", etc.
    - Available Functions: `JOIN`, `GROUP BY`, etc.
- Date Filtering:
  - Date Filtering: On, Before, After, Between, etc.
    - Available Functions: `WHERE` with `BETWEEN`, etc.
  - Relative Date Logic: "Last 30 days", "Previous month", etc.
    - Available Functions: `CURRENT_DATE`, `INTERVAL`, `DATE_SUB`, etc.
  - Day of Week Logic: Weekend, Weekday, etc.
    - Available Functions: `DAYOFWEEK`, `WEEKDAY`, etc.
- Trends and Comparisons:
  - Compare Two Time Periods: Increased, Decreased, Change, Growth, etc.
    - Available Functions: `LEAD`, `LAG`, `SELF JOIN`, etc.
  - Compare Same Metric Over Time Period: "Year over year", "Month over month", etc.
    - Available Functions: `DAY`, `YEAR`, `MONTH`, `SELF JOIN`, etc.
  - Absolute or Directional Difference: "Biggest change", etc.
    - Available Functions: `ABS`, `LEAD`, `LAG`, `MAX`, etc.
- Time Based or Sequence Analysis:
  - Compare Rows Over Time: Consecutive, Previous, Next, etc.
    - Available Functions: `LEAD`, `LAG`, `ROW_NUMBER`, `DATEDIFF`, etc.
  - Identify Top/Bottom Records Per Group: First, Last, Latest, Earliest, etc.
    - Available Functions: `ROW_NUMBER`, `RANK`, `MIN`, `MAX`, `GROUP BY`, `ORDER BY`, `LIMIT`, etc.
  - Track Values Across Events: "Over time", "Change over time", etc.
    - Available Functions: `LEAD`, `LAG`, `DATEDIFF`, etc. (Window functions will generally help here)
  - Difference Between Rows: "Difference between orders", etc.
    - Available Functions: `LEAD`, `LAG`, etc.
- Unions and Self Joins:
  - Merge Results From Multiple Queries: "Combine results from", etc.
    - Available Functions: `UNION`, `UNION ALL`, etc.
  - Add Rows From Different Sources: "Include both", "Either", "Or", etc.
    - Available Functions: `UNION`, etc.
  - Append Two Datasets Vertically: "List all X and all Y", etc.
    - Available Functions: `UNION`, etc.
  - Join Results From Two Tables in the Same Column: "From multiple sources", etc.
    - Available Functions: `UNION`, etc.
  - Compare Two Entries in the Same Table: "Compare a row to another row", etc.
    - Available Functions: `SELF JOIN`, etc.
  - Relate One Record to Another in the Same Table: Previous, Next, Consecutive, etc.
    - Available Functions: `SELF JOIN`, `LEAD`, `LAG`, etc.
  - Compare Two Time Points for the Same Entity: Before, After, etc.
    - Available Functions: `SELF JOIN`, etc.
  - Create Combinations of Related Rows in the Same Table: "Pair of records", etc.
    - Available Functions: `SELF JOIN`, etc.
  - Relate Two Instances of the Same Entity: "Same customer/product with different dates/values", etc.
    - Available Functions: `SELF JOIN`, etc.
  - Include Cumulative Average Alongside Individual Averages: "Include overall average alongside individual stats"
    - Available Functions: `CROSS JOIN`, `WITH`, etc. (**Use caution with cross joins**)


## Advice for Beginner Problems (Can Apply to Any Problem Type)

### Example 1 - High Satisfaction Hardware Tickets (Microsoft FAANG)
Write a query to count the number of support tickets with a customer satisfaction score of 4 or 5 for products in the 'Hardware' category.

Tables: support_tickets_microsoft, products_microsoft

```sql
Table Structure:

Table: support_tickets_microsoft
ColumnName		Datatype
ticket_id 		INT
product_id 		INT
employee_id		INT
ticket_date 	DATE 
resolution_time_hours INT 
customer_satisfaction INT 

Table: products_microsoft
ColumnName		Datatype
product_id 		INT 
product_name 	VARCHAR
category 		VARCHAR
```

- Pseudocode:
  - `COUNT` the number of support tickets.
  - Support tickets should have a customer satisfaction score of 4 or 5.
  - Support tickets should also be related to products in the hardware category.
  - In one query, we could find all support tickets related to hardware products.
  - In another query, we can take that result set and filter it down to tickets with satisfaction score of 4 or 5.
  - Alternatively, instead of using two queries, we could use an inner join on product ID, then filter the results.
- Solution:
  ```sql
  SELECT
    pm.product_name,
    COUNT(DISTINCT stm.ticket_id) AS high_satisfaction_tickets -- Although ticket_id likely won't have duplicates, using COUNT DISTINCT is a good practice.
  FROM support_tickets_microsoft AS stm JOIN products_microsoft AS pm
  ON stm.product_id = pm.product_id
  WHERE pm.category = 'Hardware' AND stm.customer_satisfaction IN (4,5)
  GROUP BY pm.product_name; -- This is required because you want a ticket count for each product. Without it, you'd get one count for all products.
  ```

### Example 2 - Analysis of Highly Rated and Frequently Reviewed Content (Netflix FAANG)
Identify content that has received an average rating of 4 or more and has at least 5 reviews. Provide the following details:

Content title

Release year

Average rating

Number of reviews

Tables: reviews_ntf, content

Table names are CASE SENSITIVE

```sql
Table Structure:

Table: reviews_ntf
ColumnName		Datatype
review_id 		INT
content_id 		INT
profile_id 		INT
rating 			INT
review_text 	TEXT
review_date 	DATE

Table: content
ColumnName		Datatype
content_id 		INT
title 			VARCHAR
series_id 		VARCHAR
release_date 	DATE
duration_minutes INT
age_rating 		ENUM
```

- Pseudocode:
  - Provide the following details for Netflix content:
    - Content title
    - Release year
    - Average rating
    - Number of reviews
  - Only provide these details under the following conditions:
    - Content has an average rating of 4 or more.
    - Content has at least 5 reviews.
  - There is no need for a CTE or subquery since the tables can be joined on content_id.
- Solution:
  ```sql
  SELECT
    c.title AS content_title,
    YEAR(c.release_date) AS release_year,
    AVG(r.rating) AS average_rating,
    COUNT(r.review_id) AS number_of_reviews
  FROM content AS c JOIN reviews_ntf AS r
  ON c.content_id = r.content_id
  GROUP BY c.title
  HAVING AVG(r.rating) >= 4 AND COUNT(r.review_id) >= 5;
  ```

### Example 3 - Average Product Price by Gender and Price Tier (Beginner SQL DE Interview)
For products with available stock above 100 and ordered after January 1, 2023, calculate the average price by gender. Additionally, label each average price as 'Premium' if it’s above 170, otherwise 'Affordable'. Round the average price to 2 decimal places.
Output should contain: gender, avg_price, and price_tier.

```sql
Table Structure:

Table: dim_product_nike
entry_id
gender
product_name
category_id
price
product_reviews
comfort_score
product_rating
order_date

Table: dim_category_nike
category_id
category_name
available_color
available_stocks
```

- Pseudocode:
  - Calculate the average price by gender (grouping required).
  - Label the average price as follows:
    - Premium if > 170
    - Affordable otherwise
    - Round to 2 decimal places.
  - Only perform the calculation for the following products:
    - available stock > 100
    - ordered after January 1, 2023
  - Display gender, average price, and price tier.
  - No CTE or subquery is required because average can be calculated using `GROUP BY`. You can also perform the conditional logic in the same query because `SELECT` is processed after the grouping.
- Solution:
  ```sql
  SELECT
    p.gender,
    ROUND(AVG(p.price), 2) AS average_price,
    (
      CASE
        WHEN AVG(p.price) > 170 THEN 'Premium'
        ELSE 'Affordable'
      END
    ) AS price_tier
  FROM dim_product_nike AS p JOIN dim_category_nike AS c
  ON p.category_id = c.category_id
  WHERE c.available_stocks > 100 AND p.order_date > '2023-01-01'
  GROUP BY p.gender;
  ```
  - Don't forget the `GROUP BY` and `WHERE` at the end of your query. The grouping is required because we need to calculate the average price by gender.
  - Also, don't forget that you can compare dates directly using Comparison Operators. There are no special functions required, as with date arithmetic.


## Advice for Intermediate Problems

### Example 1 - Most Video Views per Category (Intermediate SQL DE Interview)
Fetch the first 3 youtubers from each category who have the most video views

expected column: category, youtuber , video_views, rnk

```sql
Table Structure:

Table: youtubers_table
Youtuber_id
youtuber
subscribers
video_views
category
started

Table: dim_countries_youtube
youtuber_id
Country

Table: dim_category_youtube
category
money_earned_per_1k_views
```

- Pseudocode:
  - Fetch youtubers from each category.
  - Only fetch those with the most video views.
  - Of those only fetch the first 3.
- Solution:
  ```sql
  SELECT
    ranked_categories.category,
    ranked_categories.youtuber,
    ranked_categories.video_views,
    ranked_categories.rnk
  FROM (
    SELECT
      category,
      youtuber,
      video_views,
      DENSE_RANK() OVER (PARTITION BY category ORDER BY video_views DESC) AS rnk
    FROM youtubers_table
  ) AS ranked_categories
  WHERE ranked_categories.rnk <= 3;
  ```
  - A subquery was required here because we cannot limit the ranking to the top 3 in the same query where we do the ranking. `LIMIT` cannot be used with the `ORDER BY` in the `OVER` Clause.
  - Solution with CTE:
    ```sql
    WITH ranked_categories AS (
      SELECT
        category,
        youtuber,
        video_views,
        DENSE_RANK() OVER (PARTITION BY category ORDER BY video_views DESC) AS rnk
      FROM youtubers_table
    )

    SELECT
      ranked_categories.category,
      ranked_categories.youtuber,
      ranked_categories.video_views,
      ranked_categories.rnk
    FROM ranked_categories
    WHERE ranked_categories.rnk <= 3;
    ```

### Example 2 - New Price Calculation (Intermediate SQL DE Interview)
Find the price of each car at the time of sale. Display the two highest prices for each brand.
Assume the start price is from January 2023 and all the cars were sold later that year.
Columns to Display: brand, price

```sql
Table Structure:

Table: dim_brands_tesla
brand
start_price
monthly_inflation_rate
square_footage

Table: fact_sales_tesla
VIN
brand
payment_method
sale_date
```

- Pseudocode:
  - Find the price of each car at the time of sale.
  - Only display the two highest price cars for each brand.
  - Assume the start price is from 01/2023 and all cars were sold after that.
  - Display: brand, price. Only display for the two highest prices per brand.
  - **In a real interview, ask the interviewer what the purpose of the monthly_inflation_rate column is.**
  - In this case, we use a start date of '2023-01-01' and the sale_date to calculate the number of months. Then, use the monthly_inflation_rate to calculate the price increase, then add that to the start_price.
- Solution:
  ```sql
  WITH prices AS (
    SELECT
      VIN,
      f.brand,
    TRUNCATE(start_price * POWER(1 + monthly_inflation_rate/100, MONTH(sale_date) - 1), 0) AS current_price
    FROM fact_sales_tesla AS f JOIN dim_brands_tesla AS d ON f.brand = d.brand
  )

  SELECT
    DISTINCT brand,
    current_price 
  FROM (
    SELECT
      *,
      DENSE_RANK() OVER (PARTITION BY brand ORDER BY current_price DESC) AS rnk
    FROM prices
  ) AS brand_ranking
  WHERE brand_ranking.rnk <= 2;
  ```
  - Here, `DISTINCT` ensures the final report only lists each unique qualifying price once per brand. It ensures unique brand-price pairs that qualify for the top 2 rankings. For example, if the dealership sold 5 Model S cars, they'd all probably have the same start price and current price based on the inflation rate. Using `DISTINCT` ensures that, after the table is filtered using `WHERE`, only unique brands are chosen for each rank of 1 and 2.
  - Probably could have used another CTE instead of a subquery to calculate the ranking. That would have been more readable.

### Example 3 - Frequently Restocked Products (Walmart FAANG)
Find products that had stock replenished (quantity > 0) at least once between January and June 2023.

Output: product_id, product_name, restock_count

Tables: walmart_products, walmart_inventory_updates

Table names are CASE SENSITIVE

```sql
Table Structure:

Table: walmart_products
ColumnName		Datatype
product_id 		INT
product_name 	VARCHAR
category 	    VARCHAR
price 			DECIMAL
stock_count 	INT

Table: walmart_inventory_updates
ColumnName		Datatype
update_id 	    INT
product_id      INT
update_date     DATE
quantity_added  INT
```

- Pseudocode:
  - Display product_id, product_name, restock_count.
  - Only display for products that have had their stock replenished twice between 01/2023 and 06/2023
- Solution:
  ```sql
  SELECT
    p.product_id,
    p.product_name,
    COUNT(update_id) AS restock_count
  FROM walmart_inventory_updates AS iu JOIN walmart_products AS p
  ON iu.product_id = p.product_id
  WHERE iu.update_date BETWEEN '2023-01-01' AND '2023-07-01' AND
  iu.quantity_added > 0
  GROUP BY product_id, product_name
  HAVING COUNT(update_id) >= 1;
  ```

### Example 4 - Last post from top publisher (Intermediate SQL DE Interview)
Find the name and the content of the last post published by the top publisher (who has the most posts). 
Columns to Display: name, content

```sql
Table Structure:

Table: dim_publishers_reddit
publisher_id
name
city

Table: fact_reddit
post_id
publisher_id
comm_num
score
creation_date
content
```

- Pseudocode
  - Find the name and content of the last post for the top publisher.
  - Top publisher means the one with the most posts.
  - Display: name, content (LAST post).
- Solution:
  ```sql
  WITH publisher_stats AS (
    SELECT
      p.Publisher_id,
      p.Name,
      COUNT(f.Post_id) num_posts
    FROM dim_publishers_reddit p JOIN fact_reddit f
    USING(Publisher_id)
    GROUP BY p.Publisher_id, p.Name
    ORDER BY num_posts DESC
  ),
  post_stats AS (
    SELECT
      s.Publisher_id,
      s.Name,
      s.num_posts,
      RANK() OVER (ORDER BY s.num_posts DESC) AS publisher_rnk,
      RANK() OVER (PARTITION BY f.Publisher_id ORDER BY f.Creation_date DESC) AS post_date_rnk,
      f.Content
    FROM publisher_stats s JOIN fact_reddit f
    USING(Publisher_id)
    ORDER BY s.num_posts DESC
  )

  SELECT
      DISTINCT Name, Content
  FROM post_stats
  WHERE publisher_rnk = 1 AND post_date_rnk = 1;
  ```


## Advice for Advanced Problems

### Example 1 - High-Value Engaged Customers (Walmart FAANG)
Find customers who meet both criteria:
Total spent in 2023 > $1500
At least 5 reviews in 2023 with avg rating ≥ 4

Output: customer_name, total_spent, avg_rating, review_count

Tables: walmart_customers, walmart_orders, walmart_reviews

Table names are CASE SENSITIVE

```sql
Table Structure:

Table: walmart_customers
ColumnName		Datatype
customer_id     INT
customer_name 	VARCHAR
email           VARCHAR
join_date 	    DATE

Table: walmart_orders
ColumnName		Datatype
order_id        INT
store_id 	    INT
customer_id     INT
employee_id    	INT
order_date 	    DATE
total_amount 	DECIMAL

Table: walmart_reviews
ColumnName		Datatype
review_id 		INT
product_id 	    INT
customer_id     INT
rating         DECIMAL
review_date 	DATE
```

- Pseudocode:
  - Find customers who meet the following criteria:
    - Spent more than $1500 in 2023.
    - Left at least 5 reviews in 2023 with avg rating ≥ 4.
  - Display customer_name, total_spent, avg_rating, review_count
  - Need to calculate the total spend per customer in 2023.
  - Need to calculate the average rating per customers who left at least 5 reviews.
- Solution:
  ```sql
  WITH customer_review_stats AS (
    SELECT
      customer_id,
      COUNT(review_id) AS review_count,
      AVG(rating) AS avg_rating
    FROM walmart_reviews
    WHERE review_date BETWEEN '2023-01-01' AND '2024-01-01' -- You could also just use YEAR(review_date) = 2023
    GROUP BY customer_id
    HAVING COUNT(review_id) >= 5 AND AVG(rating) >= 4
  ),
  customer_spending_stats AS (
    SELECT
      customer_id,
      SUM(total_amount) AS total_spent
    FROM walmart_orders
    WHERE order_date BETWEEN '2023-01-01' AND '2024-01-01' -- You could also just use YEAR(order_date) = 2023
    GROUP BY customer_id
    HAVING SUM(total_amount) > 1500
  )

  SELECT
    customer_name,
    total_spent,
    avg_rating,
    review_count
  FROM customer_review_stats JOIN customer_spending_stats USING(customer_id)
  JOIN walmart_customers USING(customer_id);
  ```
  - A good approach to solving problems like these, which have layers (i.e. total spend, then review count and average), is to use CTEs rather than joins or subqueries.
  - In this case, the data can be prepared without joining the tables, so you should avoid doing so.
  - For presentation purposes, it's also best to order the data by total_spent in descending order.

### Example 2 - Year-Over-Year Sales Change (Advanced SQL DE Interview)
Calculate the year-over-year (YoY) percent change in total Nike product sales.
Return each year with the percentage change in sales compared to the previous year, rounded to 2 decimal points.
If a previous year does not exist for comparison, return NULL.

Columns to Display: order_year, percent_change

```sql
Table Structure:

Table Name: dim_product_nike
entry_id
gender
product_name
category_id
price
product_reviews
comfort_score
product_rating
order_date

Table Name: dim_category_nike
category_id
category_name
available_color
available_stocks
```

- Pseudocode:
  - Calculate the year-over-year change in total sales as a percentage.
  - Display each year with the percentage change.
  - Need to calculate the total sales per year.
  - Need to calculate the percentage change.
- Solution:
```sql
WITH yearly_sales_stats AS (
  SELECT
    YEAR(order_date) AS order_year,
    SUM(price) AS total_product_sales
  FROM dim_product_nike
  GROUP BY YEAR(order_date)
)

SELECT
  order_year,
  ROUND((
    total_product_sales - LAG(total_product_sales) OVER (ORDER BY order_year)) /
    LAG(total_product_sales) OVER (ORDER BY order_year) * 100, 2
  ) AS percent_change
FROM yearly_sales_stats;
```

### Example 3 - Cumulative Balance (Advanced SQL DE Interview)
Find the cumulative balance over transactions of the account owned by 'James Thompson'.
Columns to Display: Output the transaction dates and cumulative balances - columns should be: name, transaction_date, transaction_amount, cumulative_balance

```sql
Table Structure:

Table: dim_accounts_paypal
account_id
name
city
balance

Table: fact_transactions_paypal
transaction_id
payer_id
receiver_id
amount
transaction_date
```

- Pseudocode:
  - Filter table down to transactions for James Thompson's account.
  - Find the cumulative balance over transactions for this account.
  - Clarify with interviewer whether this means over time.
  - Clarify with interviewer if account_id = payer_id
  - Display: name, transaction_date, transaction_amount, cumulative_balance
  - Join the tables based on payer/receiver id.
  - Filter based on account name to get James Thompson's transactions.
  - Based on transaction date, find the current balance, previous balance, and amount
  - Use these columns to find the cumulative transaction total.
  - Use the initial balance and cumulative transaction total to find the cumulative balance.
- Solution:
  ```sql
  WITH cumulative_transactions AS (
    SELECT
      a.name,
      t.transaction_date,
      a.balance,
      (
        CASE
          WHEN a.account_id = t.payer_id THEN -1 * t.amount
          ELSE t.amount
        END
      ) AS transaction_amount,
      SUM(
        CASE
          WHEN a.account_id = t.payer_id THEN -1 * t.amount
          ELSE t.amount
        END
      ) OVER (ORDER BY t.transaction_date) AS transaction_total
    FROM fact_transactions_paypal t JOIN dim_accounts_paypal a
    ON a.account_id = t.payer_id OR a.account_id = t.receiver_id
    WHERE a.name = 'James Thompson'
    ORDER BY t.transaction_date
  )

  SELECT
    name,
    transaction_date,
    transaction_amount,
    (balance + transaction_total) AS cumulative_balance
  FROM cumulative_transactions;
  ```
