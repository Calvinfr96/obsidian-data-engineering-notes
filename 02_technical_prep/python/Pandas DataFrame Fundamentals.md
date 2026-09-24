---
aliases:
  - "pandas_dataframe_fundamentals"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Python and Pandas/pandas_dataframe_fundamentals.md"
imported: 2026-09-24
---

# Pandas DataFrame Fundamentals

Review: [[02_technical_prep/python/Python Data Engineering Toolkit|Python Data Engineering Toolkit]] · [[02_technical_prep/python/Python Algorithms Practice|Python Algorithms Practice]]

## Contents

- [[#Introduction to Pandas Library]]
- [[#Series Creation]]
- [[#DataFrame Creation]]
- [[#Functions]]
- [[#Grouping and Aggregation]]
- [[#Merging DataFrames]]
- [[#Pivot Tables]]
- [[#Date and Time Handling]]
- [[#Resampling and Frequency]]
- [[#Time-Based Indexing]]
- [[#Moving (Rolling) Averages]]
- [[#Shifting and Lagging]]
- [[#Helpful Functions]]

## Introduction to Pandas Library
- Pandas is a popular python library used for working with structured data and provides easy-to-use tools for data manipulation and analysis.
- Pandas is typically used for:
  - Handling large datasets efficiently.
  - Cleaning and transforming data.
  - Analyzing and summarizing data.
  - Working with structured and unstructured data.
- Pandas can be installed using the following command line prompt:
  ```bash
  pip install pandas
  ```
- Pandas can be imported in order to be used in your code as follows:
  ```python
  import pandas as pd (alias optional)
  ```
- Pandas makes it easy to perform the following tasks:
  - Load data from csv files and databases.
  - Handle missing values.
  - Filter and transform data.
  - Group and summarize data.
- Example Code: Using Pandas to Create a DataFrame From a Dictionary:
  ```python
  import pandas as pd

  data = {
      'Name': ['Alice', 'Bob', 'Charlie'],
      'Age': [25, 30, 35],
      'City': ['New York', 'San Francisco', 'Los Angeles']
  }

  # Create DataFrame from Dictionary
  df = pd.DataFrame(data)

  # Display the DataFrame
  print(df)
  ```

## Series Creation
- A series in pandas is a one-dimensional, labeled array that can hold data of any type, including integers, floats, strings, and objects. Each element in a series has a value and an index.
- A series in pandas is created from a list as follows:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  index = ['a', 'b', 'c', 'd']
  series = pd.Series(data, index=index)
  print(series)

  # Output
  a    10
  b    20
  c    30
  d    40
  dtype: int64
  ```
  - The first parameter is the data that will be stored in the series and is required.
  - The second parameter is the label for each data element and is optional. The default index is a zero-based, numerical index.
- A series can also be created from a dictionary:
  ```python
  import pandas as pd

  data = {'Alice': 25, 'Bob': 30, 'Charlie': 35}
  series = pd.Series(data)
  print(series)

  # Output
  Alice      25
  Bob        30
  Charlie    35
  dtype: int64
  ```
- When a series is made from a scalar value, all indexes are assigned the scalar value:
  ```python
  import pandas as pd

  value = 5
  index = ['a', 'b', 'c']
  series = pd.Series(value, index=index)
  print(series)

  # Output
  a    5
  b    5
  c    5
  dtype: int64
  ```
- Accessing series properties:
  - `series.index` returns the index of the series.
  - `series.values` returns the values of the series.
  - `series.dtype` returns the data type of the series.

### Accessing Elements in a Series
- Pandas allows you to access elements in a series using either index positions (similar to python indexing) and index labels (custom labels assigned during series creation). For example:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  series = pd.Series(data)

  # Access first element
  print(series[0]) # Outputs 10

  # Access second element
  print(series[1]) # Outputs 20
  ```
  ```python
  import pandas as pd

  data = [100, 200, 300, 400]
  index = ['a', 'b', 'c', 'd']
  series = pd.Series(data, index=index)

  # Access element with label 'b'
  print(series['b']) # Outputs 200

  # Access element with label 'd'
  print(series['d']) # Outputs 400

  # Access last element
  print(series[-1]) # Will fail since this only works with numerical indexing. Custom indexing was assigned in this case.
  ```
- You can access multiple elements within a series using a list of indexes. For example:
  ```python
  import pandas as pd

  data = [100, 200, 300, 400]
  index = ['a', 'b', 'c', 'd']
  series = pd.Series(data, index=index)

  # Access elements with labels 'a' and 'c'
  print(series[['a', 'c']]) # Note the double brackets. You need to pass the indexes as one list, not multiple arguments.

  # Output
  a    100
  c    300
  dtype: int64
  ```
- A portion of a series can be extracted using the `[start:stop]` syntax similar to python. For example:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40, 50]
  series = pd.Series(data)

  # Slice from index 1 to 3
  print(series[1:4])

  # Output
  1    20
  2    30
  3    40
  dtype: int64
  ```
  ```python
  import pandas as pd

  data = [100, 200, 300, 400]
  index = ['a', 'b', 'c', 'd']
  series = pd.Series(data, index=index)

  # Slice from label 'b' to 'd'
  print(series['b':'d'])

  # Output
  b    200
  c    300
  d    400
  dtype: int64 # Note that, when using custom indexing, the output includes the ending index.
  ```
- Data can be filtered from a series as follows:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40, 50]
  series = pd.Series(data)

  # Select values greater than 25
  print(series[series > 25])

  # Output
  2    30
  3    40
  4    50
  dtype: int64
  ```
- The `get()` method can also be used to access elements in a series. It will return `None` if the index is not found, instead of throwing an error. For example:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  index = ['a', 'b', 'c', 'd']
  series = pd.Series(data, index=index)

  # Get value with label 'b'
  print(series.get('b')) # Outputs 20

  # Attempt to get value with label 'x' (returns None)
  print(series.get('x')) # Outputs None
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  numbers = [5, 10, 15, 20, 25]
  numbers_series = pd.Series(numbers)

  first = numbers_series[0]
  last = numbers_series[len(numbers_series) -1]
  elements = numbers_series[[1, 3]] # Accesses elements 2 and 4
  elements = numbers_series[1:4] # Accesses the elements, 2, 3, and 4
  print(first)
  print(last)
  print(elements)

  fruits = {'apple': 100, 'banana': 50, 'orange': 80}
  fruits_series = pd.Series(fruits)

  banana = fruits_series['banana']
  sliced = fruits_series['banana':'orange']
  values = fruits_series[fruits_series > 70]
  apple = fruits_series.get('apple')
  mango = fruits_series.get('mango')
  print(banana)
  print(sliced)
  print(values)
  print(apple)
  print(mango)
  ```

### Operations on a Series
- Pandas allows you to perform a wide range of operations directly on a series, including mathematical, statistical, and element-wise operations.
- Mathematical Operations:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  series = pd.Series(data)

  # Add 5 to each element
  result = series + 5
  print(result)

  # Output
  0    15
  1    25
  2    35
  3    45
  dtype: int64
  ```
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  series = pd.Series(data)

  # Divide each element by 5
  result = series / 5
  print(result) # The division converts the int to a float

  # Output
  0    2.0
  1    4.0
  2    6.0
  3    8.0
  dtype: float64
  ```
- Statistical Operations:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  series = pd.Series(data)

  # Calculate the mean of the Series
  mean = series.mean()
  print(mean) # Outputs 25.0

  # Calculate the sum of all elements
  total = series.sum()
  print(total) # Outputs 100

  # Get the minimum and maximum values
  minimum = series.min()
  maximum = series.max()
  print(minimum, maximum) # Outputs 10 40

  # Count the number of elements
  count = series.count()
  print(count) # Outputs 4

  # Calculate the standard deviation
  std_dev = series.std()
  print(std_dev) # Outputs 12.909944487358056

  # Calculate the median
  median = series.median()
  print(median) # Outputs 25.0
  ```
- Element-Wise Operations:
  ```python
  import pandas as pd

  data = [10, 20, 30, 40]
  series = pd.Series(data)

  # Double each value using apply()
  result = series.apply(lambda x: x * 2)
  print(result)

  # Output
  0    20
  1    40
  2    60
  3    80
  dtype: int64

  # Square each value using map()
  result = series.map(lambda x: x ** 2)
  print(result)

  # Output
  0     100
  1     400
  2     900
  3    1600
  dtype: int64

  # Replace values in a Series
  result = series.replace({10: 100, 30: 300})
  print(result)

  # Output
  0    100
  1     20
  2    300
  3     40
  dtype: int64
  ```
  - The `apply()` method accepts an anonymous function that will be applied to each element in the series.
  - The `map()` method also accepts an anonymous function that will be applied to each element in the series. The primary differences between `apply()` and `map()` are:
    - `map()` only works on series while `apply()` works on series and data frames.
    - `map()` accepts functions, dictionaries, and series while `apply()` only accepts functions.
    - `map()` performs element-wise transformations and substitutions while `apply()` can also perform more complex aggregations.
    - `map()` performs faster for simple, element-wise operations on a series or dictionary.
- Operations Between Series:
  ```python
  import pandas as pd

  series1 = pd.Series([1, 2, 3, 4])
  series2 = pd.Series([10, 20, 30, 40])

  # Add two Series
  result = series1 + series2
  print(result)

  # Output
  0    11
  1    22
  2    33
  3    44
  dtype: int64
  ```
  - Series must be the same length to perform these operations.
- Comprehensive Example:
  ```python
  import pandas as pd

  numbers = [2, 4, 6, 8, 10]
  numbers_series = pd.Series(numbers)

  add_5 = numbers_series + 5
  print(add_5)

  multiply_3 = numbers_series * 3
  print(multiply_3)

  mean = numbers_series.mean()
  std_dev = numbers_series.std()
  print(mean)
  print(std_dev)

  series1 = pd.Series([10, 20, 30])
  series2 = pd.Series([5, 15, 25])
  series_sum = series1 + series2
  print(series_sum)

  series3 = pd.Series([10, 20, 30, 40])
  square_series3 = series3.apply(lambda x: x ** 2)
  modify_series3 = series3.replace({20: 200})
  print(square_series3)
  print(modify_series3)
  ```

## DataFrame Creation
- A data frame is a two-dimensional, labeled data structure similar to a table in a relational database or spreadsheet.
  - Rows and columns have labels (index and column names).
  - Can hold different data types.
  - Missing values are handled as `NaN`.
- A data frame consists of:
  - Index - Acts as a label for rows.
  - Column - The column labels.
  - Data - The values stored in the table.
- Example:
  ```python
  Index	Name	   Age	City
  0	    Alice	   25	  New York
  1	    Bob	     30	  Chicago
  2	    Charlie	 35	  Boston
  ```
- The syntax for creating a data frame in pandas is as follows:
  ```python
  pd.DataFrame(data, index=[row labels], columns=[column labels])
  ```
  - `data` - Data from the list, dictionary, or array.
  - `index` - Labels for the rows (optional). Zero-based index label by default.
  - `columns` - Labels for the columns (optional). Zero-based index label by default.
- DataFrame Creation Example:
  ```python
  import pandas as pd

  data = [[1, 'Alice', 25], [2, 'Bob', 30], [3, 'Charlie', 35]]
  df = pd.DataFrame(data, columns=['ID', 'Name', 'Age'])
  print(df)

  # Output
    ID     Name  Age
  0   1    Alice   25
  1   2      Bob   30
  2   3  Charlie   35
  ```
  ```python
  import pandas as pd

  data = {
      'ID': [1, 2, 3],
      'Name': ['Alice', 'Bob', 'Charlie'],
      'Age': [25, 30, 35]
  }
  df = pd.DataFrame(data)
  print(df)

  # Output
    ID     Name  Age
  0   1    Alice   25
  1   2      Bob   30
  2   3  Charlie   35
  ```
  ```python
  import pandas as pd

  data = [
      {'ID': 1, 'Name': 'Alice', 'Age': 25},
      {'ID': 2, 'Name': 'Bob', 'Age': 30},
      {'ID': 3, 'Name': 'Charlie', 'Age': 35}
  ]
  df = pd.DataFrame(data)
  print(df)

  # Output
    ID     Name  Age
  0   1    Alice   25
  1   2      Bob   30
  2   3  Charlie   35
  ```
  ```python
  import pandas as pd
  import numpy as np

  data = np.array([[1, 'Alice', 25], [2, 'Bob', 30], [3, 'Charlie', 35]])
  df = pd.DataFrame(data, columns=['ID', 'Name', 'Age'])
  print(df)

  # Output
    ID     Name Age
  0  1    Alice  25
  1  2      Bob  30
  2  3  Charlie  35
  ```
  ```python
  import pandas as pd

  data = pd.Series([100, 200, 300], index=['a', 'b', 'c'])
  df = pd.DataFrame(data, columns=['Value'])
  print(df)

  # Output
    Value
  a    100
  b    200
  c    300
  ```
- Defining Custom Index and Columns:
  ```python
  import pandas as pd

  data = [[1, 'Alice', 25], [2, 'Bob', 30], [3, 'Charlie', 35]]
  df = pd.DataFrame(data, index=['row1', 'row2', 'row3'], columns=['ID', 'Name', 'Age'])
  print(df)

  # Output
        ID     Name  Age
  row1    1    Alice   25
  row2    2      Bob   30
  row3    3  Charlie   35
  ```
- Adding Columns to an Existing DataFrame:
  ```python
  df['City'] = ['New York', 'Chicago', 'Boston']
  print(df)

  # Output
        ID     Name  Age      City
  row1    1    Alice   25  New York
  row2    2      Bob   30   Chicago
  row3    3  Charlie   35    Boston
  ```
- Adding Rows to an Existing DataFrame:
  ```python
  df.loc['row4'] = [4, 'David', 28, 'Seattle']
  print(df)

  # Output
        ID     Name  Age      City
  row1    1    Alice   25  New York
  row2    2      Bob   30   Chicago
  row3    3  Charlie   35    Boston
  row4    4    David   28   Seattle
  ```
- Deleting a Column:
  ```python
  df.drop('City', axis=1, inplace=True)
  print(df)

  # Output
        ID     Name  Age
  row1    1    Alice   25
  row2    2      Bob   30
  row3    3  Charlie   35
  row4    4    David   28
  ```
- Deleting a Row:
  ```python
  df.drop('row4', axis=0, inplace=True)
  print(df)

  # Output
        ID     Name  Age
  row1    1    Alice   25
  row2    2      Bob   30
  row3    3  Charlie   35
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  people = [[101, 'John', 29], [102, 'Anna', 25], [103, 'Peter', 32]]
  people_frame = pd.DataFrame(people, columns=['ID', 'Name', 'Age'])
  people_frame['City'] = ['NY', 'LA', 'Chicago']
  print(people_frame)

  products = {'Product': ['A', 'B', 'C'], 'Price': [100, 200, 150]}
  products_frame = pd.DataFrame(products)
  products_frame.loc[3] = ['D', 180]
  products_frame.drop(2, axis=0, inplace=True)
  print(products_frame)
  ```

### Removing Duplicates
- A duplicate row in a data frame refers to two or more rows that match **exactly** with one another. Duplicate rows can be identified using the `duplicated()` method as follows:
  ```python
  DataFrame.duplicated(subset=None, keep='first')
  ```
  - `subset` refers to the list of columns to check for duplicates (all columns by default).
  - `keep` refers to the occurrence that should be kept (marked as `False`. `first` by default). `first` keeps the first occurrence. `last` keeps the last occurrence. `False` marks all duplicates as `True`.
- Identifying Duplicates in a DataFrame:
  ```python
  import pandas as pd

  data = {
      'ID': [1, 2, 1, 3, 4, 4],
      'Name': ['Alice', 'Bob', 'Alice', 'Charlie', 'David', 'David'],
      'Age': [25, 30, 25, 35, 40, 40]
  }

  df = pd.DataFrame(data)

  # Identify duplicates
  print(df.duplicated()) # subset = None, keep='first' by default.

  # Output
  0    False
  1    False
  2     True
  3    False
  4    False
  5     True
  dtype: bool
  ```
  ```python
  # Check for duplicates based only on the 'Name' column
  print(df.duplicated(subset=['Name'])) # keep='first' by default.

  # Output
  0    False
  1    False
  2     True
  3    False
  4    False
  5     True
  dtype: bool
  ```
  ```python
  # Mark all but the last occurrence as duplicate
  print(df.duplicated(keep='last'))

  # Output
  0     True
  1    False
  2    False
  3    False
  4     True
  5    False
  dtype: bool
  ```
- Duplicate rows can in a data frame be removed using the `drop_duplicates()` method as follows:
  ```python
  DataFrame.drop_duplicates(subset=None, keep='first', inplace=False)
  ```
  - `subset` refers to the list of columns to check for duplicates (all columns by default).
  - `keep` refers to the occurrence that should be kept (marked as `False`. `first` by default). `first` keeps the first occurrence. `last` keeps the last occurrence. `False` marks all duplicates as `True`.
  - `inplace` refers to the modification of the data frame (`False` by default). `True`modifies the data frame directly.
- Removing Duplicates in a DataFrame:
  ```python
  import pandas as pd

  data = {
      'ID': [1, 2, 1, 3, 4, 4],
      'Name': ['Alice', 'Bob', 'Alice', 'Charlie', 'David', 'David'],
      'Age': [25, 30, 25, 35, 40, 40]
  }

  df = pd.DataFrame(data)

  # Remove all duplicate rows
  df_cleaned = df.drop_duplicates()
  print(df_cleaned)

  # Output
    ID     Name  Age
  0   1    Alice   25
  1   2      Bob   30
  3   3  Charlie   35
  4   4    David   40
  ```
  ```python
  # Remove duplicates based on the 'Name' column
  df_cleaned = df.drop_duplicates(subset=['Name'])
  print(df_cleaned)

  # Output
    ID     Name  Age
  0   1    Alice   25
  1   2      Bob   30
  3   3  Charlie   35
  4   4    David   40
  ```
  ```python
  # Remove duplicates based on 'Name' and 'Age'
  df_cleaned = df.drop_duplicates(subset=['Name', 'Age'])
  print(df_cleaned)

  # Output
    ID     Name  Age
  0   1    Alice   25
  1   2      Bob   30
  3   3  Charlie   35
  4   4    David   40
  ```
  ```python
  # Keep the last occurrence of duplicates
  df_cleaned = df.drop_duplicates(keep='last')
  print(df_cleaned)

  # Output
    ID     Name  Age
  1   2      Bob   30
  2   1    Alice   25
  3   3  Charlie   35
  5   4    David   40
  ```
  ```python
  # Remove all instances of duplicates
  df_cleaned = df.drop_duplicates(keep=False)
  print(df_cleaned)

  # Output
    ID     Name  Age
  1   2      Bob   30
  3   3  Charlie   35
  ```
- Duplicate rows in a data frame can be counted using the `sum()` method as follows:
  ```python
  # Count the number of duplicates
  num_duplicates = df.duplicated().sum()
  print(f"Number of duplicate rows: {num_duplicates}") # Outputs 2 for the 2 sets of duplicate rows
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'ID': [1, 2, 1, 3, 2],
      'Name': ['Alice', 'Bob', 'Alice', 'Charlie', 'Bob'],
      'Age': [25, 30, 25, 35, 30],
      'City': ['New York', 'Chicago', 'New York', 'Boston', 'Chicago']
  }

  df = pd.DataFrame(data)
  duplicates = df.duplicated()
  df_cleaned = df.drop_duplicates()
  df_cleaned_name = df.drop_duplicates(subset=['Name'], keep='last')
  duplicate_count = df.duplicated().sum()

  print(duplicates)
  print(df_cleaned)
  print(df_cleaned_name)
  print(duplicate_count)
  ```

### Slicing and Indexing
- Slicing and indexing are methods used to access specific rows, columns, or subsets of data in a data frame or series.
- Indexing allows you to access elements or specific groups of elements using row and column labels or numeric positions.
- Slicing allows you to access a range of rows or columns in a data frame or series.
- Selecting a Single Column:
  ```python
  import pandas as pd

  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Age': [25, 30, 35, 40],
      'City': ['New York', 'Chicago', 'Boston', 'Seattle']
  }

  df = pd.DataFrame(data)

  # Select a single column
  print(df['Name'])

  # Using dot notation
  print(df.Age)

  # Output
  0     Alice
  1       Bob
  2   Charlie
  3     David
  Name: Name, dtype: object

  0    25
  1    30
  2    35
  3    40
  Name: Age, dtype: int64
  ```
- Selecting Multiple Columns:
  ```python
  # Select multiple columns
  print(df[['Name', 'Age']])

  # Output
        Name  Age
  0    Alice   25
  1      Bob   30
  2  Charlie   35
  3    David   40
  ```
- Selecting a Row by Index:
  ```python
  # Select the first row using iloc
  print(df.iloc[0])

  # Select row using label with loc
  print(df.loc[0])

  # Output
  Name       Alice
  Age           25
  City    New York
  Name: 0, dtype: object
  ```
  - The `iloc[]` method allows you to slice rows or columns using the numeric index position.
  - The `loc[]` method allows you to slice rows or columns using labels.
- Slicing Rows by Index Position:
  ```python
  # Slice rows from index 1 to 2 (excludes index 3)
  print(df.iloc[1:3])

  # Output
        Name  Age    City
  1      Bob   30  Chicago
  2  Charlie   35   Boston
  ```
- Slicing Rows by Labels:
  ```python
  # Slice rows from label 1 to 2 (inclusive)
  print(df.loc[1:2])

  # Output
        Name  Age    City
  1      Bob   30  Chicago
  2  Charlie   35   Boston
  ```
- Slicing Columns by Position:
  ```python
  # Slice the first two columns
  print(df.iloc[:, 0:2])

  # Output
        Name  Age
  0    Alice   25
  1      Bob   30
  2  Charlie   35
  3    David   40
  ```
- Slicing Columns by Labels:
  ```python
  # Slice specific columns by label
  print(df.loc[:, ['Name', 'City']])

  # Output
        Name      City
  0    Alice  New York
  1      Bob   Chicago
  2  Charlie    Boston
  3    David   Seattle
  ```
- Conditional Selection of Rows:
  ```python
  # Select rows where Age > 30
  print(df[df['Age'] > 30])

  # Output
        Name  Age    City
  2  Charlie   35   Boston
  3    David   40  Seattle
  ```
  ```python
  # Select rows where Age > 30 AND City is Boston
  print(df[(df['Age'] > 30) & (df['City'] == 'Boston')])

  # Output
        Name  Age    City
  2  Charlie   35   Boston
  ```
- The `isin()` method can be used to match values from a list. For example:
  ```python
  # Select rows where City is in ['Chicago', 'Boston']
  print(df[df['City'].isin(['Chicago', 'Boston'])])

  # Output
        Name  Age    City
  1      Bob   30  Chicago
  2  Charlie   35   Boston
  ```
- The index of a data frame can be reset using the `reset_index()` method as follows:
  ```python
  # Reset index
  df.reset_index(drop=True, inplace=True)
  print(df)
  ```
- The index of a data frame can be set using the `set_index()` method as follows:
  ```python
  # Set 'Name' as the index
  df.set_index('Name', inplace=True)
  print(df)

  # Output
  Name    Age      City
                   
  Alice     25  New York
  Bob       30   Chicago
  Charlie   35    Boston
  David     40   Seattle
  ```
- Rows can be filtered using the `query()` method as follows:
  ```python
  # Select rows where Age > 30
  print(df.query('Age > 30'))

  # Output
  Name     Age    City
                
  Charlie   35   Boston
  David     40  Seattle
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Age': [25, 30, 35, 40],
      'City': ['New York', 'Chicago', 'Boston', 'Seattle']
  }

  df = pd.DataFrame(data)
  ages = df.Age
  rows1_3 = df.iloc[1:3]
  rows_age_over_30 = df[df['Age'] > 30]
  names_and_cities = df.iloc[:, [0, 2]]
  filtered_rows = df[df['City'].isin(['Boston', 'Chicago'])]
  print(ages)
  print(rows1_3)
  print(rows_age_over_30)
  print(names_and_cities)
  print(filtered_rows)
  ```

### Indexing Operations
- Indexing in pandas allows you to select specific rows and columns from a data frame or series using labels or numeric positions.
- The `loc[]` method allows you to select rows and columns using **label-based** indexing. It includes the start and end index in slicing. This method also supports boolean indexing or conditional selection. The syntax is as follows:
  ```python
  DataFrame.loc[row_label, column_label]
  ```
- Selecting Rows and Columns by Label:
  ```python
  import pandas as pd

  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Age': [25, 30, 35, 40],
      'City': ['New York', 'Chicago', 'Boston', 'Seattle']
  }

  df = pd.DataFrame(data, index=['a', 'b', 'c', 'd'])

  # Select a single row using loc
  print(df.loc['a'])

  # Output
  Name       Alice
  Age           25
  City    New York
  Name: a, dtype: object

  # Select multiple rows using loc
  print(df.loc[['a', 'c']])

  # Output
        Name  Age      City
  a    Alice   25  New York
  c  Charlie   35    Boston

  # Select rows 'a' and 'c' and columns 'Name' and 'Age'
  print(df.loc[['a', 'c'], ['Name', 'Age']])

  # Output
        Name  Age
  a    Alice   25
  c  Charlie   35

  # Slice rows from 'b' to 'd' and select columns from 'Name' to 'Age'
  print(df.loc['b':'d', 'Name':'Age'])

  # Output
        Name  Age
  b      Bob   30
  c  Charlie   35
  d    David   40

  # Select rows where Age > 30
  print(df.loc[df['Age'] > 30])

  # Output
        Name  Age    City
  c  Charlie   35  Boston
  d    David   40  Seattle
  ```
- The `iloc[]` method allows you to select rows and columns based on their **integer position**. Indexes start at zero and the end index is excluded. The syntax is as follows:
  ```python
  DataFrame.iloc[row_index, column_index]
  ```
- Selecting Rows and Columns by Index:
  ```python
  # Select the first row
  print(df.iloc[0])

  # Output
  Name       Alice
  Age           25
  City    New York
  Name: a, dtype: object

  # Select the first and third rows
  print(df.iloc[[0, 2]])

  # Output
        Name  Age    City
  a    Alice   25  New York
  c  Charlie   35   Boston

  # Select the first and third rows and the first two columns
  print(df.iloc[[0, 2], [0, 1]])

  # Output
        Name  Age
  a    Alice   25
  c  Charlie   35

  # Slice rows from index 1 to 3 and columns from index 0 to 2
  print(df.iloc[1:3, 0:2])

  # Output
        Name  Age
  b      Bob   30
  c  Charlie   35

  # Select rows where Age > 30 using iloc
  print(df.iloc[(df['Age'] > 30).values])

  # Output
        Name  Age    City
  c  Charlie   35  Boston
  d    David   40  Seattle
  ```
  - You cannot use direct conditional selection with `iloc[]`, but you can combine `iloc[]` with boolean indexing.
- Differences Between `loc[]` and `iloc[]`:
  - `loc[]` uses label-based (row and column names) indexing while `iloc[]` uses position-based (integer position) indexing.
  - Conditional selection is only directly supported with `loc[]`. `iloc[]` requires boolean indexing.
- Updating Value With `loc[]`:
  ```python
  # Update Age for Bob
  df.loc['b', 'Age'] = 32
  print(df)
  ```
- Updating Value With `iloc[]`:
  ```python
  # Update Age for second row (Bob)
  df.iloc[1, 1] = 33
  print(df)
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Age': [25, 30, 35, 40],
      'City': ['New York', 'Chicago', 'Boston', 'Seattle']
  }

  df = pd.DataFrame(data)
  charlie = df.loc[2]
  print(charlie)
  second_row = df.iloc[1]
  print(second_row)
  two_rows = df.iloc[1:3]
  print(two_rows)
  over_30 = df.loc[df['Age'] > 30]
  print(over_30)
  two_columns = df.iloc[:, :2]
  print(two_columns)
  df.loc[1, 'Age'] = 32
  print(df)
  ```

### Data Types
- Each column in a data frame has a specific data type that defines the type of data it stores. Pandas uses NumPy data types for better performance and memory efficiency.
- Common data types in pandas include:
  - `int64` - Integer values.
  - `float64` - Decimal values.
  - `object` - Text or string data.
  - `bool` - Boolean values.
  - `datatime64` - Date and time values.
  - `category` - Categorical data (kind of like an enum). Categorical data reduces memory and improves performance for columns with repeated values.
- A data frame's data types can be checked using the `dtypes` attribute as follows:
  ```python
  import pandas as pd

  # Create DataFrame
  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Age': [25, 30, 35, 40],
      'City': ['New York', 'Chicago', 'Boston', 'Seattle'],
      'Salary': [50000.5, 60000.0, 75000.75, 80000.25],
      'Employed': [True, False, True, True]
  }

  df = pd.DataFrame(data)

  # Check data types
  print(df.dtypes)

  # Output
  Name         object
  Age           int64
  City         object
  Salary       float64
  Employed        bool
  dtype: object
  ```
- Columns of a specific data type can be selected using the `select_dtypes()` method as follows:
  ```python
  # Select numeric columns (int and float)
  numeric_cols = df.select_dtypes(include=['int64', 'float64'])
  print(numeric_cols)

  # Output
    Age   Salary
  0   25  50000.50
  1   30  60000.00
  2   35  75000.75
  3   40  80000.25

  # Select object (string) columns
  string_cols = df.select_dtypes(include=['object'])
  print(string_cols)

  # Output
        Name     City
  0    Alice  New York
  1      Bob   Chicago
  2  Charlie    Boston
  3    David   Seattle
  ```
- The data type of column can be modified using the `astype()` method as follows:
  ```python
  # Convert Age to float
  df['Age'] = df['Age'].astype('float64')
  print(df.dtypes)

  # Output
  Name         object
  Age         float64
  City         object
  Salary       float64
  Employed        bool
  dtype: object

  # Convert City to category
  df['City'] = df['City'].astype('category')
  print(df.dtypes)

  Name           object
  Age           float64
  City         category
  Salary         float64
  Employed          bool
  dtype: object

  # Convert Employed (True/False) to Integer (1/0)
  df['Employed'] = df['Employed'].astype('int64')
  print(df.dtypes)

  # Output
  Name           object
  Age           float64
  City         category
  Salary         float64
  Employed         int64
  dtype: object

  # Add a Date column
  df['Date'] = ['2025-03-18', '2025-03-19', '2025-03-20', '2025-03-21']

  # Convert to datetime
  df['Date'] = pd.to_datetime(df['Date'])
  print(df.dtypes)

  # Output
  Name                object
  Age                float64
  City              category
  Salary              float64
  Employed              int64
  Date         datetime64[ns]
  dtype: object
  ```
- When a column contains values with multiple data types, you can convert to a single data type or clean it. For example:
  ```python
  # Create a column with mixed types
  df['Mixed'] = [1, 'text', 3.5, True]

  # Convert to string
  df['Mixed'] = df['Mixed'].astype(str)
  print(df.dtypes)

  # Output
  Name                object
  Age                float64
  City              category
  Salary              float64
  Employed              int64
  Date         datetime64[ns]
  Mixed               object
  dtype: object
  ```
- When a conversion fails due to invalid data, it can be handled using `errors=coerce`. When this is used, invalid values are converted to `NaN` (Not a Number). For example:
  ```python
  # Invalid data conversion
  df['Mixed'] = pd.to_numeric(df['Mixed'], errors='coerce')
  print(df.dtypes)

  # Output
  Name                object
  Age                float64
  City              category
  Salary              float64
  Employed              int64
  Date         datetime64[ns]
  Mixed              float64
  dtype: object
  ```
  - `errors` can either be `raise` (raises an error), `ignore` (returns original object), or `coerce` (converts object to `NaN`).
- Memory usage can be reduced by performing the following conversions:
  - `int64` to `int32`
  - `float64` to `float32`
  - `object` to `category`
  - For example:
  ```python
  # Convert Age and Salary to float32
  df['Age'] = df['Age'].astype('float32')
  df['Salary'] = df['Salary'].astype('float32')

  # Convert City to category
  df['City'] = df['City'].astype('category')

  print(df.dtypes)

  # Output
  Name                object
  Age                float32
  City              category
  Salary              float32
  Employed              int64
  Date         datetime64[ns]
  Mixed              float64
  dtype: object
  ```
- Missing (`NaN`) values that result from converting a column's data type using `errors=coerce` can be handled using handled using the `fillna()` method as follows:
  ```python
  # Fill NaN values after conversion
  df['Mixed'] = df['Mixed'].fillna(0)
  print(df['Mixed'])

  # Output
  0    1.0
  1    0.0
  2    3.5
  3    1.0
  Name: Mixed, dtype: float64
  ```

### Reading From CSV and Excel in DataFrame
- In real-world data engineering or analysis projects, data is often stored in external csv and excel files. Pandas provides several methods that can be used to load data from these files into a data frame for analysis and processing.
- The `read_csv()` method allows you to read data directly from a csv file into a data frame. The syntax for this method is as follows:
  ```python
  pd.read_csv(filepath, **options)
  ```
  - `filepath` -	Path to the csv file.
  - `sep` -	Delimiter (default = ,).
  - `header` -	Row number to use as column names (default = 0).
  - `usecols` -	List of columns to read.
  - `index_col` -	Column to use as the row labels.
  - `dtype` -	Data type for each column.
  - `na_values` -	Values to treat as NaN (missing).
  - `encoding` -	File encoding (e.g., 'utf-8', 'latin1').
- Reading CSV File Examples:
  ```python
  import pandas as pd

  # data.csv
  Name,Age,City,Salary,Employed
  Alice,25,New York,50000.5,True
  Bob,30,Chicago,60000.0,False
  Charlie,35,Boston,75000.75,True
  David,40,Seattle,80000.25,True

  # Read CSV file
  df = pd.read_csv('data.csv')

  # Display DataFrame
  print(df)

  # Output
        Name  Age      City   Salary  Employed
  0    Alice   25  New York  50000.5      True
  1      Bob   30   Chicago  60000.0     False
  2  Charlie   35    Boston  75000.8      True
  3    David   40   Seattle  80000.2      True

  # data_semicolon.csv
  Name;Age;City;Salary;Employed
  Alice;25;New York;50000.5;True
  Bob;30;Chicago;60000.0;False
  Charlie;35;Boston;75000.75;True
  David;40;Seattle;80000.25;True

  # Read CSV with a semicolon delimiter
  df = pd.read_csv('data_semicolon.csv', sep=';')

  print(df)

  # Output
        Name  Age      City   Salary  Employed
  0    Alice   25  New York  50000.5      True
  1      Bob   30   Chicago  60000.0     False
  2  Charlie   35    Boston  75000.8      True
  3    David   40   Seattle  80000.2      True

  # Read CSV and specify column data types
  df = pd.read_csv('data.csv', dtype={'Age': 'int32', 'Salary': 'float32'})
  print(df.dtypes)

  # Output
  Name         object
  Age           int32
  City         object
  Salary       float32
  Employed     object
  dtype: object

  # Use 'Name' as index
  df = pd.read_csv('data.csv', index_col='Name')
  print(df)

  # Output
  Name    Age      City   Salary  Employed
                                    
  Alice     25  New York  50000.5      True
  Bob       30   Chicago  60000.0     False
  Charlie   35    Boston  75000.8      True
  David     40   Seattle  80000.2      True

  # data_with_missing.csv
  Name,Age,City,Salary,Employed
  Alice,25,New York,50000.5,True
  Bob,,Chicago,60000.0,False
  Charlie,35,,75000.75,True
  David,40,Seattle,,True

  # Treat empty cells as NaN
  df = pd.read_csv('data_with_missing.csv', na_values=['', 'NA', 'null'])
  print(df)

  # Output
        Name   Age      City   Salary  Employed
  0    Alice  25.0  New York  50000.5      True
  1      Bob   NaN   Chicago  60000.0     False
  2  Charlie  35.0       NaN  75000.8      True
  3    David  40.0   Seattle      NaN      True
  ```
- The `read_excel()` method allows you to read data directly from excel file (`.xlsx` or `.xls`) into a data frame. The syntax for this method is as follows:
  ```python
  pd.read_excel(filepath, sheet_name=0, **options)
  ```
  - `filepath` -	Path to the Excel file
  - `sheet_name` -	Sheet to read (default = 0)
  - `header` -	Row number to use as column names
  - `index_col` -	Column to use as row labels
  - `dtype` -	Data type for each column
- Reading Excel File Examples
  ```python
  import pandas as pd

  # data.xlsx
  Name	Age	City	Salary	Employed
  Alice	25	New York	50000.5	True
  Bob	30	Chicago	60000.0	False
  Charlie	35	Boston	75000.75	True
  David	40	Seattle	80000.25	True

  # Read Excel file
  df = pd.read_excel('data.xlsx', sheet_name='Sheet1')
  print(df)

  # Read data from second sheet
  df = pd.read_excel('data.xlsx', sheet_name=1)
  print(df)

  # Read specific columns
  df = pd.read_excel('data.xlsx', usecols=['Name', 'City'])
  print(df)
  ```

### Handling Missing Values in a DataFrame
- In real-world data, missing values are common due to a variety of reasons, including:
  - Data entry errors.
  - Data loss during transmission.
  - Incomplete data collection.
- Handling missing values is important for several reasons, including:
  - Machine learning models cannot handle missing values directly.
  - Statistical analysis can be skewed by missing values.
  - Missing values can affect the accuracy of business insights.
- Missing values in a data frame can be identified using the following methods:
  ```sql
  import pandas as pd

  # Create a sample DataFrame
  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Age': [25, None, 35, 40],
      'City': ['New York', 'Chicago', None, 'Seattle'],
      'Salary': [50000.5, 60000.0, None, 80000.25]
  }

  df = pd.DataFrame(data)

  # Identify missing values
  print(df.isna())

  # Output
      Name    Age   City  Salary
  0  False  False  False   False
  1  False   True  False   False
  2  False  False   True    True
  3  False  False  False   False

  print(df.isnull())
  print(df.notna())

  df.info()

  # Output
  <class 'pandas.core.frame.DataFrame'>
  RangeIndex: 4 entries, 0 to 3
  Data columns (total 4 columns):
  #   Column  Non-Null Count  Dtype  
  ---  ------  --------------  -----  
  0   Name    4 non-null      object 
  1   Age     3 non-null      float64
  2   City    3 non-null      object 
  3   Salary  3 non-null      float64
  dtypes: float64(2), object(2)
  memory usage: 256.0 bytes
  ```
  - `isna()` - Returns `True` when the value is missing (`NaN`) and `False` otherwise.
  - `isnull()` - Same as `isna()`.
  - `notna()` - Returns `True` for non-missing values and `False` otherwise.
  - `info()` - Provides an overview of the data frame, including a count of the non-null values.
- The `dropna()` method can be used to remove rows where *any* value is missing. For example:
  ```python
  # Remove rows with missing values
  df_cleaned = df.dropna()
  print(df_cleaned)

  # Output
        Name   Age      City   Salary
  0    Alice  25.0  New York  50000.5
  3    David  40.0   Seattle  80000.2

  # Remove columns with missing values
  df_cleaned = df.dropna(axis=1)
  print(df_cleaned)

  # Output
        Name
  0    Alice
  1      Bob
  2  Charlie
  3    David

  # Drop rows where all values are missing
  df_cleaned = df.dropna(how='all')
  print(df_cleaned)
  ```
- The `fillna()` method can be used to fill missing values with a default value, instead of removing them. For example:
  ```python
  # Fill missing values with a fixed value
  df_filled = df.fillna(0)
  print(df_filled)

  # Output
        Name   Age      City   Salary
  0    Alice  25.0  New York  50000.5
  1      Bob   0.0   Chicago  60000.0
  2  Charlie  35.0  unknown      0.0
  3    David  40.0   Seattle  80000.2

  # Fill missing values with mean
  df['Age'].fillna(df['Age'].mean(), inplace=True)
  df['Salary'].fillna(df['Salary'].median(), inplace=True)
  print(df)

  # Fill missing values with mean and median for Pandas 3.o
  df['Age'] = df['Age'].fillna(df['Age'].mean())
  df['Salary'] = df['Salary'].fillna(df['Salary'].median())
  print(df)

  # Output
        Name   Age      City   Salary
  0    Alice  25.0  New York  50000.5
  1      Bob  33.3   Chicago  60000.0
  2  Charlie  35.0      NaN  60000.0
  3    David  40.0   Seattle  80000.2
  ```
- The `ffill()` method fills a missing value with the previous row's value. For example:
  ```python
  df.ffill(inplace=True)
  print(df)
  ```
- The `bfill()` method fills a missing value with next row's value. For example:
  ```python
  df.bfill(inplace=True)
  print(df)
  ```
- The `interpolate()` method can be used to fill missing values based on linear patterns. For example:
  ```python
  # Interpolate missing values
  df['Age'] = df['Age'].interpolate()
  print(df)
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Emma'],
      'Age': [25, None, 35, 40, None],
      'City': ['New York', 'Chicago', None, 'Seattle', None],
      'Salary': [50000.5, 60000.0, None, 80000.25, None],
      'Employed': [True, False, True, True, False]
  }

  df = pd.DataFrame(data)
  print(f"Total missing values: {df.isnull().sum().sum()}")

  df['Age'] = df['Age'].fillna(df['Age'].mean())
  df['City'] = df['City'].fillna('unknown')
  df['Salary'] = df['Salary'].fillna(df['Salary'].median())
  df_cleaned = df.dropna(how='all')
  print(f"Cleaned DataFrame:\n{df_cleaned}")
  ```
  - Note that `None` was used to simulate missing values in this example, not `'NaN'`.

### Customizing Output Options
- By default, pandas automatically limits the number of rows and columns displayed to avoid overwhelming the console. In real-world scenarios, you may want to adjust these settings to:
  - View large datasets more easily.
  - Display more or less decimal points for accuracy.
  - Format large numbers for better readability.
  - Improve data analysis and debugging.
- Pandas provides a variety of options that you can control using the `pd.set_option()` method. The syntax for this method is as follows:
  ```python
  pd.set_option('display.<option>', value)
  ```
- The maximum number of rows and columns displayed in the output can be adjusted as follows:
  ```python
  # Sample DataFrame
  data = {
      'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Emma', 'Frank', 'George', 'Henry', 'Ian', 'Jack'],
      'Age': [25, 30, 35, 40, 29, 33, 38, 45, 27, 31],
      'City': ['New York', 'Chicago', 'Boston', 'Seattle', 'Austin', 'Dallas', 'San Francisco', 'Los Angeles', 'Miami', 'Denver'],
      'Salary': [50000.5, 60000.0, 75000.75, 80000.25, 45000.0, 55000.5, 70000.0, 85000.0, 48000.0, 51000.0]
  }

  df = pd.DataFrame(data)

  # Increase the maximum number of rows displayed
  pd.set_option('display.max_rows', 100)

  # Increase the maximum number of columns displayed
  pd.set_option('display.max_columns', 50)

  # Display the DataFrame
  print(df)
  ```
- Display settings can be reset using the `reset_option()` method as follows:
  ```python
  pd.reset_option('display.max_rows')
  pd.reset_option('display.max_columns')
  ```
- The precision of float values can be adjusted as follows:
  ```python
  # Set float precision to 2 decimal places
  pd.set_option('display.float_format', '{:.2f}'.format)

  print(df)

  # Output
        Name  Age           City   Salary
  0    Alice   25       New York  50000.50
  1      Bob   30        Chicago  60000.00
  2  Charlie   35         Boston  75000.75
  3    David   40        Seattle  80000.25
  4     Emma   29         Austin  45000.00
  5    Frank   33         Dallas  55000.50
  6   George   38  San Francisco  70000.00
  7    Henry   45    Los Angeles  85000.00
  8      Ian   27          Miami  48000.00
  9     Jack   31         Denver  51000.00
  ```
- Column width can be adjusted using the `max_colwidth()` method as follows:
  ```python
  # Set maximum column width to 20 characters
  pd.set_option('display.max_colwidth', 20)

  # Display the DataFrame
  print(df)
  ```
- Large or small float values are displayed using scientific notation by default. This can be disabled as follows:
  ```python
  # Disable scientific notation
  pd.set_option('display.float_format', '{:.2f}'.format)

  # Example with large and small values
  df_large = pd.DataFrame({'Value': [0.00012345, 123456789.12345]})
  print(df_large)

  # Output
          Value
  0        0.00
  1 123456789.12
  ```
- You can display all available options using the `pd.describe_option()` method.
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Product': ['Laptop', 'Phone', 'Tablet', 'Monitor', 'Keyboard'],
      'Category': ['Electronics', 'Electronics', 'Electronics', 'Electronics', 'Accessories'],
      'Price': [1000.50, 500.75, 300.25, 150.99, 50.49],
      'Quantity': [5, 10, 15, 7, 20]
  }

  df = pd.DataFrame(data)
  pd.set_option('display.max_rows', 5)
  pd.set_option('display.max_columns', 2)
  pd.set_option('display.float_format', '{:.2f}'.format)
  pd.set_option('display.max_colwidth', 15)
  print(f"Customized DataFrame Output:\n{df}")

  pd.reset_option('display.max_rows')
  pd.reset_option('display.max_columns')
  pd.reset_option('display.float_format')
  pd.reset_option('display.max_colwidth')
  print(f"\nAfter Resetting Options:\n{df}")
  ```
### Multi-Level Indexing
- A multi-level or hierarchical index allows you to have multiple levels of indexing in the same data frame. This helps in organizing and accessing data in a more structured way.
- Multi-level indexing is useful when you have data that can be organized into multiple groups or categorized into multiple levels, such as:
  - Sales data grouped by country and city.
  - Employee data grouped by department and team.
  - Financial data organized by year and quarter.
- Multi-level indexing can provide benefits such as:
  - Grouping and organizing data based on multiple dimensions.
  - Accessing subsets of data using complex indexing.
  - Enhancing performance of group-based operations.
- A multi-level index can be created by setting multiple columns as the index in the `set_index()` method. For example:
  ```python
  import pandas as pd

  # Sample data
  data = {
      'Country': ['USA', 'USA', 'India', 'India', 'Canada', 'Canada'],
      'City': ['New York', 'Chicago', 'Mumbai', 'Delhi', 'Toronto', 'Vancouver'],
      'Population': [8500000, 2700000, 20000000, 18000000, 2900000, 2500000]
  }

  df = pd.DataFrame(data)

  # Create multi-level index using 'Country' and 'City'
  df.set_index(['Country', 'City'], inplace=True)

  # Display DataFrame
  print(df)

  # Output
                  Population
  Country City               
  USA     New York     8500000
          Chicago      2700000
  India   Mumbai      20000000
          Delhi       18000000
  Canada  Toronto      2900000
          Vancouver    2500000
  ```
- A multi-level index can also be created directly while creating the data frame, using the `index` parameter as follows:
  ```python
  index = pd.MultiIndex.from_tuples(
      [('USA', 'New York'), ('USA', 'Chicago'),
      ('India', 'Mumbai'), ('India', 'Delhi'),
      ('Canada', 'Toronto'), ('Canada', 'Vancouver')],
      names=['Country', 'City']
  )

  # Create DataFrame with Multi-level index
  data = [8500000, 2700000, 20000000, 18000000, 2900000, 2500000]
  df = pd.DataFrame(data, index=index, columns=['Population'])

  # Display DataFrame
  print(df)

  # Output
                  Population
  Country City               
  USA     New York     8500000
          Chicago      2700000
  India   Mumbai      20000000
          Delhi       18000000
  Canada  Toronto      2900000
          Vancouver    2500000
  ```
- Data can be accessed with a multi-level index by passing the indexes to the `.loc[]` method as a tuple. For example:
    ```python
    # Access population of Mumbai
    print(df.loc[('India', 'Mumbai')])

    # Output
    Population    20000000
    Name: (India, Mumbai), dtype: int64
    ```
- You can also access all values under a specific level by using partial indexing. For example:
  ```python
  # Get all data for 'India'
  print(df.loc['India'])

  # Output
          Population
  City               
  Mumbai     20000000
  Delhi      18000000
  ```
- The index of a data frame can always be reset to columns using the `rest_index()` method as follows:
  ```python
  # Reset index to columns
  df_reset = df.reset_index()
  print(df_reset)

  # Output
    Country      City  Population
  0     USA  New York     8500000
  1     USA   Chicago     2700000
  2   India   Mumbai    20000000
  3   India    Delhi    18000000
  4  Canada   Toronto     2900000
  5  Canada Vancouver     2500000
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Country': ['USA', 'USA', 'India', 'India', 'Canada', 'Canada'],
      'City': ['New York', 'Chicago', 'Mumbai', 'Delhi', 'Toronto', 'Vancouver'],
      'Population': [8500000, 2700000, 20000000, 18000000, 2900000, 2500000],
      'Area': [789, 589, 603, 1484, 630, 1150]  # in square kilometers
  }
  df = pd.DataFrame(data)
  df.set_index(['Country', 'City'], inplace=True)

  mumbai_population = df.loc[('India', 'Mumbai')]
  print(f"Population of Mumbai:\n{mumbai_population}")

  india_cities = df.loc['India']
  print(f"\nAll cities under India:\n{india_cities}")

  df_reset = df.reset_index()
  print(f"\nDataFrame after resetting the index:\n{df_reset}")
  ```

### Data Type Conversion
- Data type conversion is an important concept in pandas because:
  - It ensures data is in the correct format for analysis.
  - It improves performance when working with large datasets.
  - It allows compatibility when working with other data sources and systems.
- The following methods can be used to convert data types:
  - `astype()` - Converts data to a specified type.
  - `to_numeric()` - Converts data to a numeric format.
  - `to_datetime()` - Converts data to a date/time format.
  - `to_string()` - Converts data to a string format.
  - `convert_dtypes()` - Automatically converts to the best possible data type.
- Using `astype()`:
  ```python
  # Convert 'Age' to float
  df['Age'] = df['Age'].astype('float')

  # Check data types
  print(df.dtypes)

  # Output
  Name         object
  Age         float64
  Salary      float64
  Join_Date     object
  dtype: object
  ```
- Using `to_numeric()`:
  ```python
  # Convert 'Salary' to integer
  df['Salary'] = pd.to_numeric(df['Salary'], downcast='integer')

  # Check data types
  print(df.dtypes)

  # Output
  Name         object
  Age         float64
  Salary      float64
  Join_Date     object
  dtype: object
  ```
  - If all values in a column are whole numbers, the column will be converted to `int`. If any value is a decimal, the column will be converted to `float`.
- Using `to_datetime()`:
  ```python
  # Convert 'Join_Date' to datetime
  df['Join_Date'] = pd.to_datetime(df['Join_Date'])

  # Check data types
  print(df.dtypes)

  # Output
  Name                 object
  Age                 float64
  Salary                 int32
  Join_Date    datetime64[ns]
  dtype: object
  ```
- Using `astype()` to Convert to String:
  ```python
  # Convert 'Age' to string
  df['Age'] = df['Age'].astype('str')

  # Check data types
  print(df.dtypes)

  # Output
  Name         object
  Age          object
  Salary         int32
  Join_Date    datetime64[ns]
  dtype: object
  ```
- Using `convert_dtypes()`:
  ```python
  # Automatically convert to best data type
  df = df.convert_dtypes()

  # Check data types
  print(df.dtypes)

  # Output
  Name         string
  Age          string
  Salary         Int32
  Join_Date    datetime64[ns]
  dtype: object
  ```

### Conditional Selection
- Conditional selection allows filtering of rows based specified conditions, allowing:
  - Extraction of data that meets specified criteria.
  - Application of comparison operations such as `>`, `<`, and `==`
  - Combination of multiple conditions using logical operators.
- Conditional selection is useful because:
  - It allows you to focus on specific and relevant subsets of data.
  - Helps you analyze patterns and tends based on specified criteria.
  - Allows you to dynamically filter data for better insights.
- Comparison Operators: `<`, `<=`, `>`, `>=`, `==`, and `!=`
- Logical Operators: `&` (AND), `|` (OR), and `~` (NOT)
- Example:
  ```python
  import pandas as pd

  # Sample data
  data = {
      'Product': ['Laptop', 'Phone', 'Tablet', 'Monitor', 'Keyboard'],
      'Price': [1000, 500, 300, 150, 50],
      'Stock': [10, 50, 30, 5, 100],
      'Category': ['Electronics', 'Electronics', 'Electronics', 'Electronics', 'Accessories']
  }

  df = pd.DataFrame(data)

  result = df[df['Price'] > 300]
  print(result)

  # Output
    Product  Price  Stock     Category
  0  Laptop   1000     10  Electronics
  1   Phone    500     50  Electronics

  result = df[(df['Price'] > 300) & (df['Stock'] < 20)]
  print(result)

  # Output
    Product  Price  Stock     Category
  0  Laptop   1000     10  Electronics

  result = df[(df['Price'] > 500) | (df['Stock'] > 40)]
  print(result)

  # Output
    Product  Price  Stock     Category
  0   Laptop   1000     10  Electronics
  1    Phone    500     50  Electronics
  4 Keyboard     50    100  Accessories
  ```
- The `isin()` method allows you to filter rows in a column based on values in a specified list. For example:
  ```python
  result = df[df['Category'].isin(['Electronics'])]
  print(result)

  # Output
    Product  Price  Stock     Category
  0   Laptop   1000     10  Electronics
  1    Phone    500     50  Electronics
  2   Tablet    300     30  Electronics
  3  Monitor    150      5  Electronics
  ```
- The `between()` method allows you to filter rows in a column based on a specified range. For example:
  ```python
  result = df[df['Price'].between(100, 500)]
  print(result)

  # Output
    Product  Price  Stock     Category
  1    Phone    500     50  Electronics
  2   Tablet    300     30  Electronics
  3  Monitor    150      5  Electronics
  ```
- The `str.contains` method allows you to filter string values based on a specified substring. For example:
  ```python
  result = df[df['Product'].str.contains('Phone')]
  print(result)

  # Output
    Product  Price  Stock     Category
  1   Phone    500     50  Electronics
  ```
- The `query()` method allows you to use SQL-like syntax to filter rows. For example:
  ```python
  result = df.query('Price > 300 and Stock < 20')
  print(result)

  # Output
    Product  Price  Stock     Category
  0  Laptop   1000     10  Electronics
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Student': ['John', 'Anna', 'Mike', 'Sara', 'Tom'],
      'Marks': [85, 78, 92, 65, 88],
      'Grade': ['A', 'B', 'A', 'C', 'A'],
      'Subject': ['Math', 'Science', 'English', 'Math', 'History']
  }
  df = pd.DataFrame(data)
  print(f"Original DataFrame:\n{df}")
  print(f"\nStudents with Marks greater than 80:\n{df[df['Marks'] > 80]}")
  print(f"\nStudents with Grade equal to 'A':\n{df[df['Grade'] == 'A']}")
  print(f"\nStudents with Marks between 50 and 90:\n{df[df['Marks'].between(50, 90)]}")
  print(f"\nStudents where Subject contains 'Math':\n{df[df['Subject'].str.contains('Math')]}")
  ```
## Functions
- Functions in pandas allow you to apply custom logic and calculations to a data frame or series. You can:
  - Perform row-wise or column-wise operations.
  - Apply transformations to specific columns.
  - Create new columns based on existing ones.
  - Clean and modify data using custom logic.
  - Automate repetitive tasks.
  - Make data transformations scalable and efficient.
  - Apply complex logic to data without writing loops.
- Types of Functions in Pandas:
  - `apply()` - Applies a function to each row or column. Useful for applying complex functions on a row or column.
  - `map()` - Applies a function to each element of a series (column). Useful for simple, element-wise operations.
  - `applymap()` - Applies a function element-wise to the entire data frame.
  - `transform()` - Applies a function to a column and returns a transformed data frame.
- Using `apply()`:
  ```python
  import pandas as pd

  # Sample data
  data = {
      'Product': ['Laptop', 'Phone', 'Tablet', 'Monitor', 'Keyboard'],
      'Price': [1000, 500, 300, 150, 50],
      'Stock': [10, 50, 30, 5, 100]
  }

  df = pd.DataFrame(data)

  # Increase the price by 10%
  df['New_Price'] = df['Price'].apply(lambda x: x * 1.10)
  print(df)

  # Output
    Product  Price  Stock  New_Price
  0   Laptop   1000     10     1100.0
  1    Phone    500     50      550.0
  2   Tablet    300     30      330.0
  3  Monitor    150      5      165.0
  4 Keyboard     50    100       55.0
  ```
- Using `map()`:
  ```python
  # Convert price to string format
  df['Price_Str'] = df['Price'].map(lambda x: f"${x}")
  print(df)

  # Output
    Product  Price  Stock  Price_Str
  0   Laptop   1000     10     $1000
  1    Phone    500     50      $500
  2   Tablet    300     30      $300
  3  Monitor    150      5      $150
  4 Keyboard     50    100       $50
  ```
- Using `applymap()`:
  ```python
  # Convert all numeric values to strings
  result = df[['Price', 'Stock']].applymap(str)
  print(result)

  # Output
    Price Stock
  0  1000    10
  1   500    50
  2   300    30
  3   150     5
  4    50   100
  ```
- Using `transform()`:
  ```python
  # Double the price
  df['Double_Price'] = df['Price'].transform(lambda x: x * 2)
  print(df)

  # Output
    Product  Price  Stock  Double_Price
  0   Laptop   1000     10         2000
  1    Phone    500     50         1000
  2   Tablet    300     30          600
  3  Monitor    150      5          300
  4 Keyboard     50    100          100
  ```
- Creating a Custom Function:
  ```python
  # Calculate the total value of stock
  def total_value(row):
      return row['Price'] * row['Stock']

  df['Total_Value'] = df.apply(total_value, axis=1)
  print(df)

  # Output
    Product  Price  Stock  Total_Value
  0   Laptop   1000     10        10000
  1    Phone    500     50        25000
  2   Tablet    300     30         9000
  3  Monitor    150      5          750
  4 Keyboard     50    100         5000
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Product': ['Laptop', 'Phone', 'Tablet', 'Monitor', 'Keyboard'],
      'Price': [1000, 500, 300, 150, 50],
      'Stock': [10, 50, 30, 5, 100],
      'Category': ['Electronics', 'Electronics', 'Electronics', 'Electronics', 'Accessories']
  }
  df = pd.DataFrame(data)
  print(f"Original DataFrame:\n{df}")

  df['Discounted_Price'] = df['Price'].apply(lambda x: x * 0.90)
  print(f"\nDataFrame after adding Discounted_Price:\n{df}")

  df['Price_Str'] = df['Price'].map(lambda x: f"${x}")
  print(f"\nDataFrame after adding Price_Str:\n{df}")

  df[['Price', 'Stock']] = df[['Price', 'Stock']].applymap(str)
  print(f"\nDataFrame after converting numeric values to strings:\n{df}")

  def total_value(row):
      return float(row['Price']) * float(row['Stock'])
  df['Stock_Value'] = df.apply(total_value, axis = 1)
  print(f"\nDataFrame after adding Stock_Value:\n{df}")
  ```
  - Note `float(row['Price']) * float(row['Stock'])`. We can't do `float(row['Price'] * row['Stock'])`. We need to convert each value first, then multiply.

## Grouping and Aggregation
- Grouping and aggregation allows you to:
  - Group data based on specific columns.
  - Perform calculations like sum, average, and count on grouped data.
  - Summarize large datasets efficiently.
- Grouping helps organize data into categories while aggregation calculates meaningful insights from grouped data.
- Grouping and aggregation can be useful for several reasons, including:
  - Handling of large datasets efficiently.
  - Summarizing data and statistical analysis.
  - Understanding patterns and relationships in data.
- Key Methods for Grouping and Aggregation:
  - `groupby()` - Groups data based on values in one or more columns.
  - `sum()` -  Returns the sum of values for each group.
  - `mean()` - Returns the average of values for each group.
  - `count()` - Returns the count of non-null values for each group.
  - `max()` - Returns the maximum value for each group.
  - `min()` - Returns the minimum value for each group.
  - `agg()` - Allows multiple aggregate functions at once.
- Grouping Data Using `groupby()`:
  ```python
  import pandas as pd

  # Sample data
  data = {
      'Product': ['Laptop', 'Phone', 'Tablet', 'Monitor', 'Keyboard', 'Mouse', 'Headphones', 'Phone', 'Tablet', 'Laptop'],
      'Category': ['Electronics', 'Electronics', 'Electronics', 'Electronics', 'Accessories', 'Accessories', 'Accessories', 'Electronics', 'Electronics', 'Electronics'],
      'Price': [1000, 500, 300, 150, 50, 20, 80, 600, 250, 1100],
      'Stock': [10, 50, 30, 5, 100, 150, 50, 40, 20, 5]
  }

  df = pd.DataFrame(data)

  # Group by Category
  grouped = df.groupby('Category')
  ```
- Aggregation Using `sum()`:
  ```python
  # Sum of Price and Stock by Category
  result = df.groupby('Category').sum()
  print(result)

  # Output
              Price  Stock
  Category                
  Accessories    150    300
  Electronics   4250    155
  ```
- Aggregation Using `mean()`:
  ```python
  # Mean of Price and Stock by Category
  result = df.groupby('Category').mean()
  print(result)

  # Output
              Price  Stock
  Category                
  Accessories  50.0  100.0
  Electronics  708.3  25.83
  ```
- Counting Values Using `count()`:
  ```python
  # Count of records by Category
  result = df.groupby('Category').count()
  print(result)

  # Output
              Product  Price  Stock
  Category                        
  Accessories       3      3      3
  Electronics       7      7      7
  ```
- Minimum and Maximum Values:
  ```python
  # Maximum and minimum price by Category
  result = df.groupby('Category')['Price'].max()
  print(result)

  # Output
  Category
  Accessories     80
  Electronics   1100
  ```
- Performing Multiple Aggregations Using `agg()`:
  ```python
  # Find the sum and mean of Price and Stock by Category
  result = df.groupby('Category').agg({'Price': ['sum', 'mean'], 'Stock': ['sum', 'mean']})
  print(result)

  # Output
                Price             Stock          
                  sum       mean   sum     mean
  Category                                     
  Accessories     150  50.000000   300  100.000
  Electronics    4250  708.333333   155   25.833
  ```
- Grouping by Multiple Columns:
  ```python
  # Group by Category and Product
  result = df.groupby(['Category', 'Product']).sum()
  print(result)

  # Output
                      Price  Stock
  Category   Product                
  Accessories Headphones     80     50
              Keyboard       50    100
              Mouse          20    150
  Electronics Laptop       2100     15
              Monitor       150      5
              Phone        1100     90
              Tablet        550     50
  ```
- Comprehensive Example:
  ```python
  import pandas as pd

  data = {
      'Item': ['Apple', 'Banana', 'Orange', 'Milk', 'Cheese', 'Bread', 'Butter', 'Eggs', 'Apple', 'Banana'],
      'Category': ['Fruit', 'Fruit', 'Fruit', 'Dairy', 'Dairy', 'Bakery', 'Dairy', 'Dairy', 'Fruit', 'Fruit'],
      'Price': [1.2, 0.5, 0.8, 1.5, 2.0, 1.0, 1.8, 0.3, 1.1, 0.6],
      'Quantity': [10, 20, 15, 5, 2, 30, 3, 12, 8, 18]
  }
  df = pd.DataFrame(data)
  print(f"Original DataFrame:\n{df}")

  category_group = df.groupby('Category')
  sum_result = category_group.sum()
  print(f"\nSum of Price and Quantity by Category:\n{sum_result}")
  avg_result = category_group.mean()
  print(f"\nMean of Price and Quantity by Category:\n{avg_result}")
  count_result = category_group['Item'].count()
  print(f"\nCount of items in each category:\n{count_result}")
  max_result = category_group['Price'].max()
  print(f"\nMaximum Price in each category:\n{max_result}")
  min_result = category_group['Price'].min()
  print(f"\nMinimum Price in each category:\n{min_result}")
  agg_result = category_group.agg({'Price': ['sum', 'mean'], 'Quantity': ['sum', 'mean']})
  print(f"\nAggregated result (sum and mean of Price and Quantity):\n{agg_result}")
  ```

## Merging DataFrames
- Merging is used to combine two or more data frames based on common columns or indexes. It is similar to SQL joins, where tables are combined using a common key.
- The `merge()` method is used to perform different types of joins, including left, right, inner, and outer joins.
- Merging is important for several reasons, including:
  - Combines data from multiple sources into one data frame.
  - Enhances data completeness by adding missing information.
  - Useful for performing relational data analysis.
- Inner Join:
  ```python
  import pandas as pd

  # Sample data for DataFrame 1
  data1 = {
      'Customer_ID': [101, 102, 103, 104],
      'Name': ['Alice', 'Bob', 'Charlie', 'David'],
      'Location': ['New York', 'California', 'Texas', 'Florida']
  }

  # Sample data for DataFrame 2
  data2 = {
      'Customer_ID': [101, 103, 104, 105],
      'Purchase': ['Laptop', 'Tablet', 'Phone', 'Monitor'],
      'Amount': [1000, 300, 600, 150]
  }

  # Create DataFrames
  df1 = pd.DataFrame(data1)
  df2 = pd.DataFrame(data2)

  # Inner join
  result = pd.merge(df1, df2, on='Customer_ID', how='inner')
  print(result)

  # Output
    Customer_ID     Name    Location Purchase  Amount
  0          101    Alice    New York   Laptop    1000
  1          103  Charlie       Texas   Tablet     300
  2          104    David     Florida    Phone     600
  ```
- Left Join:
  ```python
  # Left join
  result = pd.merge(df1, df2, on='Customer_ID', how='left')
  print(result)

  # Output
    Customer_ID     Name    Location Purchase  Amount
  0          101    Alice    New York   Laptop  1000.0
  1          102      Bob  California      NaN     NaN
  2          103  Charlie       Texas   Tablet   300.0
  3          104    David     Florida    Phone   600.0
  ```
  - Missing values are filled with `NaN`.
- Right Join:
  ```python
  # Right join
  result = pd.merge(df1, df2, on='Customer_ID', how='right')
  print(result)

  # Output
    Customer_ID     Name    Location Purchase  Amount
  0          101    Alice    New York   Laptop  1000.0
  1          103  Charlie       Texas   Tablet   300.0
  2          104    David     Florida    Phone   600.0
  3          105      NaN         NaN  Monitor   150.0
  ```
- Outer Join:
  ```python
  # Outer join
  result = pd.merge(df1, df2, on='Customer_ID', how='outer')
  print(result)

  # Output
    Customer_ID     Name    Location Purchase  Amount
  0          101    Alice    New York   Laptop  1000.0
  1          102      Bob  California      NaN     NaN
  2          103  Charlie       Texas   Tablet   300.0
  3          104    David     Florida    Phone   600.0
  4          105      NaN         NaN  Monitor   150.0
  ```
  - Missing values are filled with `NaN`.
- When the column names vary between the two datasets, use the `left_on` and `right_on` parameters to match them. For example:
  ```python
  # Inner join
  result = pd.merge(df1, df2, left_on='Customer_ID', right_on='Customer_ID', how='inner')
  print(result)
  ```

## Pivot Tables
- A pivot table is used to summarize and analyze data in a data frame. It allows you to reorganize and group data to perform aggregate calculations such sum, average, and count.
- Pivot tables provide key benefits to data engineers and analysts, including:
  - Providing a quick and effective way to summarize data.
  - Grouping and aggregating data quickly.
  - Comparing different categories and calculating business insights.
- The syntax for creating a pivot table is as follows:
  ```python
  pd.pivot_table(data, 
                values='column_to_aggregate', 
                index='column_to_group', 
                columns='column_to_pivot', 
                aggfunc='function')
  ```
  - `data` - The data frame to pivot.
  - `values` - The column(s) to be aggregated. It tells the pivot table which column(s) contains the numerical figures to be aggregated. When omitted, all numerical columns are included by default.
  - `index` - The column(s) used to group the data. It groups the data by the unique values in these column(s). The unique values become rows in the resulting pivot table. When multiple columns are specified, the rows in the pivot table become hierarchical.
  - `columns` - The column(s) to pivot. The unique values become columns in the resulting pivot table. When multiple columns are specified, the columns in the pivot table become hierarchical.
  - `aggfunc` - The aggregate function to be applied.
- Basic Pivot Table:
  ```python
  import pandas as pd

  # Sample data
  data = {
      'Date': ['2025-01-01', '2025-01-01', '2025-01-02', '2025-01-02', '2025-01-03'],
      'Region': ['East', 'West', 'East', 'West', 'East'],
      'Product': ['Laptop', 'Tablet', 'Laptop', 'Tablet', 'Phone'],
      'Sales': [1000, 700, 1200, 800, 600]
  }

  # Calculate total sales by region and product
  pivot = pd.pivot_table(df, 
                        values='Sales', 
                        index='Region', 
                        columns='Product', 
                        aggfunc='sum', 
                        fill_value=0)

  print(pivot)

  # Output
  Product  Laptop  Phone  Tablet
  Region                        
  East       2200    600       0
  West          0      0    1500
  ```
  - The `index` is `Region` and `columns` is `Product` because we're trying to calculate total product sales in each region. `index` determines the rows of the resulting pivot table and `columns` determines the columns of the resulting pivot table.
  - The `fill_value` parameter at the end tells the pivot table how to replace missing values.
- Pivot Table With Multiple Aggregations:
  ```python
  pivot = pd.pivot_table(df, 
                        values='Sales', 
                        index='Region', 
                        columns='Product', 
                        aggfunc=['sum', 'mean'], 
                        fill_value=0)

  print(pivot)

  # Output
            sum              mean         
  Product Laptop Phone Tablet Laptop Phone Tablet
  Region                                      
  East     2200   600      0   1100   600      0
  West        0     0   1500      0     0    750
  ```
- Pivot Table With Margins (Totals):
  ```python
  pivot = pd.pivot_table(df, 
                        values='Sales', 
                        index='Region', 
                        columns='Product', 
                        aggfunc='sum', 
                        fill_value=0,
                        margins=True) # Adds Total

  print(pivot)

  # Output
  Product  Laptop  Phone  Tablet   All
  Region                              
  East       2200    600       0  2800
  West          0      0    1500  1500
  All        2200    600    1500  4300
  ```
  - `margins=True` Adds row and column totals.
- Pivot Table With Different Aggregations Per Column:
  ```python
  pivot = pd.pivot_table(df, 
                        values='Sales', 
                        index='Region', 
                        columns='Product', 
                        aggfunc={'Sales': ['sum', 'mean', 'count']}, 
                        fill_value=0)

  print(pivot)

  # Output
              Sales                    
                sum   mean count
  Product  Laptop Phone Tablet Laptop Phone Tablet Laptop Phone Tablet
  Region                                                        
  East       2200   600      0   1100   600      0     2     1     0
  West          0     0   1500      0     0    750     0     0     2
  ```
  - When a single string is passed to the `aggfunc` parameter, it applies that function to all `values`.
  - When a list is passed to the `aggfunc` parameter, it applies those functions to all `values`.
  - When a dictionary is passed to the `aggfunc` parameter, it maps the specified functions to the specified columns.

## Date and Time Handling
- Pandas provides powerful tools for handling date and time data, allowing for easy manipulation, extraction, and analysis of dates and times.
- Why Date and Time Handling is Important:
  - Enables time-based analysis and grouping.
  - Helps detect patterns over time.
  - Supports operations such as filtering, resampling, and shifting.
  - Essential for business intelligence and time-based forecasting.
- Pandas Date and Time Data Types
  - `datetime64` - stores date and time values
  - `Timedelta` - Represents differences between two dates or times.
  - `Period` - represents periods of time, such as a month or year.
- Date and time data can be created using the `pd.to_datetime()` method as follows:
  ```python
  import pandas as pd

  # Create date series
  dates = pd.to_datetime(['2025-01-01', '2025-02-15', '2025-03-20'])
  print(dates)

  # Output
  DatetimeIndex(['2025-01-01', '2025-02-15', '2025-03-20'], dtype='datetime64[ns]', freq=None)
  ```
- Extracting Components From Dates:
  ```python
  data = {'Date': pd.to_datetime(['2025-01-01', '2025-02-15', '2025-03-20'])}
  df = pd.DataFrame(data)

  # Extract year, month, and day
  df['Year'] = df['Date'].dt.year
  df['Month'] = df['Date'].dt.month
  df['Day'] = df['Date'].dt.day

  print(df)

  # Output
          Date  Year  Month  Day
  0 2025-01-01  2025      1    1
  1 2025-02-15  2025      2   15
  2 2025-03-20  2025      3   20
  ```
- Handling Date and Time:
  ```python
  data = {'DateTime': pd.to_datetime(['2025-01-01 08:30', '2025-02-15 14:45', '2025-03-20 19:15'])}
  df = pd.DataFrame(data)

  # Extract hour and minute
  df['Hour'] = df['DateTime'].dt.hour
  df['Minute'] = df['DateTime'].dt.minute

  print(df)

  # Output
              DateTime  Hour  Minute
  0 2025-01-01 08:30:00     8      30
  1 2025-02-15 14:45:00    14      45
  2 2025-03-20 19:15:00    19      15
  ```
- A range of dates can be created using the `pd.date_range()` method as follows:
  ```python
  date_range = pd.date_range(start='2025-01-01', periods=7, freq='D')
  print(date_range)

  # Output
  DatetimeIndex(['2025-01-01', '2025-01-02', '2025-01-03', '2025-01-04',
                '2025-01-05', '2025-01-06', '2025-01-07'],
                dtype='datetime64[ns]', freq='D')
  ```
- Time can be added or subtracted using the `pd.Timedelta()` method as follows:
  ```python
  df['Next Week'] = df['DateTime'] + pd.Timedelta(days=7)
  print(df)

  # Output
              DateTime  Hour  Minute            Next Week
  0 2025-01-01 08:30:00     8      30 2025-01-08 08:30:00
  1 2025-02-15 14:45:00    14      45 2025-02-22 14:45:00
  2 2025-03-20 19:15:00    19      15 2025-03-27 19:15:00
  ```
- Filtering Data Based on Date:
  ```python
  filtered = df[df['DateTime'] > '2025-02-01']
  print(filtered)

  # Output
              DateTime  Hour  Minute            Next Week
  1 2025-02-15 14:45:00    14      45 2025-02-22 14:45:00
  2 2025-03-20 19:15:00    19      15 2025-03-27 19:15:00
  ```

## Resampling and Frequency
- Resampling is the process of changing the frequency of time-series data. It helps in aggregating data over a specified time period, such as daily, weekly, or monthly. It can be used to calculate statistics such as sum, mean, and count.
  - Examples of resampling include converting hourly data to daily data or converting monthly data to quarterly data.
  - Increasing the frequency (e.g. monthly to daily) is known as upsampling.
  - Decreasing the frequency (e.g. daily to monthly) is known as downsampling.
  - Resampling can be useful for tasks such as analyzing data at different granularity (e.g. daily, weekly, monthly), filling missing data points, and smoothing out fluctuations in data.
- The `resample()` method in pandas is used for resampling data and has the following syntax:
  ```python
  df.resample('<frequency>').<aggregation function>()
  ```
  - `'D'` = Daily
  - `'W'` = Weekly
  - `'M'` = Monthly
  - `'Q'` = Quarterly
  - `'Y'` = Yearly
  - `'H'` = Hourly
  - `'T'` = Minute
  - `'S'` = Second
- Resampling Example:
  ```python
  import pandas as pd

  # Step 1: Create a fixed sample DataFrame
  data = {
      'DateTime': pd.date_range(start='2025-01-01', periods=10, freq='D'),
      'Sales': [100, 200, 150, 400, 300, 250, 500, 450, 600, 550]
  }

  df = pd.DataFrame(data)
  df.set_index('DateTime', inplace=True)

  print("Original DataFrame:")
  print(df)

  # Step 2: Resample Data (Sum of Sales per Week)
  weekly_sales = df.resample('W').sum()

  print("Weekly Sales (Sum):")
  print(weekly_sales)

  # Step 3: Resample Data (Average Sales per Month)
  monthly_sales = df.resample('M').mean()

  print("Monthly Sales (Mean):")
  print(monthly_sales)
  ```

## Time-Based Indexing
- Time-based indexing allows you to use a datetime column as an index in a data frame, allowing you to efficiently slice, filter, and resample the data based on time intervals. This can be very useful when working with time-series data for analysis, aggregation, and visualization.
- Time-based indexing can be useful for:
  - Efficiently querying and filtering data using date and time ranges.
  - Performing time-based operations like resampling, shifting, and rolling.
  - Analyzing trends over time.
- To enable time-based indexing, you need to:
  - Convert the date column in the data frame to `datetime` using `pd.to_datetime()` method.
  - Set that column as the data frame index using the `set_index()` method.
  ```python
  df['DateTime'] = pd.to_datetime(df['DateTime'])
  df.set_index('DateTime', inplace=True)
  ```
- Time-Based Indexing Example:
  ```python
  import pandas as pd

  # Step 1: Create a fixed sample DataFrame
  data = {
      'DateTime': pd.date_range(start='2025-01-01', periods=10, freq='D'),
      'City': ['New York', 'Chicago', 'Los Angeles', 'New York', 'Chicago',
              'Los Angeles', 'New York', 'Chicago', 'Los Angeles', 'New York'],
      'Sales': [1200, 800, 1500, 600, 750, 1700, 950, 1800, 500, 1300]
  }

  df = pd.DataFrame(data)

  # Step 2: Convert 'DateTime' column to DateTime format
  df['DateTime'] = pd.to_datetime(df['DateTime'])

  # Step 3: Set 'DateTime' column as the index
  df.set_index('DateTime', inplace=True)

  print("Original DataFrame:")
  print(df)

  # Step 4: Filter data for a specific date range
  filtered_df = df.loc['2025-01-03':'2025-01-07']
  print("\nFiltered Data (Between 2025-01-03 and 2025-01-07):")
  print(filtered_df)

  # Step 5: Access data by year, month, and day
  print("\nData for January 2025:")
  print(df.loc['2025-01'])

  # Step 6: Extract specific year and month using attributes
  df['Year'] = df.index.year
  df['Month'] = df.index.month

  print("\nDataFrame after extracting Year and Month:")
  print(df)
  ```

## Moving (Rolling) Averages
- Moving (rolling) averages are used to "smooth out" short-term fluctuations in time-series data to identify long-term trends. This can be especially useful for financial data, sales forecasting, and stock price analysis.
- A **moving** average is the average of a fixed number of previous data points in a time series. A moving average is calculated by taking the average of data points over a specific, fixed-size window. It helps reduce noise and show trends over time. There are two types of moving averages:
  - Small Moving Average (SMA) is the average over a fixed window.
  - Exponential Moving Average (EMA) is the same as an SMA, but gives more weight to recent data points.
- The `rolling()` method is used to calculate moving averages. This syntax is as follows:
  ```python
  df['rolling_average'] = df['column'].rolling(window=3).mean()
  ```
  - `rolling()` creates the fixed-size window over which the data will be aggregated.
  - `window=3` specifies the width of the window (the number of periods).
  - `mean()` specifies the type of aggregation being performed. In this case, average.
- Moving Averages Example:
  ```python
  import pandas as pd

  # Step 1: Create sample data
  data = {
      'Date': pd.date_range(start='2025-01-01', periods=10, freq='D'),
      'Sales': [200, 220, 210, 215, 250, 300, 280, 270, 260, 310]
  }

  df = pd.DataFrame(data)

  # Step 2: Set 'Date' column as the index
  df.set_index('Date', inplace=True)

  # Step 3: Calculate rolling average with a window size of 3
  df['Rolling_Avg'] = df['Sales'].rolling(window=3).mean()

  print("DataFrame with Rolling Average:")
  print(df)

  # Output
              Sales  Rolling_Avg
  Date                          
  2025-01-01    200          NaN
  2025-01-02    220          NaN
  2025-01-03    210        210.0
  2025-01-04    215        215.0
  2025-01-05    250        225.0
  2025-01-06    300        255.0
  2025-01-07    280        276.6
  2025-01-08    270        283.3
  2025-01-09    260        270.0
  2025-01-10    310        280.0
  ```
  - The first two averages are `NaN` because there aren't two previous data points with which to create a window of three data points.
  - The window size can affect how smooth the aggregated data is. Smaller windows are more responsive to changes and have more fluctuations. Larger windows lead to smoother trends but respond to changes more slowly.
- Applying an Exponential Moving Average:
  ```python
  # Calculate exponential moving average with span of 3
  df['EMA'] = df['Sales'].ewm(span=3, adjust=False).mean()
  print(df)

  # Output
              Sales  Rolling_Avg         EMA
  Date                                      
  2025-01-01    200          NaN  200.000000
  2025-01-02    220          NaN  210.000000
  2025-01-03    210   210.000000  210.000000
  2025-01-04    215   215.000000  212.500000
  2025-01-05    250   225.000000  231.250000
  2025-01-06    300   255.000000  265.625000
  2025-01-07    280   276.666667  272.812500
  2025-01-08    270   283.333333  271.406250
  2025-01-09    260   270.000000  265.703125
  2025-01-10    310   280.000000  287.851562
  ```
  - EMA gives more weight to recent data points and reacts faster to recent changes than SMA.
  - The `ewm()` method is used instead of the `rolling()` method.
  - `span=3` is used to calculate the decay factor, which is used when calculating the average. It does not represent the size of the window.
  - `adjust=False` tells pandas how to calculate the average.

## Shifting and Lagging
- Shifting is used to move data forward or backward in time. It's useful for comparing data over different periods, calculating percent change, and identifying trends.
- Shifting moves data along the time index by a specified number of periods.
- Lagging is a type of shifting where data is moved to a previous period.
- Shifting and lagging are applied in pandas using the `shift()` method, which has the following syntax:
  ```python
  df['shifted_column'] = df['column'].shift(periods=1)
  ```
  - `periods=1` specifies the number of periods to shift (positive for forward and negative for backwards).
  - `fill_value` specifies the value to fill missing data after shifting (optional).
- Shifting and Lagging Example:
  ```python
  import pandas as pd

  # Step 1: Create sample data
  data = {
      'Date': pd.date_range(start='2025-01-01', periods=10, freq='D'),
      'Sales': [200, 220, 210, 215, 250, 300, 280, 270, 260, 310]
  }

  df = pd.DataFrame(data)
  df.set_index('Date', inplace=True)

  # Step 2: Shift sales data by 1 period (lagging)
  df['Sales_Shifted_1'] = df['Sales'].shift(periods=1)

  # Step 3: Shift sales data by -1 period (leading)
  df['Sales_Leading_1'] = df['Sales'].shift(periods=-1)

  print("DataFrame with Shifted and Leading Data:")
  print(df)

  # Output
              Sales  Sales_Shifted_1  Sales_Leading_1
  Date                                            
  2025-01-01    200              NaN             220.0
  2025-01-02    220            200.0             210.0
  2025-01-03    210            220.0             215.0
  2025-01-04    215            210.0             250.0
  2025-01-05    250            215.0             300.0
  2025-01-06    300            250.0             280.0
  2025-01-07    280            300.0             270.0
  2025-01-08    270            280.0             260.0
  2025-01-09    260            270.0             310.0
  2025-01-10    310            260.0               NaN
  ```
  - The first value in `Sales_Shifted_1` is `NaN` because there is no previous value to "shift forward".
  - The last value in `Sales_Leading_1` is `NaN` because there is no proceeding value to "shift backwards".
  - The optional `fill_value` parameter could be used to set a default value to use instead of `NaN`.
- Filling Missing Values After Shifting
  ```python
  df['Sales_Shifted_Fill'] = df['Sales'].shift(periods=1, fill_value=0)
  print(df)

  # Output
              Sales  Sales_Shifted_Fill
  Date                               
  2025-01-01    200                   0
  2025-01-02    220                 200
  2025-01-03    210                 220
  2025-01-04    215                 210
  ```
- Calculating Differences Using Shifting
  ```python
  df['Sales_Diff'] = df['Sales'] - df['Sales'].shift(1)
  print(df)

  # Output
              Sales  Sales_Diff
  Date                         
  2025-01-01    200         NaN
  2025-01-02    220        20.0
  2025-01-03    210       -10.0
  2025-01-04    215         5.0
  2025-01-05    250        35.0
  ```
- Calculating Percentage Change Using Shifting
  ```python
  df['Sales_Percent_Change'] = df['Sales'].pct_change() * 100
  print(df)

  # Output
              Sales  Sales_Percent_Change
  Date                                  
  2025-01-01    200                  NaN
  2025-01-02    220                 10.0
  2025-01-03    210                 -4.55
  2025-01-04    215                  2.38
  2025-01-05    250                 16.28
  ```
  - The `pct_change()` method uses a `shift` value of 1 in this case, since the first value is `NaN`.

## Helpful Functions

### Cut Function
- The `cut()` method is used to segment and sort data values into "bins". This is useful for converting a continuously changing variable to one that changes over certain categories. For example, this method could convert ages into groups of age ranges. This is similar to partitioning in SQL.
- Example Usage of Cut Method
  ```python
  data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve'],
    'Age': [23, 35, 62, 45, 16]
  }
  df = pd.DataFrame(data)

  # Divide ages equally according to the value of the bins parameter
  df["Age_Group"] = pd.cut(df["Age"], bins = 4)

  # Dividing ages according to custom set of intervals
  bins = [0, 18, 35, 50, 100] # Divides into intervals of (0 - 18), (18 - 35), (35 - 50), and (50 - 100).
  labels = ["Child", "Young Adult", "Middle Aged", "Senior] # Assigns labels to each interval created above.
  df["Age_Group"] = pd.cut(df["Age"], bins = bins, labels = labels) # Instead of the values being those that are calculated by pandas, the labels are used as values.
  ```

### Explode Function
- The `explode()` method is used to transform each element of a list-like (iterable) row to its own, individual row, while replicating index values.
- Example Usage of Explode Method
  ```python
  data = {
    'article_id': [1, 2, 3],
    'title': ['Article 1', 'Article 2', 'Article 3'],
    'tags': [
      ['python', 'data science', 'machine learning'],
      ['python', 'programming'],
      ['data science', 'statistics]
    ]
  }
  df = pd.DataFrame(data)

  df.explode("tags") # Separates each element in tags list of each row into its own row, while replicating index values. For example, the method would create two additional rows for the tags list in the first row, so that each tag has its own row.
  ```
  - If you don't want to replicate index values, you can pass the optional parameter `ignore_index=True`.

### Shift Function
- The `shift()` method is used to shift an index by a desired number of periods with an optional time frequency. When the frequency is not specified, the indexes are shifted without realigning the data.
- Example Usage of Shift Method:
  ```python
  data = {
    'Sales': [1200, 2500, 4000, 5700, 8600],
    'Region_Code': [13, 23, 18, 33, 48],
    'Personnel': [12, 25, 37, 44, 63]
  }
  df = pd.DataFrame(data)

  df.shift(periods = 3) # Shifts the data frame rows downward by 3 periods, filling in missing values with NaN.

  df.shift(periods = 1, axis = 1) # Shifts the data frame columns to the right by 1, filling in missing values with NaN.

  df.shift(periods = 3, fill_value = 0) # Shifts the data frame rows downward by 3 periods, filling in missing values with 0.

  df[['Region_Code', 'Personnel']] = df[['Region_Code', 'Personnel']].shift(3) # Shifts the rows in the Region_Code and Personnel columns downward by 3 periods, filling in missing values with NaN.
  ```
  - To shift rows upward or columns to the left, you would use a negative number for the `periods` parameter.
  - The `shift()` method does not mutate the existing data frame and instead returns a new data frame.

### Apply Function
- The `apply()` method is used to apply a specified function along an axis (row or column) of a data frame.
- Example Usage of Apply Method:
  ```python
  data = {
    'first_name': ['Alice', 'Bob', 'Charlie', 'David', 'Mark', 'Irwin'],
    'last_name': ['Smith', 'Jones', 'Brown', 'Millar', 'Picard', 'Kent'],
    'score': [82, 91, 68, 75, 21, 32]
  }
  df = pd.DataFrame(data)

  def get_results(score):
    if score >= 35:
      return "Pass"
    else:
      return "Fail"

  df["result"] = df["score"].apply(get_results) # Applies the function to the score column, assigning the appropriate value, then storing the results in a new result column.

  df["score_squared"] = df["score"].apply(lambda x: x ** 2) # Applies the function to the score column, assigning the appropriate value, then storing the results in a new score_squared column.

  df["full_name"] = df.apply(lambda row: f"{row["first_name"]} {row["last_name"]}", axis = 1) # Applies the function to each row, assigning the appropriate value, then storing the results in a new full_name column.
  ```

### Melt Function
- The `melt()` method is used to unpivot a data frame from wide (more columns) to long (less columns) format, optionally leaving identifiers set.
- Example Usage of Melt Method:
  ```python
  data = {
    'student': ['Alice', 'Bob'],
    'math': [90, 78],
    'english': [85, 80],
    'history': [88, 70]
  }
  df = pd.DataFrame(data) # Wide format (columns for each subject).

  melted_df = pd.melt(
    df,
    id_vars = "Student", # Column which will remain the same in the new data frame.
    value_vars = ["Math", "English", "History"], # Columns which will become rows in the new data frame.
    var_name = "subject", # Name of the column header that will contain the older columns in the new data frame.
    value_name = "score" # Name of column header that will contain the values of the old columns in the new data frame.
  ) # Results in new data frame with student, subject, score columns in the long format.
  ```

### Pivot Function
- The `pivot()` method is used to return a data frame that is reshaped according to given index / column values.
- Example Usage of Pivot Method:
```python
data = {
  'month': ['Jan', 'Jan', 'Feb', 'Feb', 'Mar', 'Mar'],
  'product': ['Apple', 'Oranges', 'Apple', 'Oranges', 'Apple', 'Oranges',],
  'sales': [250, 200, 300, 220, 280, 210]
}
df = pd.DataFrame(data)

pivot_df = pd.pivot(
  df,
  index = "month", # The rows of the new data frame.
  columns = "product", # The columns of the new data frame (unique values in the column).
  values = "sales" # The values that will fill the elements of the new data frame.
) # Apples and Oranges become column headers and Jan, Feb, and Mar become index (row) labels.
```
