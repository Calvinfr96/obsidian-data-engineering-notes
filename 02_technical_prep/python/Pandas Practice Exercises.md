---
aliases:
  - "pandas_practice_questions.py"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Python and Pandas/pandas_practice_questions.py"
imported: 2026-09-24
---

# Pandas Practice Exercises

Review: [[02_technical_prep/python/Python Data Engineering Toolkit|Python Data Engineering Toolkit]]

## Contents

- [[#Setup]]
- [[#1. Filter Rows]]
- [[#2. Grouping and Aggregation]]
- [[#3. Sorting]]
- [[#4. Filtering and Aggregation]]
- [[#5. Creating a New Column]]
- [[#6. Find The Maximum]]
- [[#7. Count Occurrences]]
- [[#8. Merging Two DataFrames]]
- [[#9. Dropping Rows]]
- [[#10. Apply Function]]
- [[#11. Pivot Table]]
- [[#12. Multi-Level Grouping]]
- [[#13. Reshaping With Melt]]
- [[#14. Advanced Filtering]]
- [[#15. Adding Rolling Average]]
- [[#16. Ranking]]
- [[#17. Percentage of Total]]
- [[#18. Filtering Based on String Conditions]]
- [[#19. Cumulating Sum]]
- [[#20. Custom Aggregation]]
- [[#21. Extracting Date Information]]
- [[#22. Merge With Condition]]
- [[#23. Index Manipulation]]
- [[#24. Apply Custom Function]]
- [[#25. Resampling Time Series]]
- [[#26. Filtering With Multiple Conditions]]
- [[#27. Multi-Indexing]]
- [[#28. Handling Missing Data]]
- [[#29. Duplicates Handling]]

## Setup

```python
import pandas as pd
```

## 1. Filter Rows

```python
# From the dataset below, filter all rows where City is "NY" and Discount_Applied is "Yes".
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Age": [24, 45, 31, 28],
    "Gender": ["Male", "Female", "Male", "Female"],
    "Purchase_Amount_USD": [253, 364, 540, 120],
    "Discount_Applied": ["Yes", "No", "Yes", "No"],
    "City": ["NY", "SF", "NY", "Chicago"]
}

df = pd.DataFrame(data)
df2 = df.query("City == 'NY' & Discount_Applied == 'Yes'")
print(df2)
```

## 2. Grouping and Aggregation

```python
# Using the data below, calculate the average Purchase_Amount_USD for each City.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "City": ["NY", "SF", "LA", "NY"],
    "Purchase_Amount_USD": [253, 364, 540, 120]
}

df = pd.DataFrame(data)
average = df.groupby('City').mean()
print(average['Purchase_Amount_USD'])
```

## 3. Sorting

```python
# Sort the dataset below by Review_Rating in descending order.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 364, 540, 120],
    "Review_Rating": [4.2, 3.7, 4.8, 3.2]
}

df = pd.DataFrame(data)
sorted_df = df.sort_values(by = 'Review_Rating', ascending = False)
print(sorted_df)
```

## 4. Filtering and Aggregation

```python
# How many customers applied a discount in "Electronics"?
data = {
    "Customer_ID": [1, 2, 3, 4, 5],
    "Category": ["Electronics", "Furniture", "Electronics", "Clothing", "Electronics"],
    "Discount_Applied": ["Yes", "No", "Yes", "Yes", "No"]
}

df = pd.DataFrame(data)
filtered_df = df.query("Category == 'Electronics' and Discount_Applied == 'Yes'")
print(filtered_df['Customer_ID'].count())
```

## 5. Creating a New Column

```python
# Create a new column called Age_Group that categorizes ages into <25, 25-35, and 36+.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Age": [24, 45, 31, 28]
}

df = pd.DataFrame(data)
df["Age_Group"] = pd.cut(
    df["Age"],
    bins = [0, 25, 35, float('inf')],
    labels  = ['<25', '25-35', '36+'],
    right = False
)
print(df)
```

## 6. Find The Maximum

```python
# Find the category with the highest total purchase amount.
data = {
    "Category": ["Electronics", "Furniture", "Clothing", "Electronics", "Clothing"],
    "Purchase_Amount_USD": [253, 364, 540, 120, 200]
}

df = pd.DataFrame(data)
category_revenue = df.groupby('Category').sum()
print(category_revenue.idxmax())
```

## 7. Count Occurrences

```python
# Count the number of purchases made by each city.
data = {
    "Customer_ID": [1, 2, 3, 4, 5],
    "City": ["NY", "SF", "LA", "NY", "SF"]
}

df = pd.DataFrame(data)
city_purchases = df['City'].value_counts()
print(city_purchases)
```

## 8. Merging Two DataFrames

```python
# Merge the two datasets below on Customer_ID.
data1 = {
    "Customer_ID": [1, 2, 3, 4],
    "Age": [24, 45, 31, 28]
}
data2 = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 364, 540, 120]
}

df1 = pd.DataFrame(data1)
df2 = pd.DataFrame(data2)
merged = pd.merge(df1, df2, on = 'Customer_ID', how = 'inner')
print(merged)
```

## 9. Dropping Rows

```python
# Remove all rows where Review_Rating is below 3.0.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Review_Rating": [4.2, 2.8, 3.7, 3.2]
}

df = pd.DataFrame(data)
filtered_df = df.query("Review_Rating >= 3.0")
print(filtered_df)
```

## 10. Apply Function

```python
# Add a new column called Discount_Flag that is 1 if Discount_Applied is "Yes" and 0 otherwise.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Discount_Applied": ["Yes", "No", "Yes", "No"]
}

df = pd.DataFrame(data)

def apply_discount(row):
  if row["Discount_Applied"] == "Yes":
    return 1
  else:
    return 0

df["Discount_Flag"] = df.apply(apply_discount, axis = 1)
print(df)
```

## 11. Pivot Table

```python
# Create a pivot table that shows the average Purchase_Amount_USD for each Category and City.
data = {
    "Customer_ID": [1, 2, 3, 4, 5],
    "Category": ["Electronics", "Furniture", "Electronics", "Clothing", "Furniture"],
    "City": ["NY", "SF", "NY", "LA", "SF"],
    "Purchase_Amount_USD": [253, 364, 540, 120, 200]
}

df = pd.DataFrame(data)
pivot_df = pd.pivot_table(
    df,
    values = "Purchase_Amount_USD",
    index = "Category",
    columns = "City",
    aggfunc = "mean"
)
print(pivot_df)
```

## 12. Multi-Level Grouping

```python
# Group by Category and City, and calculate both the total and average Purchase_Amount_USD for each group.
data = {
    "Category": ["Electronics", "Furniture", "Electronics", "Clothing", "Furniture"],
    "City": ["NY", "SF", "NY", "LA", "SF"],
    "Purchase_Amount_USD": [253, 364, 540, 120, 200]
}

df = pd.DataFrame(data)
grouped_df = df.groupby(['Category', 'City']).agg(['sum', 'mean'])
print(grouped_df)
```

## 13. Reshaping With Melt

```python
# Reshape the dataset below using the melt function to create a long format where each row represents a City and its Purchase_Amount_USD.
data = {
    "Customer_ID": [1, 2, 3],
    "City_NY": [253, 364, 540],
    "City_SF": [120, 300, 250]
}

df = pd.DataFrame(data)
melted_df = pd.melt(
    df,
    id_vars = "Customer_ID",
    value_vars = ["City_NY", "City_SF"],
    var_name = "City",
    value_name = "Purchase_Amount_USD"
)
print(melted_df)
```

## 14. Advanced Filtering

```python
# Filter customers who have a Purchase_Amount_USD greater than 300 and a Review_Rating of 4.0 or higher.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 400, 540, 120],
    "Review_Rating": [4.2, 3.8, 4.8, 4.0]
}

df = pd.DataFrame(data)
filtered_df = df.query("Purchase_Amount_USD > 300 and Review_Rating >= 4")
print(filtered_df)
```

## 15. Adding Rolling Average

```python
# Add a new column called Rolling_Avg that calculates the 3-period rolling average of Purchase_Amount_USD.
data = {
    "Customer_ID": [1, 2, 3, 4, 5],
    "Purchase_Amount_USD": [253, 364, 540, 120, 450]
}

df = pd.DataFrame(data)
df["Rolling_Avg"] = df["Purchase_Amount_USD"].rolling(window = 3).mean()
print(df)
```

## 16. Ranking

```python
# Rank the customers based on Purchase_Amount_USD in descending order.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 400, 540, 120]
}

df = pd.DataFrame(data)
df["Rank"] = df['Purchase_Amount_USD'].rank(ascending = False, method = 'dense')
print(df)
```

## 17. Percentage of Total

```python
# Calculate the percentage of the total Purchase_Amount_USD for each row.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 400, 540, 120]
}

df = pd.DataFrame(data)
df["Percentage_of_Total"] = (df["Purchase_Amount_USD"] / df["Purchase_Amount_USD"].sum()) * 100
print(df)
```

## 18. Filtering Based on String Conditions

```python
# Filter all rows where the City starts with "S".
data = {
    "Customer_ID": [1, 2, 3, 4],
    "City": ["NY", "SF", "LA", "Seattle"]
}

df = pd.DataFrame(data)
filtered = df[df['City'].str.startswith('S')]
print(filtered)
```

## 19. Cumulating Sum

```python
# Add a new column called Cumulative_Sum that calculates the cumulative sum of Purchase_Amount_USD.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 400, 540, 120]
}

df = pd.DataFrame(data)
df["Cumulative_Sum"] = df['Purchase_Amount_USD'].cumsum()
print(df)
```

## 20. Custom Aggregation

```python
# For each City, calculate the minimum, maximum, and mean Purchase_Amount_USD.
data = {
    "City": ["NY", "SF", "NY", "LA", "SF"],
    "Purchase_Amount_USD": [253, 364, 540, 120, 450]
}

df = pd.DataFrame(data)
output = df.groupby('City').agg(['min', 'max', 'mean'])
print(output)
```

## 21. Extracting Date Information

```python
# Extract the year and month from the Purchase_Date column and create two new columns: Year and Month.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Date": ["2023-01-15", "2022-05-20", "2023-07-11", "2022-12-01"]
}

df = pd.DataFrame(data)
df["Purchase_Date"] = pd.to_datetime(df["Purchase_Date"])
df["Year"] = df["Purchase_Date"].dt.year
df["Month"] = df["Purchase_Date"].dt.month
print(df)
```

## 22. Merge With Condition

```python
# Merge two datasets based on Customer_ID but only include rows where Purchase_Amount_USD is greater than 300.
data1 = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 400, 540, 120]
}
data2 = {
    "Customer_ID": [1, 2, 3, 4],
    "City": ["NY", "SF", "LA", "Chicago"]
}


df1 = pd.DataFrame(data1)
df2 = pd.DataFrame(data2)
merged = pd.merge(df1[df1['Purchase_Amount_USD'] > 300], df2, on = "Customer_ID", how = "inner")
print(merged)
```

## 23. Index Manipulation

```python
# Set the Customer_ID column as the index, and then reset it back to a regular column.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "City": ["NY", "SF", "LA", "Chicago"]
}

df = pd.DataFrame(data)
df.set_index('Customer_ID', inplace = True)
df.reset_index(drop = False, inplace = True) # We don't want to get rid of the Customer_ID column after resetting the index.
print(df)
```

## 24. Apply Custom Function

```python
# Create a new column called High_Spender that is True if Purchase_Amount_USD is greater than 500, else False.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Purchase_Amount_USD": [253, 400, 540, 120]
}

def is_high_spender(row):
  if row["Purchase_Amount_USD"] > 500:
    return True
  else:
    return False

df = pd.DataFrame(data)
df["High_Spender"] = df.apply(is_high_spender, axis = 1)
print(df)
```

## 25. Resampling Time Series

```python
# Resample the data to calculate the monthly total of Purchase_Amount_USD.
data = {
    "Purchase_Date": ["2023-01-15", "2023-01-20", "2023-02-10", "2023-03-01"],
    "Purchase_Amount_USD": [253, 400, 540, 120]
}

df = pd.DataFrame(data)
df['Purchase_Date'] = pd.to_datetime(df['Purchase_Date'])
df.set_index('Purchase_Date', inplace = True)
resampled = df.resample('ME').sum() # 'M' is being deprecated and 'ME' should be used instead.
print(resampled)
```

## 26. Filtering With Multiple Conditions

```python
# Filter all rows where Category is either "Electronics" or "Furniture" and City is not "NY".
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Category": ["Electronics", "Furniture", "Clothing", "Electronics"],
    "City": ["NY", "SF", "LA", "Chicago"]
}

df = pd.DataFrame(data)
output = df[(df["Category"].isin(["Electronics", "Furniture"])) & (df["City"] != "NY")]
print(output)
```

## 27. Multi-Indexing

```python
# Create a multi-index using City and Category and calculate the sum of Purchase_Amount_USD for each combination.
data = {
    "City": ["NY", "SF", "NY", "LA"],
    "Category": ["Electronics", "Furniture", "Clothing", "Electronics"],
    "Purchase_Amount_USD": [253, 364, 540, 120]
}

df = pd.DataFrame(data)
output = df.groupby(["City", "Category"]).sum()
print(output)
```

## 28. Handling Missing Data

```python
# Fill missing values in the Review_Rating column with the average rating.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Review_Rating": [4.2, None, 3.7, None]
}

df = pd.DataFrame(data)
df["Review_Rating"] = df["Review_Rating"].fillna(df["Review_Rating"].mean())
print(df)
```

## 29. Duplicates Handling

```python
# Remove duplicate rows based on the Category column, keeping only the first occurrence.
data = {
    "Customer_ID": [1, 2, 3, 4],
    "Category": ["Electronics", "Furniture", "Electronics", "Clothing"]
}

df = pd.DataFrame(data)
df.drop_duplicates(subset = "Category", keep = "first", inplace = True)
print(df)
```
