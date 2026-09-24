---
aliases:
  - "python_practice_questions.py"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Python and Pandas/python_practice_questions.py"
imported: 2026-09-24
---

# Python Fundamentals Practice

Review: [[02_technical_prep/python/Python Algorithms Practice|Python Algorithms Practice]] · [[02_technical_prep/python/Python Data Engineering Toolkit|Python Data Engineering Toolkit]]

## Contents

- [[#if-elif-else Block Example]]
- [[#Cool Example of How to Avoid Jumping to a New Line When Doing Consecutive Print Statements]]
- [[#Cool "number pyramid" Example Using Nested Loops]]
- [[#String Slicing]]
- [[#String Methods upper(), lower(), and title()]]
- [[#Using f-strings Instead of .format()]]
- [[#List Methods]]
- [[#Working With Tuples]]
- [[#Dictionary Practice Problem]]
- [[#Modifying Global Variables Practice Problem]]
- [[#Exception Handling Practice Problem]]
- [[#Basic OOP Practice Problem]]
- [[#Using Infinity / Negative Infinity]]
- [[#Using sum() to Count Instead of For Loop]]
- [[#Counting Word Frequencies Practice Problem]]

## if-elif-else Block Example

```python
condition1 = False
condition2 = False
condition3 = True
if condition1:
    # Code to execute if condition1 is True
    pass
elif condition2:
    # Code to execute if condition1 is False and condition2 is True
    pass
elif condition3:
    # Code to execute if condition1 and condition2 are False but condition3 is True
    pass
else:
    # Code to execute if none of the above conditions are True
    pass
```

## Cool Example of How to Avoid Jumping to a New Line When Doing Consecutive Print Statements

```python
for i in range(3):  # Outer loop runs 3 times
    for j in range(3):  # Inner loop runs 3 times per outer iteration
        print("*", end=" ")  # Print * without a newline
    print()  # Move to the next line after inner loop ends (print does not require an argument). prints:
    """
    * * *
    * * *
    * * *
    """
```

## Cool "number pyramid" Example Using Nested Loops

```python
for i in range(1, 6):  # Outer loop controls rows
    for j in range(1, i + 1):  # Inner loop prints numbers up to i
        print(j, end=" ")
    print()  # Move to the next line

# Cool Multiplication Table Example Using Nested Loops:
for i in range(1, 6):  # Outer loop (1 to 5)
    for j in range(1, 6):  # Inner loop (1 to 5)
        print(i * j, end="\t")  # Print multiplication result
    print()  # Move to the next line

# Nested loops Practice Problem:
for i in range(1, 6):
    for j in range(i):
        print("*", end=" ")
    print()
```

## String Slicing

```python
text = "Python"
# format: string[start:end:step]
print(text[::2])   # Every second character (Pto)
print(text[1::2])  # Every second character starting from index 1 (yhn)
print(text[::-1])  # Reverse string (nohtyP)
print(text[5:0:-1]) # Only prints 'nohty'. Must use above expression to reverse full string.
# As with range, when the step is negative, start > end is required
```

## String Methods upper(), lower(), and title()

```python
text = "hello world"

print(text.upper())   # Convert to uppercase
print(text.lower())   # Convert to lowercase
print(text.title())   # Capitalize each word (i.e. 'Hello World')
print('Hello World'.swapcase()) # Reverses the case of each character (i.e hELLO wORLD)

# Removing Extra Spaces:
text = "  Python  "
print(text.strip())   # Removes spaces from both sides
print(text.lstrip())  # Removes spaces from the left
print(text.rstrip())  # Removes spaces from the right

# Replacing Text in a String:
text = "I love Java"
new_text = text.replace("Java", "Python") # Returns original string when replacement is not found
print(new_text) # Prints I love Python

# Splitting and Joining Strings:
sentence = "Python is easy to learn"
words = sentence.split()  # Splits at spaces by default
print(words)

words = ['Python', 'is', 'awesome']
sentence = " ".join(words)
print(sentence)

# Finding a String in a Substring:
text = "Data Engineering"
index = text.find("Engine") # Returns the index of the first character. In this case, 5
print(index)
```

## Using f-strings Instead of .format()

```python
name = "Alice"
age = 25
print(f"My name is {name} and I am {age} years old.") # Prints My name is Alice and I am 25 years old

pi = 3.14159
print(f"Pi rounded to 2 decimal places: {pi:.2f}") # The width is optional, but the precision is required

# Formatting Strings Using % Formatting (Old Method - Less Recommended):
name = "Emma"
score = 95
print("Student: %s, Score: %d" % (name, score))

# Formatting Strings Practice Problem:
item = "Laptop"
price = 899.99

print(f"The {item} costs ${price}")
print("{} price: ${}".format(item, price))
print("Price of %s: $%.2f" % (item, price)) # Most common placeholders are %s, %f, and %d
```

## List Methods

```python
fruits = ["apple", "banana"]
fruits.append("cherry")
print(fruits) # Prints ['apple', 'banana', 'cherry']

colors = ["red", "blue", "green"]
colors.insert(1, "yellow")  # Insert at index 1
print(colors) # Prints ['red', 'yellow', 'blue', 'green']

numbers = [10, 20, 30, 20, 40]
numbers.remove(20)  # Removes the first 20
print(numbers) # Prints [10, 30, 20, 40]

animals = ["cat", "dog", "rabbit"]
removed_animal = animals.pop(1)  # Removes 'dog'. Argument is optional and defaults to the last element
print("Updated List:", animals)
print("Removed Element:", removed_animal)

items = ["pen", "pencil", "eraser"]
items.clear()
print(items) # Prints []

numbers = [5, 2, 9, 1, 7]
numbers.sort()
print(numbers) # Prints [1, 2, 5, 7, 9]
numbers.sort(reverse=True)
print(numbers) # Prints [9, 7, 5, 2, 1]

letters = ["a", "b", "c", "d"]
letters.reverse()
print(letters) # Prints ['d', 'c', 'b', 'a']

original = [1, 2, 3]
copied = original.copy()
copied.append(4)
print("Original:", original) # Prints Original: [1, 2, 3]
print("Copied:", copied) # Prints Copied: [1, 2, 3, 4]
```

## Working With Tuples

```python
single_element_tuple = ("hello",)  # Comma is required
print(type(single_element_tuple))  # Prints ("hello",)

# Printing Elements of a Tuple in Reverse Order:
nums = (1, 2, 3, 4, 5)

for i in range(len(nums) - 1, -1, -1):
    print(nums[i])

# Slicing and Indexing:
numbers = (10, 20, 30, 40, 50) # Slicing and indexing work the same here as they do with lists
print(numbers[1:4])   # Elements from index 1 to 3
print(numbers[:3])    # First three elements
print(numbers[2:])    # Elements from index 2 to the end

# Counting Occurrences:
numbers = (1, 2, 3, 2, 4, 2)
print(numbers.count(2))  # Number of times 2 appears

# Find First Index of an Element:
animals = ("cat", "dog", "rabbit", "dog")
print(animals.index("dog"))  # First occurrence of "dog"
```

## Dictionary Practice Problem

```python
employee = {
    "name": "Alice",
    "department": "IT",
    "salary": 60000
}
print("Keys:", employee.keys())
print("Values:", employee.values())
print("Items:", employee.items())

employee.update({"salary": 65000, "position": "Developer"})
print("Updated Dictionary:", employee)

removed_value = employee.pop("department")
print("Removed Value:", removed_value)
print("Dictionary After Removal:", employee)

print("Is 'bonus' in dictionary?", "bonus" in employee)

# Dictionary Comprehension Example:
my_dict = {"a": 1, "b": 2, "c": 3}
reversed_dict = {value: key for key, value in my_dict.items()}
print(reversed_dict)

# Avoid Modifying a Dictionary Using copy():
def combine_dicts(dict1, dict2):
    merged = dict1.copy()
    merged.update(dict2)
    print("Merged Dictionary:", merged)

combine_dicts({"a": 1, "b": 2}, {"c": 3, "d": 4})
```

## Modifying Global Variables Practice Problem

```python
counter = 0

def increment_counter():
    global counter # Declare use of the global variable within the function
    counter += 1 # Modify the global variable
    return counter # Return the modified global variable. All three must be done on separate lines

increment_counter()
increment_counter()
increment_counter()
print(f"Final value of counter: {counter}")
```

## Exception Handling Practice Problem

```python
def invalid_index():
    try:
        test_list = [10, 20, 30]
        test_list[4]
    except IndexError as e: # Typically won't specify error in real code
        print(f"Error: {e}")

invalid_index()

# Using The Exception Error to Catch Errors Generically:
try:
    num = int(input("Enter a number: "))
    result = 10 / num
except Exception as e:
    print("An error occurred:", e)  # Prints the actual error message

# Performing Tasks After Program Execution:
def divide_100(num):
    try:
        print(f"Result: {100/num}")
    except ValueError as e1:
        print(e1)
    except ZeroDivisionError as e2:
        print(e2)
    finally:
        print("End of program") # Runs regardless of error being thrown

divide_100(23)

# Handling Multiple Exceptions Using One Except Block:
try:
    num = int(input("Enter a number: "))  
    result = 10 / num  
    print("Result:", result)
except (ValueError, ZeroDivisionError) as e:
    print("Error:", e)  # This will print the actual error message
```

## Basic OOP Practice Problem

```python
class Person():
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def introduce(self):
        print(f"Name: {self.name}, Age: {self.age}")

alice = Person("Alice", 25)
bob = Person("Bob", 30)

alice.introduce()
bob.introduce()

# Polymorphism Practice Problem:
class Vehicle: # Parentheses only required when extending a class
    def __init__(self, brand):
        self.brand = brand
    
    def start_engine(self):
        print("Vehicle engine started")

class Car(Vehicle):
    def start_engine(self):
        print("Car engine started")

class Bike(Vehicle):
    def start_engine(self):
        print("Bike engine started")

car = Car("Toyota")
bike = Bike("Harley Davidson")

car.start_engine()
bike.start_engine()
```

## Using Infinity / Negative Infinity

```python
dict = {'a': 10, 'b': 20, 'c': 30}
highest_value = float('-inf')
current_key = ''

for key, value in dict.items():
    if value > highest_value:
        highest_value = value
        current_key = key

print(current_key)
```

## Using sum() to Count Instead of For Loop

```python
s = "HelloWorld"
uppercase_count = sum(1 for char in s if char.isupper())
lowercase_count = sum(1 for char in s if char.islower())
print("Uppercase letters:", uppercase_count)
print("Lowercase letters:", lowercase_count)
```

## Counting Word Frequencies Practice Problem

```python
sentence = "hello world hello"
words = sentence.split()
word_freq = {}
for word in words:
    word_freq[word] = word_freq.get(word, 0) + 1
print(word_freq)

# Alternative:
def count_words(sentence):
    frequencies = {}
    words = sentence.split()
    
    for word in words:
        if frequencies.get(word, 0) == 0:
            frequencies[word] = 1
        else:
            frequencies[word] += 1
    
    print(frequencies)
    
count_words('hello world hello')
```
