---
aliases:
  - "python_faang_problems"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Python and Pandas/python_faang_problems.md"
imported: 2026-09-24
---

# Pandas Interview Practice

Review: [[02_technical_prep/python/Python Data Engineering Toolkit|Python Data Engineering Toolkit]] · [[02_technical_prep/python/Python Algorithms Practice|Python Algorithms Practice]]

## Facebook

### Reactions at Content Type
Using the two DataFrames `posts` and `reactions`, write a Pandas query to determine the total number of reactions each `content_type` has received.
- Join the two DataFrames appropriately.
- Count how many reactions are associated with each `content_type`.
- Order the result by the number of reactions in descending order.
- Your final output should include: `content_type` and `reaction_count`.

Answer: 
```python
# Using posts and reactions data frames, write a query to determine:
# The number of reactions for each content type.

merged = pd.merge(posts, reactions, on = 'post_id', how = 'inner')
content_groups = (
    merged.groupby('content_type')
        .agg(reaction_count=('reaction_id', 'count'))
        .reset_index()
        .sort_values(by = 'reaction_count', ascending = False)
)
print(content_groups)
```
- The `agg` method that comes after `groupby` creates a new column that stores the aggregated results.
- The `reset_index()` method resets the index from `content_type` (automatically changed after performing the grouping) to the standard numerical index.
- The `sort_values` method is similar to `ORDER BY` in SQL.

### User With Top Reactions
Using Pandas, write code to identify the user who has given the most reactions to posts with the content type 'video'.

DataFrames:
- `posts`: contains columns `post_id`, `user_id`, `content`, `content_type`, `post_date`
- `reactions`: contains columns `reaction_id`, `post_id`, `user_id`, `reaction_type`, `reaction_date`

Return the user_id of the top engager (most reactions to video posts).

Answer:
```python
# Find the user who has given the mosts reactions to 'video' posts.
# data frames: posts, reactions

# Step 1: Filter posts to only 'Video' content
video_posts = posts[posts["content_type"] == "Video"]

# Step 2: Join posts and reactions on post_id
merged_df = pd.merge(video_posts, reactions, on="post_id")

# Step 3: Group by reacting user (reactions.user_id), count reaction_id
reaction_counts = (
    merged_df.groupby("user_id_y")
    .agg(reaction_count = ('reaction_id', 'count'))
    .reset_index()
    .sort_values(by="reaction_count", ascending=False)
    .head(1)
)

print(reaction_counts)
```
- When two data frames are merged in pandas, columns that are shared between the two, but not used to merge them, receive a suffix of `_x` and `_y`. This is seen in `user_id_y` in the `groupby` method.
- The `sort_values` and `head` methods act similar to an `ORDER BY` and `LIMIT` in SQL.

### Reaction Time
Using Pandas, write code to compute the average time (in minutes) a user takes to react to a post after it has been posted.

DataFrames:
- `posts`: contains columns `post_id`, `user_id`, `content`, `content_type`, `post_date`
- `reactions`: contains columns `reaction_id`, `post_id`, `user_id`, `reaction_type`, `reaction_date`

Answer:
```python
# Compute the average time in minutes a user takes to react to a post.

merged = pd.merge(posts, reactions, on = 'post_id', how = 'inner')
merged = merged[merged['reaction_date'] > merged['post_date']]
merged['reaction_date'] = pd.to_datetime(merged['reaction_date'])
merged['post_date'] = pd.to_datetime(merged['post_date'])
merged['reaction_time_minutes'] = (merged['reaction_date'] - merged['post_date']).dt.seconds / 60

grouped = (
    merged.groupby('user_id_y')
        .agg(average_reaction_time = ('reaction_time_minutes', 'mean'))
        .reset_index()
)

print(grouped)
```

### Most Engaging Content Types
Using Pandas, write code to identify which type of content received the most reactions (likes and comments combined) during the first six months of 2023, based on their reaction date.

DataFrames:
- `posts`: contains columns `post_id`, `user_id`, `content`, `content_type`, `post_date`
- `reactions`: contains columns `reaction_id`, `post_id`, `user_id`, `reaction_type`, `reaction_date` 

Answer:
```python
# Identify the content type that received the most reactions during the
# first 6 months of 2023 based on reaction date.

reactions['reaction_date'] = pd.to_datetime(reactions['reaction_date'])
filtered_reactions = reactions[reactions['reaction_date'].between('2023-01-01', '2023-06-30')]

merged = pd.merge(posts, filtered_reactions, on = 'post_id', how = 'inner')
output = (
    merged.groupby('content_type')
        .agg(reaction_count = ('reaction_id', 'count'))
        .reset_index()
        .sort_values(by = 'reaction_count', ascending = False)
        .head(1)
)

print(output)
```

### Campaign Expenditure
Using Pandas, write code to determine the total expenditure for each campaign. The result should include the campaign ID, campaign name, and total expenditure, and should be ordered by total expenditure in descending order.

DataFrames:
- `campaigns`: contains columns `campaign_id`, `campaign_name`, `start_date`, `end_date`
- `adsets`: contains columns `adset_id`, `campaign_id`, `adset_name`, `budget`
- `ads`: contains columns `ad_id`, `adset_id`, `ad_name`, `impressions`, `clicks`, `spent`

Answer:
```python
# Find the total expenditure for each campaign
# Include: campaign_id, campaign_name, total_expenditure (descending order)

merged = (
    pd.merge(
        pd.merge(campaigns, adsets, on = 'campaign_id', how = 'inner'),
        ads, on = 'adset_id', how = 'inner'
    )
)
output = (
    merged.groupby(['campaign_id', 'campaign_name'])
        .agg(total_expenditure = ('spent', 'sum'))
        .reset_index()
        .sort_values(by = 'total_expenditure', ascending = False)
)

print(output)
```
- Here, we see how to merge more than two data frames at once by merging a merged data frame with a third data frame.

### User Engagement Frequency
Using Pandas, write code to find which users engaged (either liked or commented) on more than 2 different posts in the first six months of 2023 with respect to their post date. Display the `user_id` and `post_count` (number of posts).

DataFrames:
- `posts`: contains columns `post_id`, `user_id`, `content`, `content_type`, `post_date`
- `reactions`: contains columns `reaction_id`, `post_id`, `user_id`, `reaction_type`, `reaction_date`

Answer:
```python
# Find users who engaged with a post more than twice in the first six months
# of 2023.
# Display: user_id, post_count

posts['post_date'] = pd.to_datetime(posts['post_date'])
filtered_posts = posts[posts['post_date'].between('2023-01-01', '2023-06-30')]
merged = pd.merge(filtered_posts, reactions, on = 'post_id', how = 'inner')

post_counts = (
    merged.groupby('user_id_y')['post_id']
        .count()
        .reset_index(name = 'post_count')
)
output = post_counts[post_counts['post_count'] > 2]
output.columns = ['user_id', 'post_count']

print(output)
```
- Another way of re-defining the aggregate column after grouping (besides using the `agg` function as in the above examples).

### Top 3 Most Reacted Posts
Write code to find the top 3 posts with the highest number of reactions from the first 3 months of 2023 with respect to the reaction date. In case of a tie, use the earliest post date as a tie-breaker. Provide their post IDs and total reaction counts.

DataFrames:
- `posts`: contains columns `post_id`, `user_id`, `content`, `content_type`, `post_date`
- `reactions`: contains columns `reaction_id`, `post_id`, `user_id`, `reaction_type`, `reaction_date`

Answer:
```python
# Find the top 3 posts wrt number of reactions during the first 3
# months of 2023

reactions['reaction_date'] = pd.to_datetime(reactions['reaction_date'])
filtered_reactions = reactions[reactions['reaction_date'].dt.month <= 3]
post_reaction_counts = (
    filtered_reactions.groupby('post_id')['reaction_id']
        .count()
        .reset_index(name = 'total_reactions')
)

output = (
    pd.merge(
        post_reaction_counts,
        posts[['post_id', 'post_date']],
        on = 'post_id',
        how = 'left'
    )
    .sort_values(
        by = ['total_reactions', 'post_date'],
        ascending = [False, True]
    )
    .head(3)
)

print(output[['post_id', 'total_reactions']])
```

### Top Performing Ad Sets
Write code for each campaign to identify the ad set with the highest Click Through Rate (CTR). The CTR is calculated as:
```python
CTR=(Total Impressions/Total Clicks​)×100%​
Recognizing top-performing ad sets is crucial to replicate their success patterns in future campaigns.
```

DataFrames:
- `campaigns`: contains columns `campaign_id`, `campaign_name`, `start_date`, `end_date`
- `adsets`: contains columns `adset_id`, `campaign_id`, `adset_name`, `budget`
- `ads`: contains columns `ad_id`, `adset_id`, `ad_name`, `impressions`, `clicks`, `spent`

Answer:
```python
# Identify the ad set with highest CTR for each campaign
# CR = (total_impressions / total_clicks) * 100
# Display: campaign_name, adset_name, click_through_rate

merged_df = pd.merge(ads, adsets, on = 'adset_id', how = 'inner')

adset_performance = (
    merged_df.groupby(['campaign_id', 'adset_name'], as_index = False)
        .agg(
            total_impressions = ('impressions', 'sum'),
            total_clicks = ('clicks', sum)
        )
)
adset_performance['click_through_rate'] = (
    adset_performance['total_clicks'] /
    adset_performance['total_impressions']
)
adset_performance['rnk'] = (
    adset_performance
        .groupby('campaign_id')['click_through_rate']
        .rank(method = 'first', ascending = False)
)
top_adsets = adset_performance[adset_performance['rnk'] == 1]

output = (
    pd.merge(
        top_adsets,
        campaigns[['campaign_id', 'campaign_name']],
        on='campaign_id', how='inner'
    )
)
final_output = (
    output[['campaign_name', 'adset_name', 'click_through_rate']]
        .sort_values(
            by = ['campaign_name', 'click_through_rate'],
            ascending = [True, False]
        )
)
print(final_output)
```
- Ranking in pandas is performed using the `rank` method after `groupby`. The `groupby` along with the specified column act similarly to `PARTITION BY` in SQL. The column chosen after `groupby` (in the square brackets) along with the `ascending` argument of `rank` act similarly to `ORDER BY` in SQL.
- The `method` argument of the `rank` function compares to SQL as follows:
  - `min` acts similarly to `RANK`, where all ties receive the minimum rank (2 for ties between 2, 3, 4, etc.), skipping ranks as appropriate.
  - `dense` acts similarly to `DENSE_RANK`, where all ties receive the minimum rank and ranks are not skipped.
  - `first` acts similarly to `ROW_NUMBER`, where all rows receive a unique rank and ties are not considered.

## Apple

### Highest Product Sales
Write a query to identify the month in 2023 during which the highest number of product units were sold.

DataFrames:
- `sales`: contains columns `sale_id`, `product_id`, `store_id`, `customer_id`, `unit_price`, `unit_sold`, `sale_date`

Answer:
```python
# Identify the month in 2023 with the highest product unit sales.

sales['sale_date'] = pd.to_datetime(sales['sale_date'])
sales_2023 = sales[sales['sale_date'].dt.year == 2023].copy()
sales_2023['month_of_2023'] = sales_2023['sale_date'].dt.month_name()
output = (
    sales_2023.groupby('month_of_2023')
        .agg(total_units_sold = ('units_sold', 'sum'))
        .reset_index()
        .sort_values(by = 'total_units_sold', ascending = False)
        .head(1)
)
print(output)
```
- The `copy()` method is used here because we can't add columns to a data frame directly after filtering it.

### Average Price Difference
Find the average price difference (selling price minus unit price) for each product. The result should include:
- `product_name`
- `price`
- `average_price_difference`
The average price difference across all sales of that product.

DataFrames:
- `sales`: contains columns `sale_id`, `product_id`, `store_id`, `customer_id`, `unit_price`, `unit_sold`, `sale_date`
- `product`: contains columns `product_id`, `product_name`, `category`, `launch_date`, `price`

Answer:
```python
# Find the average price difference for each product
# Display: product_name, price, average_price_difference

merged = pd.merge(product, sales, on = 'product_id', how = 'inner')
merged['price_difference'] = merged['price'] - merged['unit_price']
output = (
    merged.groupby(['product_id', 'product_name', 'price'])
        .agg(average_price_difference = ('price_difference', 'mean'))
        .reset_index()
        .sort_values(by = 'average_price_difference', ascending = False)
)

print(output)
```

### Product Launch Impact
Write a query to determine the top 3 products launched in 2023, based on their sales in the first two months after their launch. Display product_name, launch_date sales_revenue as the column names for name, launch date and revenue of the product.

DataFrames:
- `sales`: contains columns `sale_id`, `product_id`, `store_id`, `customer_id`, `unit_price`, `unit_sold`, `sale_date`
- `product`: contains columns `product_id`, `product_name`, `category`, `launch_date`, `price`

Answer:
```python
# Determine the top 3 products launched in 2023 based on the first two
# months of sales after their respective launches.
#Display: product_name, launch_date, sales_revenue

sales['sale_date'] = pd.to_datetime(sales['sale_date'])
product['launch_date'] = pd.to_datetime(product['launch_date'])

merged_df = pd.merge(product, sales, on = 'product_id', how = 'inner')
merged_df = merged_df[merged_df['launch_date'].dt.year == 2023]
merged_df = (
    merged_df[
        (merged_df['sale_date'] >= merged_df['launch_date']) &
        (merged_df['sale_date'] <= merged_df['launch_date'] + pd.DateOffset(months = 2))
    ]
)
merged_df['total_price'] = merged_df['unit_price'] * merged_df['units_sold']

output = (
    merged_df.groupby(['product_id','product_name', 'launch_date'])
        .agg(sales_revenue = ('total_price', 'sum'))
        .reset_index()
        .sort_values(by = 'sales_revenue', ascending = False)
        .head(3)
)

print(output)
```
- Here, instead of adding another column for the date two months after the launch date, we just filtered sales directly from the merged data frames.

### New Customer Retentions
Write a query to count the number of customers who registered in each year.
The output should include:
- registration_year
- customer_count

DataFrames:
- `customers_apple`: contains columns `customer_id`, `customer_name`, `email`, `phone`, `registration_date`

Answer:
```python
# Count the number of customer registrations per year
# Display: registration_year, customer_count

customers_apple['registration_date'] = pd.to_datetime(customers_apple['registration_date'])
customers_apple['registration_year'] = customers_apple['registration_date'].dt.year

output = (
    customers_apple.groupby('registration_year')
        .agg(customer_count = ('customer_id', 'count'))
        .reset_index()
)
print(output)
```

### Highest Sales Revenue
Write a query to determine which store has generated the highest sales revenue in 2023.

DataFrames:
- `sales`: contains columns `sale_id`, `product_id`, `store_id`, `customer_id`, `unit_price`, `unit_sold`, `sale_date`
- `stores`: contains columns `store_id`, `store_name`, `location_city`, `location_country`, `open_date`

Answer:
```python
# Find the store that generated the highest sales revenue in 2023

sales['sale_date'] = pd.to_datetime(sales['sale_date'])
sales_2023 = sales[sales['sale_date'].dt.year == 2023].copy()
sales_2023['total_amount'] = sales_2023['unit_price'] * sales_2023['units_sold']

merged = pd.merge(sales_2023, stores, on = 'store_id', how = 'left')

output = (
    merged.groupby('store_name')
        .agg(total_revenue = ('total_amount', 'sum'))
        .reset_index()
        .sort_values(by = 'total_revenue', ascending = False)
        .head(1)
)
print(output)
```

### Best Selling Products
Identify the Top 5 best-selling products for the year 2023 based on the quantity sold. Provide the product name, category, total quantity sold, and total revenue generated by each of these products.

DataFrames:
- `sales`: contains columns `sale_id`, `product_id`, `store_id`, `customer_id`, `unit_price`, `unit_sold`, `sale_date`
- `product`: contains columns `product_id`, `product_name`, `category`, `launch_date`, `price`

Answer:
```python
# Find the top 5 best-selling products in 2023 based on quantity sold.
# Display: product_name, category, total_quantity_sold, revenue_generated

sales['sale_date'] = pd.to_datetime(sales['sale_date'])
product['launch_date'] = pd.to_datetime(product['launch_date'])

sales_2023 = sales[sales['sale_date'].dt.year == 2023].copy()
sales_2023['total_amount'] = sales_2023['unit_price'] * sales_2023['units_sold']
product_sales = pd.merge(sales_2023, product, on = 'product_id', how = 'inner')

top_five_best_selling_products = (
    product_sales.groupby(['product_name', 'category'], as_index = False)
        .agg(
            total_quantity_sold = ('units_sold', 'sum'),
            total_revenue = ('total_amount', 'sum')
        )
        .sort_values(by = 'total_quantity_sold', ascending = False)
        .head(5)
)

print(top_five_best_selling_products)
```

### Most Popular Registration Month Per Year
Find the month with the highest number of registrations per year.
The output should include:
- registration_year
- registration_month (formatted as `month_name()`)
- total_registrations
If multiple months have the same highest count for a year, return only one using LIMIT 1 per year.

DataFrames:
- `customers_apple`: contains columns `customer_id`, `customer_name`, `email`, `phone`, `registration_date`

Answer:
```python
# Find the month with the highest number of registrations per year.
# Display registration_year, registration_month, total_registrations

customers_apple['registration_date'] = pd.to_datetime(customers_apple['registration_date'])
customers_apple['registration_year'] = customers_apple['registration_date'].dt.year
customers_apple['registration_month'] = customers_apple['registration_date'].dt.month_name()

total_monthly_registrations = (
    customers_apple.groupby(['registration_year', 'registration_month'], as_index = False)
        .agg(total_registrations = ('customer_id', 'count'))
)

total_monthly_registrations['rnk'] = (
    total_monthly_registrations
        .groupby('registration_year')['total_registrations']
        .rank(method = 'first', ascending = False)
)

best_months = total_monthly_registrations[total_monthly_registrations['rnk'] == 1]
print(best_months.drop(columns = 'rnk'))
```
- setting `as_index = False` in `groupby` helps to avoid needing to `reset_index()` afterwards.
