While practicing problems, note difficult concepts with unrecognized patterns.

# Problems

1. Find customers whose most recent order amount is greater than their previous order amount:
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH order_history AS (
		    SELECT
		        customer_id,
		        order_date,
		        amount,
		        LAG(amount) OVER (
		            PARTITION BY customer_id
		            ORDER BY order_date
		        ) AS previous_order_amount,
		        ROW_NUMBER() OVER (
		            PARTITION BY customer_id
		            ORDER BY order_date DESC
		        ) AS order_rank -- Used to find the latest order for each customer
		    FROM orders
		)
		SELECT
		    customer_id,
		    amount AS latest_order_amount,
		    previous_order_amount
		FROM order_history
		WHERE order_rank = 1
		  AND amount > previous_order_amount;
		```
		- Instead of finding `previous_order_amount` and `latest_order` in separate CTEs, they can both be found in one CTE using different window functions.
			- `LAG(amount) OVER(PARTITION BY customer_id ORDER BY order_date` finds the amount of the most recent order for a given customer, returning `NULL` for the first order.
			- `ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date DESC)` gives the most recent order for each customer a ranking of 1.
		- In the main query, we filter based on `order_rank = 1` to find the most recent order, then `amount > previous_order_amount` to find records for which the current amount is greater than the previous.
		- `MAX(order_date) OVER(PARTITION BY customer_id ORDER BY order_date)` would calculate the **running** maximum for each customer, not a customer's most recent oder.
		- `MAX(order_date) OVER(PARTITION BY customer_id ORDER BY order_date)` doesn't need to be used because `ROW_NUMBER()` can easily find the most recent order.
		- In general, it's easier to use `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()` to find the minimum or maximum of a column without collapsing rows. `MAX()` should mainly be used with `GROUP BY`, not as a window function.
2. Find customers whose total transaction amount increased from their first calendar month to their last calendar month.
	- `transactions`: `[transaction_id, customer_id, transaction_date, amount]`
		- `transaction_id`: INT
		- `customer_id`: INT
		- `transaction_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH format_dates AS (
			SELECT
				customer_id,
				DATE_FORMAT(transaction_date, '%Y-%m') AS month,
				amount
			FROM transactions
		),
		transaction_totals AS (
			SELECT
				customer_id,
				month,
				SUM(amount) AS monthly_total
			FROM format_dates
			GROUP BY customer_id, month
		),
		first_and_last_months AS (
			SELECT
				customer_id,
				month,
				MIN(month) OVER (PARTITION BY customer_id) AS first_month,
				MAX(month) OVER (PARTITION BY customer_id) AS last_month,
				monthly_total
			FROM transaction_totals
		)
		SELECT
			customer_id,
			MAX(CASE
				WHEN month = first_month
				THEN monthly_total
			END) AS first_month_total,
			MAX(CASE
				WHEN month = last_month
				THEN monthly_total
			END) AS last_month_total
		FROM first_and_last_months
		GROUP BY customer_id
		HAVING MAX(CASE
			WHEN month = last_month
			THEN monthly_total
		END) >				
		MAX(CASE
			WHEN month = first_month
			THEN monthly_total
		END);
		```
		- Stage 1: Format the dates from `YYYY-MM-DD` to `YYYY-MM` in order to compare first and last monthly transaction totals **across years**, not per year.
		- Stage 2: Calculate transaction totals per customer and month.
		- Stage 3: Find the first and last month for each customer.
		- Stage 4: Select `customer_id`, `first_month_total` and `last_month_total` for customer's whose `last_month_total > first_month_total`.
			- The selective `MIN()` and `MAX()` are needed here because the `first_and_last_months` CTE **does not** collapse rows when finding the first and last month.
			- We only want one record per customer_id in the final result.
			- `MIN()` and `MAX()` are redundant since the `monthly_total` filtered by the `CASE` statement return the same result, but it's still needed in order to properly `GROUP BY customer_id`.
3. Find employees whose current salary is higher than their previous recorded salary.
	- `employees`: `[employee_id, employee_name, department, salary, salary_date]`
		- `employee_id`: INT
		- `employee_name`: VARCHAR(10)
		- `department`: VARCHAR(30)
		- `salary`: INT
		- `salary_date`: DATE (YYYY-MM-DD)
	- Solution:
		```sql
		WITH previous_salaries AS (
		    SELECT
		        employee_id,
		        employee_name,
		        salary,
		        salary_date,
		        LAG(salary, 1) OVER (PARTITION BY employee_id ORDER BY salary_date) AS previous_salary,
		        MAX(salary_date) OVER (PARTITION BY employee_id) AS latest_salary_date
		    FROM employee_salaries
		)
		SELECT
		    employee_id,
		    employee_name,
		    previous_salary,
		    salary AS current_salary
		FROM previous_salaries
		WHERE salary_date = latest_salary_date
		    AND salary > previous_salary;
		```
4. Find customers whose most recent completed order is larger than their average completed order amount across all of their completed orders.
	- `orders:` `[order_id, customer_id, order_date, status, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `status`: VARCHAR(10)
		- `amount`: INT
	- Solution:
		```sql
		WITH completed_orders AS (
		    SELECT
		        customer_id,
		        order_date,
		        amount,
		        AVG(amount) OVER (PARTITION BY customer_id) AS avg_order_amount,
		        MAX(order_date) OVER (PARTITION BY customer_id) AS latest_order_date
		    FROM orders
		    WHERE status = 'completed'
		)
		
		SELECT
		    customer_id,
		    amount AS order_amount,
		    avg_order_amount
		FROM completed_orders
		WHERE order_date = latest_order_date
		    AND amount > avg_order_amount
		```
		- While `MAX()` works here theoretically, it **would not work** if there were two orders on the same `latest_order_date`.
		- Get into the habit of using `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()` to find maximums, instead of `MAX()`.
1. Find users who made their first purchase within 3 calendar days of their first login.
	- `user_events`: `[event_id, user_id, event_type, event_date]`
		- `event_id`: INT
		- `user_id`: INT
		- `event_type`: VARCHAR(20)
		- `event_date`: DATE (YYYY-MM-DD)
	- Solution:
		```sql
		WITH first_event_dates AS (
		    SELECT
		        event_id,
		        user_id,
		        event_type,
		        event_date,
		        MIN(
		            CASE
		                WHEN event_type = 'login' THEN event_date
		            END
		        ) OVER (PARTITION BY user_id) AS first_login_date,
		        MIN(
		            CASE
		                WHEN event_type = 'purchase' THEN event_date
		            END
		        ) OVER (PARTITION BY user_id) AS first_purchase_date
		    FROM user_events
		)
		
		SELECT DISTINCT
		    user_id,
		    first_login_date,
		    first_purchase_date
		FROM first_event_dates
		WHERE DATEDIFF(first_purchase_date, first_login_date) BETWEEN 0 AND 3
		```
		- Another good example of using conditional `MIN()` or `MAX()` to find different types of minimums within the same query / CTE.
1. Find customers whose total order amount is greater than the average total order amount across all customers who have placed at least one order.
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(20)
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH valid_customers AS (
		    SELECT
		        c.customer_id,
		        c.customer_name
		    FROM customers c
		    WHERE EXISTS (
		        SELECT 1
		        FROM orders o
		        WHERE o.customer_id = c.customer_id
		    )
		),
		customer_totals AS (
		    SELECT
		        vc.customer_id,
		        vc.customer_name,
		        SUM(o.amount) AS total_order_amount
		    FROM valid_customers vc
		    JOIN orders o ON vc.customer_id = o.customer_id
		    GROUP BY vc.customer_id, vc.customer_name
		),
		average_total AS (
		    SELECT
		        AVG(total_order_amount) AS avg_total
		    FROM customer_totals
		)
		
		SELECT
			ct.customer_id,
			ct.customer_name,
			ct.total_order_amount
		FROM customer_totals ct
		JOIN average_total at ON ct.total_order_amount > at.avg_total
		ORDER BY ct.total_order_amount DESC;
		```
		- Example of using a `JOIN` with out an equality condition. An alternative would have been to use `CROSS JOIN` with a `WHERE` clause to filter out totals that are less than or equal to the average.
			```sql
			SELECT
				ct.customer_id,
				ct.customer_name,
				ct.total_order_amount
			FROM customer_totals ct
			CROSS JOIN average_total at
			WHERE ct.total_order_amount > at.avg_total
			ORDER BY ct.total_order_amount DESC;
			```
		- **Possible Simplifications**:
			- The first CTE isn't necessary since the INNER JOIN in the second CTE already eliminates customers who haven't placed orders.
	- **Alternative Solution (Using Window Function)**:
		```sql
		WITH customer_totals AS (
		    SELECT
		        vc.customer_id,
		        vc.customer_name,
		        SUM(o.amount) AS total_order_amount
		    FROM valid_customers vc
		    JOIN orders o ON vc.customer_id = o.customer_id
		    GROUP BY vc.customer_id, vc.customer_name
		),
		totals_with_average AS (
		    SELECT
		        customer_id,
		        customer_name,
		        total_order_amount,
		        AVG(total_order_amount) OVER() AS avg_total
		    FROM customer_totals
		)
		
		SELECT
		    customer_id,
		    customer_name,
		    total_order_amount
		FROM totals_with_average
		WHERE total_order_amount > avg_total
		ORDER BY ct.total_order_amount DESC;
		```
		- Using `AVG()` with `OVER()` provides an alternate way of getting the global average within the same query, eliminating the separate `average_total` CTE and the subsequent cross join.
1. Find accounts whose total withdrawals exceed 50% of their total deposits.
	- Only include accounts that have **at least one** withdrawal.
	- Order by the ratio of withdrawals to deposits, highest first.
	- `transactions`: `[transaction_id, account_id, transaction_date, transaction_type, amount]`
		- `transaction_id`: INT
		- `account_id`: INT
		- `transaction_date`: DATE (YYYY-MM-DD)
		- `transaction_type`: VARCHAR(20)
		- `amount`: INT
	- Solution:
		```sql
		SELECT
		    account_id,
		    total_withdrawals,
		    total_deposits
		FROM (
		    SELECT
		        account_id,
		        SUM(
		            CASE
		                WHEN transaction_type = 'withdrawal' THEN amount
		                ELSE 0
		            END
		        ) AS total_withdrawals,
		        SUM(
		            CASE
		                WHEN transaction_type = 'deposit' THEN amount
		                ELSE 0
		            END
		        ) AS total_deposits
		    FROM transactions
		    GROUP BY account_id
		) AS account_totals
		WHERE total_withdrawals > 0
		    AND total_deposits > 0
		    AND (total_withdrawals * 1.0 / total_deposits) > 0.5
		ORDER BY (total_withdrawals * 1.0 / total_deposits) DESC;
		```
		- Use a subquery instead of a CTE when you would otherwise only need one CTE.
		- Always give derived tables (subqueries in `FROM`) an alieas.
		- `total_withdrawls` **and** `total_deposits` need to be greater than 0 to avoid divide-by-zero errors.
		- Remember to either explicitly `CAST` or use `* 1.0` when dividing integers in SQL.
2. Find customers whose total order amount is greater than $500, but whose individual orders never exceed $300.
	- Order by total_order_amount descending.
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH valid_customers AS (
		    SELECT
		        customer_id,
		        MAX(amount) AS max_order_amount
		    FROM orders
		    GROUP BY customer_id
		    HAVING MAX(amount) <= 300
		)
		SELECT
		    customer_id,
		    SUM(amount) AS total_order_amount
		FROM orders
		WHERE customer_id IN (SELECT customer_id FROM valid_customers)
		GROUP BY customer_id
		HAVING SUM(order_amount) > 500
		ORDER BY total_order_amount DESC;
		```
	- **Simplified Solution**:
		```sql
		SELECT
			customer_id,
			SUM(amount) AS total_order_amount
		FROM orders
		GROUP BY customer_id
		HAVING MAX(amount) <= 300
		 AND SUM(amount > 500)
		ORDER BY total_order_amount DESC;
		```
		- Notice that the CTE in the original solution and main query are both grouping by `customer_id`. This is a sign both conditions can be checked in just one query.
3. Find customers who have at least one order that has not been fully paid.
	- An order is considered fully paid when: `total payments for the order >= order amount`.
	- A customer should appear **once**, even if they have multiple unpaid orders.
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- `payments`: `[payment_id, order_id, payment_date, amount]`
		- `payment_id`: INT
		- `order_id`: INT
		- `payment_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Return: `customer_id | customer_name | unpaid_order_count | total_unpaid_amount`
	- Solution:
		```sql
		WITH total_payments AS ( -- Total payments for all orders.
		    SELECT
		        order_id,
		        SUM(amount) AS total_order_payment
		    FROM payments
		    GROUP BY order_id
		),
		unpaid_orders AS ( -- Unpaid order counts and amounts per customer.
		    SELECT
		        o.customer_id,
		        COUNT(
		            CASE
		                WHEN p.order_id IS NULL
		                  OR p.total_order_payment < o.amount
		                THEN 1
		            END
		        ) AS unpaid_order_count,
		        SUM(
		            CASE
		                WHEN p.order_id IS NULL
		                    THEN o.amount
		                WHEN p.total_order_payment < o.amount
		                    THEN o.amount - p.total_order_payment
		                ELSE 0
		            END
		        ) AS total_unpaid_amount
		    FROM orders o
		    LEFT JOIN total_payments p
		        ON o.order_id = p.order_id
		    GROUP BY o.customer_id
		    HAVING COUNT( -- Exclude customers with an unpaid_order_count < 1.
		        CASE
		            WHEN p.order_id IS NULL
		              OR p.total_order_payment < o.amount
		            THEN 1
		        END
		    ) > 0
		)
		SELECT
		    c.customer_id,
		    c.customer_name,
		    u.unpaid_order_count,
		    u.total_unpaid_amount
		FROM customers c
		JOIN unpaid_orders u
		    ON c.customer_id = u.customer_id
		ORDER BY u.total_unpaid_amount DESC;
		```
		- I aggregated payments by `order_id` first to ensure there was a one-to-one relationship between the aggregated payments and orders before joining them. Otherwise, joining an order to multiple payment rows could duplicate the order row and cause the order amount to be incorrectly summed.
4. Find employees whose monthly sales increased every month in which they made sales.
	- Requirements:
		- An employee must have sales in **at least two months**.
		- Compare the **monthly total sales**, not individual sales.
		- Every month's total must be strictly greater than the previous month's total.
		- Missing months should **not** count as failures. For example:
		- January → March is okay; you only compare months in which the employee actually had sales.
	- `employees`: `[employee_id, employee_name, department]`
		- `employee_id`: INT
		- `employee_name`: VARCHAR(30)
		- `department`: VARCHAR(30)
	- employee_sales: `[sale_id, employee_id, sale_date, amount]`
		- `sale_id`: INT
		- `employee_id`: INT
		- `sale_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Return: `employee_id | employee_name | department`.
	- Solution:
		```sql
		WITH monthly_sales_totals AS (
		    SELECT
		        employee_id,
		        DATE_FORMAT(sale_date, "%Y-%m") AS sale_month,
		        SUM(amount) AS total_sales
		    FROM employee_sales
		    GROUP BY
		        employee_id,
		        DATE_FORMAT(sale_date, "%Y-%m")
		    HAVING COUNT(employee_id) >= 2
		),
		monthly_sales_changes AS (
		    SELECT
		        employee_id,
		        sale_month,
		        (
		            total_sales -
		            LAG(total_sales, 1, 0) OVER(PARTITION BY employee_id ORDER BY sale_month)
		        ) AS monthly_sales_difference
		    FROM monthly_sales_totals
		)
		
		SELECT
		    t1.employee_id,
		    t1.employee_name,
		    t1.department
		FROM employees t1
		JOIN monthly_sales_changes t2
		    ON t1.employee_id = t2.employee_id
		GROUP BY t1.employee_id
		HAVING COUNT(
		    WHEN t2.monthly_sales_difference <= 0 THEN 1
		) = 0;
		```
		- Problems:
			- `HAVING COUNT(employee_id) >= 2` is filtering the wrong thing:
				- This filter asks: "Does this employee have at least two **sales records in this particular month**?"
				- The requirement is: "Does this employee have at least two **sales records in this particular month**?"
			- The `LAG()` default of `0` creates a problem:
				- This gives a difference of `total_sales` for the first months, which technically works.
				- A cleaner solution would be: `LAG(total_sales) OVER (...)`.
					- This gives the first month's difference as `NULL`.
					- Then we can explicitly ignore the first month when checking whether all changes are positive.
	- Clean Solution:
		```sql
		WITH monthly_sales_totals AS (
		    SELECT
		        employee_id,
		        DATE_FORMAT(sale_date, '%Y-%m') AS sale_month,
		        SUM(amount) AS total_sales
		    FROM employee_sales
		    GROUP BY
		        employee_id,
		        DATE_FORMAT(sale_date, '%Y-%m')
		),
		monthly_sales_changes AS (
		    SELECT
		        employee_id,
		        sale_month,
		        total_sales,
		        total_sales -
		        LAG(total_sales) OVER (
		            PARTITION BY employee_id
		            ORDER BY sale_month
		        ) AS monthly_sales_difference
		    FROM monthly_sales_totals
		)
		SELECT
		    e.employee_id,
		    e.employee_name,
		    e.department
		FROM employees e
		JOIN monthly_sales_changes m
		    ON e.employee_id = m.employee_id
		GROUP BY
		    e.employee_id,
		    e.employee_name,
		    e.department
		HAVING COUNT(*) >= 2
		   AND MIN(monthly_sales_difference) > 0;
		```
1. Find customers whose monthly spending increased for at least two **consecutive** month-over-month transitions.
	- Requirements:
		- Aggregate multiple orders within the same month.
		- Only compare months in which the customer actually placed orders.
		- A customer needs **at least 3 active months** to possibly qualify.
		- The increases must be **strictly greater**.
	- Return: `customer_id`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH monthly_orders AS (
		    SELECT
		        customer_id,
		        DATE_FORMAT(order_date, '%Y-%m') AS order_month,
		        SUM(amount) AS monthly_total
		    FROM orders
		    GROUP BY
		        customer_id,
		        DATE_FORMAT(order_date, '%Y-%m')
		),
		monthly_changes AS (
		    SELECT
		        customer_id,
		        order_month,
		        monthly_total,
		        LAG(monthly_total, 1) OVER (PARTITION BY customer_id ORDER BY order_month) AS previous_total_1,
		        LAG(monthly_total, 2) OVER (PARTITION BY customer_id ORDER BY order_month) AS previous_total_2
		    FROM monthly_orders
		)
		
		SELECT DISTINCT
		    customer_id
		FROM monthly_changes
		WHERE previous_total_1 IS NOT NULL
		    AND previous_total_2 IS NOT NULL
		    AND monthly_total > previous_total_1
		    AND previous_total_1 > previous_total_2;
		```