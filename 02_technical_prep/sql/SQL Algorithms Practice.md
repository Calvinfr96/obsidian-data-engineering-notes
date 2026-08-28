While practicing problems, note difficult concepts with unrecognized patterns.

# Problems

## General Problems

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
2. Find employees whose current salary is higher than their previous recorded salary.
	- `employees`: `[employee_id, employee_name, department, salary, salary_date]`
		- `employee_id`: INT
		- `employee_name`: VARCHAR(30)
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
3. Find customers whose most recent completed order is larger than their average completed order amount across all of their completed orders.
	- `orders:` `[order_id, customer_id, order_date, status, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `status`: VARCHAR(30)
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
4. Find customers whose total order amount is greater than the average total order amount across all customers who have placed at least one order.
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
1. Find customers whose total order amount is greater than $500, but whose individual orders never exceed $300.
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
2. Find employees whose monthly sales increased every month in which they made sales.
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
	- Attempted Solution:
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
3. Find customers whose monthly spending increased for at least two **consecutive** month-over-month transitions.
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
4. Find employees whose total sales are greater than the average total sales of employees in their department.
	- Requirements:
		- Calculate total sales per employee.
		- Compare each employee against the **average employee total within their own department**.
		- Employees with no sales should not appear.
		- Order by `department`, then `total_sales` descending.
	- Return: `employee_id | employee_name | department | total_sales`
	- `employees`: `[employee_id, employee_name, department]`
		- `employee_id`: INT
		- `employee_name`: VARCHAR(30)
		- `department`: VARCHAR(30)
	- `employee_sales`: `[sale_id, employee_id, sale_date, amount]`
		- `sale_id`: INT
		- `employee_id`: INT
		- `sale_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH employee_totals AS ( -- Calculate employee total sales.
		    SELECT
		        t1.employee_id,
		        t1.employee_name,
		        t1.department, -- Needed to calculate department-level averages later.
		        SUM(t2.amount) AS total_sales
		    FROM employees t1
		    JOIN employee_sales t2
		        ON t1.employee_id = t2.employee_id
		    GROUP BY
		        t1.employee_id,
		        t1.employee_name,
		        t1.department
		),
		department_averages AS ( -- Calculate department average sales.
		    SELECT
		        department,
		        AVG(total_sales) AS dept_avg_sales
		    FROM employee_totals
		    GROUP BY department
		)
		
		SELECT
		    t1.employee_id,
		    t1.employee_name,
		    t1.department,
		    t1.total_sales
		FROM employee_totals t1
		JOIN department_averages t2
		    ON t1.department = t2.department
		WHERE t1.total_sales > t2.dept_avg_sales
		ORDER BY
			department,
			total_sales DESC;
		```
		- Why did you use a separate aggregation and join instead of a window function?
> 	I used a separate aggregation because I can calculate the department-level average independently and then join it back to the employee-level totals. A window function would also work and would be more concise because I need the department average alongside each employee. I'd probably use the window-function approach here because it avoids the second aggregation and join.

1. Find customers who have at least two completed orders, and whose most recent completed order is fully paid.
	- Requirements:
		- Only completed orders count.
		- Customer must have **at least 2 completed orders**.
		- "Most recent" is determined by `order_date`.
		- The most recent completed order must have `total_payment >= order_amount`.
		- An order with no payments has `total_payment = 0`.
		- If two orders have the same `order_date`, use the larger `order_id` as the more recent order.
		- Return one row per customer.
	- Return: `customer_id | customer_name | latest_order_date | order_amount | total_payment`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `oder_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- `payments`: `[payment_id, order_id, payment_date, amount]`
		- `payment_id`: INT
		- `order_id`: INT
		- `payment_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Too fucking stupid to think of anything useful.
	- Brilliant Solution:
		```sql
		WITH payment_totals AS (
		    SELECT
		        order_id,
		        SUM(amount) AS total_payment
		    FROM payments
		    GROUP BY order_id
		),
		completed_orders AS (
		    SELECT
		        o.order_id,
		        o.customer_id,
		        o.order_date,
		        o.amount AS order_amount,
		        COALESCE(pt.total_payment, 0) AS total_payment,
		        ROW_NUMBER() OVER (
		            PARTITION BY o.customer_id
		            ORDER BY o.order_date DESC, o.order_id DESC
		        ) AS order_rnk,
		        COUNT(*) OVER (
		            PARTITION BY o.customer_id
		        ) AS completed_order_count
		    FROM orders o
		    LEFT JOIN payment_totals pt
		        ON o.order_id = pt.order_id
		    WHERE o.status = 'completed'
		)
		SELECT
		    c.customer_id,
		    c.customer_name,
		    co.order_date AS latest_order_date,
		    co.order_amount,
		    co.total_payment
		FROM customers c
		JOIN completed_orders co
		    ON c.customer_id = co.customer_id
		WHERE co.order_rnk = 1
		  AND co.completed_order_count >= 2
		  AND co.total_payment >= co.order_amount
		ORDER BY c.customer_id;
		```
		- Step 1: Calculate payment totals per order (all orders).
		- Step 2: Combine payment information from step 1 with **completed** order information.
			- `LEFT JOIN` and `COALESCE` are used to capture orders without a total payment.
			- `ROW_NUMBER` is used to rank orders by order date and order ID  (in descending order). This helps find the most-recent order and uses `order_id` as a tie-breaker.
			- `COUNT(*)` is used to find completed order count per customer. `COUNT(*)` can be used because `WHERE` filters specifically for completed orders.
		- Step 3: Take the columns from step 2 and filter for:
			- Orders with a rank of 1 (most-recent orders).
			- A completed order count (partitioned by customer ID) of at least 2.
			- total payment greater than order amount.
			- Finally, order by customer ID.
2. Find customers who placed orders in **at least 3 different months during 2026**, and whose **average monthly order count is at least 2**.
	- Requirements:
		- Only `completed` orders count.
		- Only orders from 2026 count.
		- Multiple orders in the same month count as **one active month**.
		- `avg_monthly_orders` should be calculated using only the months in which the customer placed orders.
		- Customers with fewer than 3 active months should be excluded.
		- Order by `avg_monthly_orders` descending.
	- Return: `customer_id | customer_name | active_months | avg_monthly_orders`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `oder_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Attempted Solution:
		```sql
		WITH monthly_customer_orders AS (
		    SELECT
		        customer_id,
		        MONTH(order_date) AS order_month,
		        COUNT(order_id) AS order_count
		    FROM orders
		    WHERE YEAR(order_date) = 2026
		    GROUP BY customer_id, MONTH(order_date);
		),
		average_order_count AS (
		    SELECT
		        customer_id,
		        order_month,
		        AVG(order_count) OVER (PARTITION BY customer_id) AS avg_monthly_orders
		    FROM monthly_customer_orders
		)
		
		SELECT
		    c.customer_id,
		    c.customer_name,
		    COUNT(o.order_month) AS active_months,
		    avg_monthly_orders
		FROM customers c
		JOIN average_order_count o
		    ON c.customer_id = o.customer_id
		WHERE avg_monthly_orders >= 2
		GROUP BY
		    c.customer_id,
		    c.customer_name
		HAVING COUNT(o.order_month) >= 3
		```
		- Problems:
			- Forget the `completed filter`: The first CTE should use `WHERE YEAR(order_date) = 2026 AND status = 'completed'`.
			- `avg_monthly_orders` shouldn't be filtered in `WHERE`.
				- This works conceptually because the value was calculated in the previous CTE, but there's a subtle problem: you're filtering the monthly rows **before** the final customer aggregation.
				- Conceptually, the customer-level conditions belong together in the final aggregation.
	- Clean Solution:
		```sql
		WITH monthly_customer_orders AS (
		    SELECT
		        customer_id,
		        MONTH(order_date) AS order_month,
		        COUNT(order_id) AS order_count
		    FROM orders
		    WHERE YEAR(order_date) = 2026
		      AND status = 'completed'
		    GROUP BY
		        customer_id,
		        MONTH(order_date)
		),
		customer_summary AS (
		    SELECT
		        customer_id,
		        COUNT(order_month) AS active_months,
		        AVG(order_count) AS avg_monthly_orders
		    FROM monthly_customer_orders
		    GROUP BY customer_id
		    HAVING COUNT(order_month) >= 3
		       AND AVG(order_count) >= 2
		)
		SELECT
		    c.customer_id,
		    c.customer_name,
		    cs.active_months,
		    cs.avg_monthly_orders
		FROM customers c
		JOIN customer_summary cs
		    ON c.customer_id = cs.customer_id
		ORDER BY cs.avg_monthly_orders DESC;
		```
1. Find customers whose **monthly completed-order revenue increased for three consecutive months during 2026**.
	- Requirements:
		- Only `completed` orders count.
		- Only orders from 2026 count.
		- Revenue should be calculated at the **customer + month** level.
		- The three months must be **consecutive calendar months**.
		- A customer can qualify if they have _any_ three-month consecutive period with increasing revenue.
	- Return: `customer_id`
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `oder_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Too fucking stupid to think of anything useful.
	- Brilliant Solution:
		```sql
		WITH monthly_orders AS (
		    SELECT
		        customer_id,
		        DATE_FORMAT(order_date, '%Y-%m-01') AS order_month,
		        SUM(amount) AS monthly_revenue
		    FROM orders
		    WHERE status = 'completed'
		      AND YEAR(order_date) = 2026
		    GROUP BY
		        customer_id,
		        DATE_FORMAT(order_date, '%Y-%m-01')
		),
		monthly_history AS (
		    SELECT
		        customer_id,
		        order_month,
		        monthly_revenue,
		        LAG(order_month, 1) OVER (
		            PARTITION BY customer_id
		            ORDER BY order_month
		        ) AS previous_month,
		        LAG(monthly_revenue, 1) OVER (
		            PARTITION BY customer_id
		            ORDER BY order_month
		        ) AS previous_revenue,
		        LAG(order_month, 2) OVER (
		            PARTITION BY customer_id
		            ORDER BY order_month
		        ) AS two_months_ago,
		        LAG(monthly_revenue, 2) OVER (
		            PARTITION BY customer_id
		            ORDER BY order_month
		        ) AS revenue_two_months_ago
		    FROM monthly_orders
		)
		SELECT DISTINCT
		    customer_id
		FROM monthly_history
		WHERE order_month = DATE_ADD(previous_month, INTERVAL 1 MONTH)
		  AND previous_month = DATE_ADD(two_months_ago, INTERVAL 1 MONTH)
		  AND monthly_revenue > previous_revenue
		  AND previous_revenue > revenue_two_months_ago;
		```
		- Step 1: Calculate monthly order revenue per customer. `DATE_FORMAT(order_date, '%Y-%m-01')` is used so date arithmetic can be performed properly later. Date arithmetic won't work with just `DATE_FORMAT(order_date, '%Y-%m')`.
		- Step 2: Use `LAG()` to find the previous month and previous revenue, going back 1 month and 2 months.
		- Step 3: In the final query, ensure that revenue increased during the 3-month period **and** that the months are **actually consecutive**. `SELECT DISTINCT` is used in case a customer has more than one qualifying 3-month period.
2. Find customers whose **latest order amount is greater than their first order amount**.
	- Requirements:
		- Customers must have at least **two orders**.
		- If multiple orders occur on the same date, use `order_id` to break the tie.
		- The first/latest order refers to chronological order by `order_date` and `order_id`.
	- Return: `customer_id | customer_name | first_order_amount | last_order_amount`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id` INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH first_and_last_orders AS (
		    SELECT
		        customer_id,
		        amount,
		        ROW_NUMBER() OVER (
		            PARTITION BY customer_id
		            ORDER BY order_date, order_id
		        ) AS first_order_rnk,
		        ROW_NUMBER() OVER (
		            PARTITION BY customer_id
		            ORDER BY order_date DESC, order_id DESC
		        ) AS last_order_rnk
		    FROM orders
		)
		SELECT
		    c.customer_id,
		    c.customer_name,
		    MAX(
		        CASE
		            WHEN first_order_rnk = 1 THEN amount
		        END
		    ) AS first_order_amount,
		    MAX(
		        CASE
		            WHEN last_order_rnk = 1 THEN amount
		        END
		    ) AS latest_order_amount
		FROM customers c
		JOIN first_and_last_orders o
		    ON c.customer_id = o.customer_id
		GROUP BY
		    c.customer_id,
		    c.customer_name
		HAVING COUNT(*) >= 2
		   AND MAX(
		        CASE
		            WHEN last_order_rnk = 1 THEN amount
		        END
		   ) >
		       MAX(
		        CASE
		            WHEN first_order_rnk = 1 THEN amount
		        END
		   );
		```
		- In the main query, `MAX()` is somewhat redundant since the rankings are already partitioned by `customer_id`. It's only used to get the first and last orders on the same row.
3. Find customers who, during **2026**:
	- Had at least **4 completed orders**.
	- Had completed orders in at least **3 different quarters**.
	- Their **highest-quarter revenue** represented at least **50% of their annual completed-order revenue**.
	- Requirements:
		- Only `completed` orders count.
		- Only 2026 orders count.
		- `annual_revenue` = total completed revenue for 2026.
		- `highest_quarter_revenue` = the largest of Q1, Q2, Q3, Q4 completed revenue.
		- `highest_quarter_percentage` = `highest_quarter_revenue / annual_revenue`.
		- A customer must have activity in at least 3 distinct quarters.
		- Order by `highest_quarter_percentage` descending.
	- Return: `customer_id | customer_name | annual_revenue | highest_quarter_revenue | highest_quarter_percentage`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id` INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Solution:
		```sql
		WITH sales_quarters AS (
		    SELECT
		        order_id,
		        customer_id,
		        (
		            CASE
		                WHEN MONTH(order_date) BETWEEN 1 AND 3 THEN 'Q1'
		                WHEN MONTH(order_date) BETWEEN 4 AND 6 THEN 'Q2'
		                WHEN MONTH(order_date) BETWEEN 7 AND 9 THEN 'Q3'
		                ELSE 'Q4'
		            END
		        ) AS sales_quarter,
		        amount
		    FROM orders
		    WHERE YEAR(order_date) = 2026 AND status = 'completed'
		),
		quarterly_stats AS (
		    SELECT
		        customer_id,
		        sales_quarter,
		        SUM(amount) AS quarterly_revenue,
		        COUNT(order_id) AS quarterly_order_count
		    FROM sales_quarters
		    GROUP BY
		        customer_id,
		        sales_quarter
		),
		annual_stats AS (
		    SELECT
		        customer_id,
		        SUM(quarterly_revenue) AS annual_revenue,
		        MAX(quarterly_revenue) AS highest_quarter_revenue,
		        SUM(quarterly_order_count) AS annual_order_count,
		        COUNT(sales_quarter) AS active_quarters
		    FROM quarterly_stats
		    GROUP BY customer_id
		)
		
		SELECT
		    c.customer_id,
		    c.customer_name,
		    s.annual_revenue,
		    s.highest_quarter_revenue,
		    (s.highest_quarter_revenue * 1.0 / s.annual_revenue) AS highest_quarter_percentage
		FROM annual_stats s
		JOIN customers c
		    ON s.customer_id = c.customer_id
		WHERE
		    s.annual_order_count >= 4
		    AND s.active_quarters >= 3
		    AND (s.highest_quarter_revenue * 1.0 / s.annual_revenue) >= 0.5
		ORDER BY highest_quarter_percentage DESC;
		```
		- Originally, in the `annual_stats` CTE, you were using `OVER (PARTITION BY customer_id)` for all of the aggregations. When you notice this, and the other columns in your table don't represent data aggregated at a different level (or no level), **it's a sign you should use `GROUP BY` instead of window functions**.
		- Overall flow:
			1. Establish quarterly "buckets", based on `order_date`, which can be used to find quarterly statistics.
			2. Find quarterly statistics (`revenue` and `order_count`) by grouping by `customer_id` and `sales_quarter`.
			3. Find annual statistics (`revenue`, `highest_quarter_revenue`, and `order_count`) by grouping quarter statistics by `customer_id`.
			4. Join with `customers` table to retrieve `customer_name`, calculate `highest_quarter_percentage`, and apply appropriate filters using `WHERE`.

## Conditional Aggregation Problems

1. Find users who made their first purchase within 3 calendar days of their first login.
	- `user_events`: `[event_id, user_id, event_type, event_date]`
		- `event_id`: INT
		- `user_id`: INT
		- `event_type`: VARCHAR(30)
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
4. Find accounts whose total withdrawals exceed 50% of their total deposits.
	- Only include accounts that have **at least one** withdrawal.
	- Order by the ratio of withdrawals to deposits, highest first.
	- `transactions`: `[transaction_id, account_id, transaction_date, transaction_type, amount]`
		- `transaction_id`: INT
		- `account_id`: INT
		- `transaction_date`: DATE (YYYY-MM-DD)
		- `transaction_type`: VARCHAR(30)
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
5. Find customers whose total spending in 2026 Q1 (January–March) was greater than their total spending in 2026 Q2 (April–June).
	- Requirements:
		- Customers must have at least one order in either Q1 or Q2.
		- Missing quarters should be treated as `$0`.
		- Customers whose Q1 and Q2 totals are equal should not appear.
		- Order by the difference `(q1_total - q2_total)` descending.
	- Return: `customer_id | q1_total | q2_total`
	- `orders`: `[order_id, customer_id, order_date, amount]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH quarterly_totals AS (
			SELECT
				customer_id,
				SUM(
					CASE
						WHEN MONTH(order_date) BETWEEN 1 AND 3 THEN amount
						ELSE 0
					END
				) AS q1_total,
				SUM(
					CASE
						WHEN MONTH(order_date) BETWEEN 4 AND 6 THEN amount
						ELSE 0
					END
				) AS q2_total
			FROM orders
			WHERE YEAR(order_date) = 2026
			GROUP BY customer_id
		)
		
		SELECT
			customer_id,
			q1_total,
			q2_total
		FROM quarterly_totals
		WHERE q1_total > q2_total
		ORDER BY (q1_total - q2_total) DESC;
		```
1. Find customers who placed a **completed order in January 2026 and another completed order in February 2026**.
	- Requirements:
		- Only `completed` orders count.
		- January means `2026-01-01` through `2026-01-31`.
		- February means `2026-02-01` through `2026-02-28`.
		- A customer may have multiple orders in either month.
		- Return the **total revenue for each month**.
		- Customers must have orders in **both** months.
		- Order by `february_revenue` descending.
	- Return: `customer_id | customer_name | january_revenue | february_revenue`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Solution:
		```sql
		SELECT
		    c.customer_id,
		    c.customer_name,
		    SUM(
		        CASE
		            WHEN MONTH(o.order_date) = 1 THEN o.amount
		            ELSE 0
		        END
		    ) AS january_revenue,
		    SUM(
		        CASE
		            WHEN MONTH(o.order_date) = 2 THEN o.amount
		            ELSE 0
		        END
		    ) AS february_revenue
		FROM customers c
		JOIN orders o
		    ON c.customer_id = o.customer_id
		WHERE YEAR(o.order_date) = 2026 AND o.status = 'completed'
		GROUP BY
		    c.customer_id,
		    c.customer_name
		HAVING
		    COUNT(
		        CASE
		            WHEN MONTH(o.order_date) = 1 THEN order_id
		        END
		    ) >= 1
		    AND COUNT(
		        CASE
		            WHEN MONTH(o.order_date) = 2 THEN order_id
		        END
		    ) >= 1
		ORDER BY february_revenue DESC;
		```
2. Find employees whose **monthly sales increased from January through March 2026**.
	- Requirements:
		- Only sales from 2026 count.
		- Only employees with sales in **all three months** should qualify.
		- Multiple sales within a month should be summed.
		- The comparison is based on **total monthly sales**, not individual transactions.
		- Order by `march_sales` descending.
		- Return: `employee_id | employee_name | january_sales | february_sales | march_sales`
	- `employees`: `[employee_id, employee_name, department]`
		- `employee_id`: INT
		- `employee_name`: VARCHAR(30)
		- `department`: VARCHAR(30)
	- `employee_sales`: `[sale_id, employee_id, sale_date, amount]`
		- `sale_id`: INT
		- `employee_id`: INT
		- `sale_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Solution:
		```sql
		WITH monthly_sales AS (
		    SELECT
		        e.employee_id,
		        e.employee_name,
		        SUM(
		            CASE
		                WHEN MONTH(s.sale_date) = 1 THEN s.amount
		                ELSE 0
		            END
		        ) AS january_sales,
		        SUM(
		            CASE
		                WHEN MONTH(s.sale_date) = 2 THEN s.amount
		                ELSE 0
		            END
		        ) AS february_sales,
		        SUM(
		            CASE
		                WHEN MONTH(s.sale_date) = 3 THEN s.amount
		                ELSE 0
		            END
		        ) AS march_sales
		    FROM employees e
		    JOIN employee_sales s
		        ON e.employee_id = s.employee_id
		    WHERE YEAR(s.sale_date) = 2026
		    GROUP BY
		        e.employee_id,
		        e.employee_name
		    HAVING
		        COUNT(
		            CASE
		                WHEN MONTH(s.sale_date) = 1 THEN s.sale_id
		            END
		        ) >= 1
		        AND COUNT(
		            CASE
		                WHEN MONTH(s.sale_date) = 1 THEN s.sale_id
		            END
		        ) >= 1
		        AND COUNT(
		            CASE
		                WHEN MONTH(s.sale_date) = 1 THEN s.sale_id
		            END
		        ) >= 1
		)
		
		SELECT
		    employee_id,
		    employee_name,
		    january_sales,
		    february_sales,
		    march_sales
		FROM monthly_sales
		WHERE
		    february_sales > january_sales
		    AND march_sales > february_sales
		ORDER BY march_sales DESC;
		```
		- The `INNER JOIN` only ensures each employee has at least one sale in **all of 2026**. It doesn't ensure an employee has a sale in each month because grouping and aggregation happen after joing.
1. Find customers who:
	- Have placed **at least one completed order** in 2026.
	- Have **never had a cancelled order** in 2026.
	- Have at least **3 total orders** in 2026, regardless of status.
	- Return: `customer_id | customer_name | total_orders | total_revenue`
		- `total_orders` = **all** 2026 orders for the customer, regardless of status.
		- `total_revenue` = revenue from **completed** 2026 orders only.
	- `customers`: `[customer_id, customer_name]`
		- customer_id: INT
		- customer_name: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHA(30)
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
		        WHERE c.customer_id = o.customer_id
		        AND YEAR(o.order_date) = 2026
		        AND o.status = 'completed'
		    )
		    AND NOT EXISTS (
		        SELECT 1
		        FROM orders o
		        WHERE c.customer_id = o.customer_id
		        AND YEAR(o.order_date) = 2026
		        AND o.status = 'cancelled'
		    )
		)
		
		SELECT
		    c.customer_id,
		    c.customer_name,
		    COUNT(o.order_id) AS total_orders,
		    SUM(
		        CASE
		            WHEN o.status = 'completed' THEN o.amount
		            ELSE 0
		        END
		    ) AS total_revenue
		FROM valid_customers c
		JOIN orders o
		    ON c.customer_id = o.customer_id
		    AND YEAR(o.order_date) = 2026
		GROUP BY
		    c.customer_id,
		    c.customer_name
		HAVING COUNT(order_id) >= 3
		ORDER BY total_revenue DESC;
		```
		- Interesting case of using `EXISTS / NOT EXISTS` with conditional aggregation.
		- The second join condition is needed because `valid_customers` only filters for **customers** based on 2026 orders. **It doesn't actually filter orders based on the year**.

## EXISTS / NOT EXISTS Problems

1. Find customers who have placed at least one completed order but have never placed a cancelled order.
	- Requirements:
		- Customer must have **at least one** order where `status = 'completed'`.
		- Customer must have **zero** orders where `status = 'cancelled'`.
		- Customers with no orders should not appear.
		- Return each customer only once.
		- Order by `customer_id`.
	- Return: `customer_id | customer_name`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_state, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Solution:
		```sql
		SELECT
		    customer_id,
		    customer_name
		FROM customers c
		WHERE EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE c.customer_id = o.customer_id
		      AND o.status = 'completed'
		)
		AND NOT EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE c.customer_id = o.customer_id
		      AND o.status = 'cancelled'
		)
		ORDER BY customer_id;
		```
		- Good example of using both `EXISTS` and `NOT EXISTS`.
		- `EXISTS` is like a function that returns a boolean that the `WHERE` clause can evaluate. `WHERE EXISTS` isn't a clause in and of itself.
		- That's why you can use `AND NOT EXISTS` after the first `EXISTS`.
2. Find customers who have placed at least one order, and every order they have placed is completed.
	- Return: `customer_id | customer_name`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_state, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Solution:
		```sql
		SELECT
		    customer_id,
		    customer_name
		FROM customers c
		WHERE EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE o.customer_id = c.customer_id
		)
		AND NOT EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE o.customer_id = c.customer_id
		      AND o.status != 'completed'
		)
		ORDER BY customer_id;
		```
		- We need to make sure that a customer has placed at least one order: `WHERE o.customer_id = c.customer_id`.
		- We also need to make sure those customers don't have orders with a status **other than** 'completed': `WHERE o.status != 'completed'`.
3. Find customers whose every order is greater than $100.
	- Requirements:
		- The customer must have at least one order.
		- **Every** order for the customer must have `amount > 100`.
		- A customer with an order exactly equal to `$100` does **not** qualify.
	- Return: `customer_id | customer_name`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_state, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `order_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- Solution:
		```sql
		SELECT
		    customer_id,
		    customer_name
		FROM customers c
		WHERE EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE c.customer_id = o.customer_id
		)
		AND NOT EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE c.customer_id = o.customer_id
		        AND o.amount <- 100
		);
		```
		- The `c.customer_id = o.customer_id` condition is required in **every** correlated subquery. Without it, `EXISTS` will check the entire orders table.
4. Find employees whose every sale is greater than the average sale amount for their department.
	- Requirements:
		- The employee must have at least one sale.
		- Calculate the **average individual sale amount** for the employee's department.
		- Every sale made by the employee must be **strictly greater** than that department average.
	- Return: `employee_id | employee_name | department`
	- `employees`: `[employee_id, employee_name, department]`
		- `employee_id`: INT
		- `employee_name`: VARCHAR(30)
		- `department`: VARCHAR(30)
	- `employee_sales`: `[sale_id, employee_id, sale_date, amount]`
		- `sale_id`: INT
		- `employee_id`: INT
		- `sale_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Attempted Solution:
		```sql
		SELECT
		    employee_id,
		    employee_name,
		    department
		FROM employees e
		WHERE NOT EXISTS (
		    SELECT 1
		    FROM employee_sales s
		    WHERE e.employee_id = s.employee_id
		        AND s.amount > AVG(amount) OVER (PARTITION BY e.department)
		);
		```
		- Problems:
			- `employee_sales` doesn't contain `department`, so the window would need department information from `employees`.
			- We need the **department-wide average individual sale**, not an average computed over only the rows being considered by the subquery.
	- Clean Solution:
		```sql
		WITH department_averages AS (
		    SELECT
		        e.department,
		        AVG(s.amount) AS avg_sale_amount
		    FROM employees e
		    JOIN employee_sales s
		        ON e.employee_id = s.employee_id
		    GROUP BY e.department
		)
		SELECT
		    e.employee_id,
		    e.employee_name,
		    e.department
		FROM employees e
		JOIN department_averages d
		    ON e.department = d.department
		WHERE EXISTS (
		    SELECT 1
		    FROM employee_sales s
		    WHERE s.employee_id = e.employee_id
		)
		AND NOT EXISTS (
		    SELECT 1
		    FROM employee_sales s
		    WHERE s.employee_id = e.employee_id
		      AND s.amount <= d.avg_sale_amount
		)
		ORDER BY e.employee_id;
		```
		- The CTE calculates department averages.
		- `EXISTS` looks for employees with **at least** one sale.
		- `NOT EXISTS` excludes employees that have **any** sale amount at or below the average sale amount for their department.
5. Find customers who have at least one completed order and whose completed orders have an average payment coverage of at least 80%.
	- Requirements:
		- Only completed orders count.
		- An order with no payments has **0% payment coverage**.
		- An order can have multiple payment records.
		- A customer must have at least one completed order.
		- `avg_payment_coverage` is the average **across completed orders**, not the customer's total payments divided by total order value.
		- Only customers with `avg_payment_coverage >= 0.80` qualify.
		- Order by `avg_payment_coverage` descending.
	- Return: `customer_id | customer_name | completed_order_count | avg_payment_coverage`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `oder_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- `payments`: `[payment_id, order_id, payment_date, amount]`
		- `payment_id`: INT
		- `order_id`: INT
		- `payment_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Attempted Solution:
		```sql
		WITH payment_totals AS ( -- Find payment totals for completed orders.
		    SELECT
		        order_id,
		        SUM(amount) AS total_payment
		    FROM payments p
		    WHERE EXISTS (
		        SELECT 1
		        FROM orders o
		        WHERE p.order_id = o.order_id
		            AND o.status = 'completed'
		    )
		    GROUP BY order_id
		),
		payment_coverage AS ( -- Find payment coverage for completed orders.
		    SELECT
		        o.order_id,
		        o.customer_id,
		        (pt.total_payment * 1.0 / o.amount) AS payment_coverage
		    FROM orders o
		    JOIN payment_totals pt
		        ON o.order_id = pt.order_id
		)
		
		SELECT
		    c.customer_id,
		    c.customer_name,
		    COUNT(order_id) AS completed_order_count,
		    AVG(payment_coverage) AS avg_payment_coverage
		FROM customers c
		JOIN payment_coverage pc
		    ON c.customer_id = pc.customer_id
		GROUP BY
		    c.customer_id
		    c.customer_name
		HAVING AVG(payment_coverage) >= 0.8
		ORDER BY avg_payment_coverage DESC;
		```
		- Problems:
			- `payment_coverage`: `orders o JOIN payment_totals pt` drops orders without payments. An order with no payments has 0% payment coverage. The `JOIN` needs to be a `LEFT JOIN` to perserve this information.
			- `payment_totals`: Because `payment_coverage` needs to use a `LEFT JOIN`, `payment_totals` needs to filter for completed orders.
	- Clean Solution:
		```sql
		WITH payment_totals AS (
		    SELECT
		        p.order_id,
		        SUM(p.amount) AS total_payment
		    FROM payments p
		    WHERE EXISTS (
		        SELECT 1
		        FROM orders o
		        WHERE p.order_id = o.order_id
		          AND o.status = 'completed'
		    )
		    GROUP BY p.order_id
		),
		payment_coverage AS (
		    SELECT
		        o.order_id,
		        o.customer_id,
		        COALESCE(pt.total_payment, 0) * 1.0 / o.amount AS payment_coverage
		    FROM orders o
		    LEFT JOIN payment_totals pt
		        ON o.order_id = pt.order_id
		    WHERE o.status = 'completed'
		)
		SELECT
		    c.customer_id,
		    c.customer_name,
		    COUNT(pc.order_id) AS completed_order_count,
		    AVG(pc.payment_coverage) AS avg_payment_coverage
		FROM customers c
		JOIN payment_coverage pc
		    ON c.customer_id = pc.customer_id
		GROUP BY
		    c.customer_id,
		    c.customer_name
		HAVING AVG(pc.payment_coverage) >= 0.80
		ORDER BY avg_payment_coverage DESC;
		```
1. Find customers where every completed order is fully paid.
	- Requirements:
		- The customer must have at least one completed order.
		- **Every completed order** must be fully paid.
		- Orders with no payments are not fully paid.
		- Customers with no completed orders should not appear.
	- Return: `customer_id | customer_name`
	- `customers`: `[customer_id, customer_name]`
		- `customer_id`: INT
		- `customer_name`: VARCHAR(30)
	- `orders`: `[order_id, customer_id, order_date, amount, status]`
		- `order_id`: INT
		- `customer_id`: INT
		- `oder_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
		- `status`: VARCHAR(30)
	- `payments`: `[payment_id, order_id, payment_date, amount]`
		- `payment_id`: INT
		- `order_id`: INT
		- `payment_date`: DATE (YYYY-MM-DD)
		- `amount`: INT
	- Attempted Solution:
		```sql
		WITH payment_totals AS (
		    SELECT
		        p.order_id,
		        SUM(amount) AS total_payment
		    FROM payments p
		    WHERE EXISTS (
		        SELECT 1
		        FROM orders o
		        WHERE o.order_id = p.order_id
		            AND o.status = 'completed'
		    )
		    GROUP BY p.order_id
		),
		payment_coverage AS (
		    SELECT
		        o.order_id,
		        o.customer_id,
		        (COALESCE(pt.total_payment, 0) * 1.0 / o.amount) AS coverage
		    FROM orders o
		    LEFT JOIN payment_totals pt
		        ON o.order_id = pt.order_id
		    WHERE o.status = 'completed'
		),
		SELECT
		    c.customer_id,
		    c.customer_name
		FROM customers c
		WHERE NOT EXISTS (
		    SELECT 1
		    FROM payment_coverage pc
		    WHERE coverage < 1
		);
		```
		- Problems:
			- Main Query: `WHERE coverage < 1` needs to be `WHERE pc.customer_id = c.customer_id`. It needs to be a **correlated subquery**.
			- Main Query: You need to ensure a customer has **at least** 1 completed order first.
	- Clean Solution:
		```sql
		WITH payment_totals AS (
		    SELECT
		        p.order_id,
		        SUM(p.amount) AS total_payment
		    FROM payments p
		    WHERE EXISTS (
		        SELECT 1
		        FROM orders o
		        WHERE o.order_id = p.order_id
		          AND o.status = 'completed'
		    )
		    GROUP BY p.order_id
		),
		payment_coverage AS (
		    SELECT
		        o.order_id,
		        o.customer_id,
		        COALESCE(pt.total_payment, 0) * 1.0 / o.amount AS coverage
		    FROM orders o
		    LEFT JOIN payment_totals pt
		        ON o.order_id = pt.order_id
		    WHERE o.status = 'completed'
		)
		SELECT
		    c.customer_id,
		    c.customer_name
		FROM customers c
		WHERE EXISTS (
		    SELECT 1
		    FROM payment_coverage pc
		    WHERE pc.customer_id = c.customer_id
		)
		AND NOT EXISTS (
		    SELECT 1
		    FROM payment_coverage pc
		    WHERE pc.customer_id = c.customer_id
		      AND pc.coverage < 1
		)
		ORDER BY c.customer_id;
		```

# Problem-Solving Advice

## Logic Errors vs. Syntax Errors

- Syntax errors, such as forgetting to write `SELECT` or forgetting to use commas to separate columns in `GROUP BY` will likely be forgiven during an interview.
- Your bigger concern should be catching **logical errors**, such as:
	- wrong join key
	- uncorrelated `NOT EXISTS`
	- wrong ranking function
	- filtering the wrong quarter
	- accidentally excluding missing data with an inner join
	- comparing the wrong grain
	- filtering before instead of after aggregation
- Those are the errors that can produce a perfectly valid-looking query with the wrong answer.

## Interview Descision Tree

1. Where is my final grain: "One row per what?"
2. Do I need to collapse rows:
	- Yes: `GROUP BY`.
	- No: Possibly a window function.
3. Am I filtering individual rows: `WHERE`.
4. Am I filtering aggregated rows: `HAVING`.
5. Am I comparing categories / time periods: `CASE` + aggregate.
6. Do I need the previous / next row: `LAG()` / `LEAD()`.
7. Do ties matter:
	- Unique row: `ROW_NUMBER()`
	- Tied positions: `RANK()`
	- Nth distinct value: `DENSE_RANK()`
8. "Has at least one": `EXISTS`
9. "Has none": `NOT EXISTS`
10. "Every X satisfies Y": `NOT EXISTS` an X that violates Y.
11. Could the related table have no matching rows: `LEFT JOIN` + possibly `COALESCE`.
12. Am I joining different grains: Stop and check for **row multiplication**.

## Start With Grain

- Before writing SQL, ask: "What should one row in my final result represent?"
- For example:

| Question                          | Final grain              |
| --------------------------------- | ------------------------ |
| Total revenue per customer        | 1 row / customer         |
| Highest order per customer        | 1+ rows / customer       |
| Employee above department average | 1 row / employee         |
| Monthly sales                     | 1 row / employee / month |
| Payment coverage                  | 1 row / order            |
| Customer's average order amount   | 1 row / customer         |

- Once you know the final grain that is required, ask: "What grain do my source tables currently have?"
	- For example, if the `payments` table has many payments per order, but you only need one row per order, you need to aggregate payments **before** joining them to order-level data.

## GROUP BY vs. Window Function

- `GROUP BY`: Use it when you **want to collapse rows**.
	```sql
	SELECT
	    customer_id,
	    SUM(amount) AS total_sales
	FROM orders
	GROUP BY customer_id;
	```
- Window Function: Use it when you want to **calculate something across related rows while keeping the individual rows**. That's also how your advanced SQL notes describe the key distinction from `GROUP BY`.
	```sql
	AVG(amount) OVER (
	    PARTITION BY customer_id
	)
	```
- Window functions **cannot** be use used directly in a `WHERE` clause because they're evaluated after filtering.
	- Memorize the following pattern to avoid this pitfall:
		```
		calculate window function
		        ↓
		CTE/subquery
		        ↓
		filter window result
		```
- Mental Shortcut:
	- Need fewer rows: `GROUP BY`
	- Need the same rows, plus additional information: Window Function

## WHERE vs. HAVING

- `WHERE`: Filters **rows before aggregation**.
	```sql
	WHERE status = 'completed'
	```
- `HAVING`: Filter **groups after aggregation**.
	```sql
	HAVING COUNT(*) >= 2
	   AND AVG(amount) > 200
	```
- Mental Model:
	```
	FROM
	 ↓
	WHERE          ← filter rows
	 ↓
	GROUP BY       ← create groups
	 ↓
	HAVING         ← filter groups
	 ↓
	SELECT
	```

## Conditional Aggregation

- Whenever you here: "Compare X and Y", where X and Y are categories, periods, or statuses, immediately think:
	```sql
	SUM(
	    CASE
	        WHEN condition THEN amount
	        ELSE 0
	    END
	)
	```
- Example:
	```sql
	SUM(CASE
	    WHEN MONTH(order_date) BETWEEN 1 AND 3
	    THEN amount
	    ELSE 0
	END) AS q1_total
	```
- Common Question Signals:
	- Q1 vs Q2
	- 2025 vs 2026
	- Completed vs cancelled
	- Deposits vs withdrawals
	- Domestic vs international
	- Active vs inactive
- When a problem asks about a fixed number of known periods, conditional aggregation is often simpler than `LAG()`.

## Ranking Functions

- `ROW_NUMBER()`: Every row gets a unique number.
	- Useful when you want: "Give me the most recent row."
	- Example:
		```sql
		ROW_NUMBER() OVER (
		    PARTITION BY customer_id
		    ORDER BY order_date DESC
		)
		...
		WHERE row_num = 1
		```
	- If there is a tie, **only one row survives**.
- `RANK()`: Ties share ranks, and **ranks have gaps**.
	- Useful when you want: "Top positions, including everyone tied for first."
- `DENSE_RANK()`: Ties share ranks, but there are **no gaps**.
	- Useful when you want: Nth distinct value.

## EXISTS / NOT EXISTS

- `EXISTS`:
	- Think: "Has at least one..."
- `NOT EXISTS`:
	- Think: "Has none..."
	- Example:
		```sql
		NOT EXISTS (
		    SELECT 1
		    FROM orders o
		    WHERE o.customer_id = c.customer_id
		      AND o.status != 'completed'
		)
		```
		- Instead of checking if orders have a status of `completed`, this checks to make sure no order exists without a status of `completed`.
- When using **either** `EXISTS` or `NOT EXISTS`, **always make sure the subquery is correlated**.
	```sql
	WHERE NOT EXISTS (
	    SELECT 1
	    FROM orders o
	    WHERE o.amount <= 100
	)
	```
	- This subquery is **not** correlated and will check all orders against the `WHERE` condition, instead of just the order IDs that match customer IDs.
	- "There are no orders anywhere in the database ≤ $100."
	```sql
	WHERE NOT EXISTS (
	    SELECT 1
	    FROM orders o
	    WHERE o.customer_id = c.customer_id
	      AND o.amount <= 100
	)
	```
	- This subquery is correlated and will only check orders associated with a customer from the `customers` table.
	- "This customer has no orders ≤ $100."
- Don't use EXISTS / NOT EXISTS before aggregating. Use HAVING instead.

## LEFT JOIN vs. INNER JOIN

- `LEFT JOIN`: Preserves information for the left table, while marking information for the right table as `NULL` when it **does not meet the join condition**.
	- Use when: "Unrelated records should still count."
	- `COALESCE` is typically used with `LEFT JOIN` to appropriately handle null values.
- `INNER JOIN`: Only preserves information from **both** tables which match the join condition.
	- Use when: "Unrelated records should not count."
	- `COALESCE` is typically not needed since only matching records from both tables are preserved.

## Aggregate Before Joining

- If `orders` has one row / order and `payments` has many rows per order, payment information should be aggregated based on `order_id` before joing with `orders` to **prevent row multiplication**.
	- Always think about the one-to-one vs. one-to-many or many-to-many relationship before joining tables.
	- Ideally, tables should only be joined when they have a one-to-one relationship. Use aggregation to ensure this.

## Using CTEs

- CTEs should primarily be used when you're creating an intermediate table at a specific grain.
- For example:
	```
	CTE 1:
	one row / order
	
	CTE 2:
	one row / customer
	
	CTE 3:
	rank customers
	
	Final:
	filter results
	```
- Typically, if you only need one CTE, it can be written as a subquery instead.

## Watch for Date Boundries

- When a problem says Q1 of 2026:
	- `MONTH(order_date) BETWEEN 1 AND 3` is **not enough**
	- You also need: `WHERE order_date >= '2026-01-01' AND order_date < '2026-04-01'`.

## Tie Breaking

- When a problem says: "If dates are tied, use the larger order ID."
	- You can't just use: `ORDER BY order_date DESC;`
	- You need: `ORDER BY order_date DESC order_id DESC;`. You need more than one condition to break the tie.