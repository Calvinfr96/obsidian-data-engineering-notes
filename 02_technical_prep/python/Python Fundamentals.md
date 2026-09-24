---
aliases:
  - "python_introduction"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Python and Pandas/python_introduction.md"
imported: 2026-09-24
---

# Python Fundamentals

Review: [[02_technical_prep/python/Python Data Engineering Toolkit|Python Data Engineering Toolkit]] · [[02_technical_prep/python/Python Algorithms Practice|Python Algorithms Practice]]

## Contents

- [[#Data Types]]
- [[#Variable Assignments]]
- [[#Lists]]
- [[#Strings]]
- [[#Dictionaries]]
- [[#Tuples]]
- [[#Sets]]
- [[#Booleans]]
- [[#Comparison and Logical Operators]]
- [[#Bitwise Operators]]
- [[#Control Flow Statements]]
- [[#Loops]]
- [[#Important Functions]]
- [[#List Comprehension]]
- [[#Functions]]
- [[#Object Oriented Programming (OOP)]]
- [[#Inheritance and Polymorphism]]
- [[#Magic Methods]]
- [[#Errors and Exceptions]]

## Data Types
- Integer (int) - Represents whole numbers.
- Floating Point (float) - Represents decimal numbers.
- String (str) - Represents an ordered, **immutable** sequence of characters.
- List (list) - Represents an ordered sequence of **mutable** primitives or objects, enclosed in square brackets. Ex: [2, 3.4, "5"]
- Dictionary (dict) - Represents an unordered sequence of key-value pairs, enclosed in curly braces. Ex: {"Key": "value", "Name": "John", "Age": 45}
- Tuple (tup) - Represents an ordered sequence of **immutable** objects, enclosed in parentheses. Once a tuple is created, it cannot be modified. Ex: (23, 45, "Hello", 100.0)
- Boolean (bool) - Represents a true or false value.
  - The `==` operator is used to determine equality of two objects and returns a boolean.
  - When converting an `int` to a `bool`, python uses the concept of truthiness. This means 0 will be converted to `False` and any other non-zero integer will be converted to `True`.
- The `type()` function can be used to determine the type of a primitive or object.

### Numbers
- Integers and floating point numbers are the two types of numbers in python.
- Arithmetic can be performed with integers, floats, or a combination of the two. For example, adding two integers will result in an integer. However, adding an integer and a float will result in a float.

## Variable Assignments
- Variables in python can contain letters, numbers, and underscores. A variable must start with either a letter or underscore, it cannot start with a number. For example:
  ```python
  name = "John"
  _name = "Jerry"
  last_name = "Doe"
  name2 = "Joe"
  ```
- Python variables are **case sensitive** and cannot contain spaces or special characters other than underscore.
- Python naming convention is to separate words in variables using underscores, instead of camel-case (i.e. `first_name` instead of `firstName`).
- The `print()` function can be used to print the value of a variable to the console.
- Mutable variables can be updated and assigned a value at the same time by combining arithmetic operators with assignment operators as follows: `+= | -= | *= | /= | //= | %= | **= |`. For example:
  ```python
  x = 2
  x += 5 # x now equals 7. Equivalent to using 'x = x + 5'.
  ```

## Lists
- A python list is a mutable collection of objects separated by commas and enclosed in square brackets. For example: `list = [1, 2, 3, 4, 5]`.
- Python uses zero-based indexing to refer to elements within lists. In the above example, 1 has an index of 0 and 5 has an index of 4. Python also uses negative indexes, which start with -1. In the above example, 1 has an index of -5 and 5 has an index of -1. Elements within a list can be referenced using the zero-based or negative index. For example, 1 can be referred to using either an index of 0 or -5.
  - One helpful example of the negative index is finding the last element in a list of unknown length. The index of the last element will always be -1.
  - To address elements in a list using their respective index, you use square brackets as follows: `list[3] = 4`.
- The `len()` function can be used to find the length of a list. For example: `len(list) = 5`.
- The `pop()` function can be used to remove the last element of a list. For example: `list.pop() = 5` and now `list = [1, 2, 3, 4]`.
- The `reverse()` function can be used to reverse the elements in a list. For example: `list.reverse() = [4, 3, 2, 1]`.
- The `append()` function can be used to add an element to the end of a list. For example: `list.append(5)` will result in `list = [1, 2, 3, 4, 5]`.

## Strings
- A python string is an ordered sequence of characters enclosed in either single or double quotes. As with lists, characters within a string can be referenced using zero-based or negative indexes. For example: `str = "Hello"` and `str[0] = 'H'` or `str[-5] = 'H'`.
- The escape character in python is `\`, just like Java. To create a new line within a string, you use `\n`. For example: `'Hello \nWorld'` will print 'Hello' and 'World' on two separate lines.
- Another commonly used escape character is `\t`, which is used to create a tab. For example: `'Hello\tWorld'` would print as `Hello  World`.
- Just as with lists, the `len()` function can be used to find the length of a string, including all spaces. For example: `len("Hello World") = 11`.

### String Slicing
- String slicing in python is a method used to extract a certain sequence of characters from a string. For example, if `str = "Hello World"`, then `str[:] = 'Hello World'`. However, `str[0:5] = Hello`. The range specified excludes the ending index.
- Another interesting way to slice strings in python involves  "jumping", where you specify an exclusive range of indexes, as well as how many characters you want to jump. For example: `str_1 = "I love python"` and `str[0:6:2] = "Ilv"` because the range include 'I love' and the "jump size" is two, so it starts with 'I', then includes every other character up to the 5th character.
- To reverse a string, you can use `::-1`. For example: `str[::-1] = "dlroW olleH"`.

### String Methods and Properties
- You can concatenate strings in python using the + symbol. For example: `str = "Hello " + "World"` would print `'Hello World'`.
- String concatenation can also be performed using the * symbol. For example: If `str = "Hello"` then `str * 3` would print `'HelloHelloHello'`
- Common String Methods (`str = "Hello World"`):
  - `upper()`: `str.upper() = 'HELLO WORLD'`.
  - `lower()`: `str.lower() = 'hello world'`.
  - `split()`: `str.split() = ['Hello', 'World']` because the default split character is a space. `str.split('o') = ['Hell', ' W', 'rld']`. 

### String Formatting
- String formatting in python is a method used to form strings dynamically based on the value of one or more variables.
- When printing a string, you use `{}` as a placeholder for the variable, along with the `format()` method. For example: `print("My name is {}".format("John"))` will print 'My name is John'. Similarly, `print("My name is {} and I am {} years old".format("John", 20))` will print 'My name is John and I am 20 years old'.
  - You can also use the index of elements in the format list to specify which value the placeholder should be replaced with. For example: `print("My name is {1} and I am {0} years old".format("John", 20))` would print 'My name is 20 and I am John years old' because John was listed first and 20 second.
  - Instead of using indexes, you can assign the elements in the format list to variables and use those. For example: `print("My name is {name} and I am {age} years old".format(name = "John", age = 20))` would print 'My name is John and I am 20 years old' because of the way we assigned the variables in the format list.
- Formatting floating point numbers in strings is done as follows:
  ```python
  number = 22/7 # 3.142857142857143
  print("My result is {number:1.3f}".format(number)) # number is optional in this case since it's the only value in the format list.
  # Prints '3.143' because the width (numbers before the decimal) is one and the precision (numbers after the decimal) is three.
  # The general form of this notation is {value:width.precision f} (leave out the space).
  ```
- F-String Method:
  ```python
  name = "John"
  print(f"My name is {name}") # Prints 'My name is John'
  # Simply placing an 'f' in front of the string enables formatting.
  ```

## Dictionaries
- A python dictionary is an unordered collection of key-value pairs enclosed in curly braces. For example: `d =  {"key1":"value1", "key2":"value2"}`.
- As with lists, you use square brackets to extract a specific value from a dictionary using the key. For example: `d['key1'] = 'value1'`. Similar to lists, dictionaries can also contain key-value pairs of any type. For example: If `d2 = {'k1':100, 'k2':'python', 'k3':[2, 3, 4]}`, then `d2['k3'][2] = 4`.
- Keys must be unique and immutable (strings, numbers, or tuples). It's typical for keys to be strings.
- The existence of a key can be checked using the `in` operator as follows:
  ```python
  if "age" in student:
      print("Age is available in the dictionary.")
  ```
- The `keys()` function of a dictionary will return a list of keys. For example: `d2.keys() = ['k1', 'k2', 'k3']`.
- The `values()` function of a dictionary will return a list of values. For example: `d2.values() = [100, 'python', [2, 3, 4]]`.
- The `items()` function of a dictionary will return a list of tuples, each of which represents a key-value pair. For example: `d2.items() = [('k1', 100), ('k2', 'python'), ('k3', [2, 3, 4])]`.
- A new key-value pair can be assigned to a dictionary as follows: `d2['k4'] = 25`. Now, `d2 = {'k1':100, 'k2':'python', 'k3':[2, 3, 4], 'k4':25}`. This method can also be used to modify the value of an existing key. For example: `d2['k1'] = 101`. Now, `d2 = {'k1':101, 'k2':'python', 'k3':[2, 3, 4], 'k4':25}`.
- The `get()` method accepts a key and a value to return if the key is not present in the map. For example: `d.get("key1", "Key is not present.")`.
- The `setdefault()` method accepts a key and a value and only sets the key's value if it does not exist.
- The `update()` method updates the dictionary with new key-value pairs. It accepts one argument in the form of key-value pairs enclosed in `{}`.
- The `pop()` method removes a specific key-value pair from the dictionary using the pair's key. For example: `d.pop("key1")`

## Tuples
- A python tuple is an immutable sequence of objects enclosed in parentheses. For example: `t = ('a', 'b', 'c', 'a', 'a', 'd', 'b')`.
- Tuples are more performant and use less memory than lists. They should be used in favor of lists unless mutability is required.
- The `count()` function tells you how many times an object appears in a tuple. For example: `t.count('a') = 3`.
- The `index()` function tells you the index of an element in a tuple. For example: `t.index('c') = 2`. For elements that appear multiple times, `index()` will return the index of the first occurrence. For example: `t.index('a') = 0`.
- Elements within a tuple are retrieved the same way they are for a list. For example: `t[4] = 'a'`.

## Sets 
- A python set is an unordered collection of **unique** elements, enclosed in curly braces.
- The `set()` function can be used to create an empty set. For example: `my_set = set()`.
  - This function can also be used to remove duplicate elements from a list. For example:
    ```python
    lst = [1, 1, 1, 2, 3, 4, 4, 5, 6, 7]
    s = set(list) # results in {1, 2, 3, 4, 5, 6, 7}
    ```
- The `add()` function can be used to add an element to a set. For example: `my_set.add(2)` will update the set from `{}` to `{2}`.
  - If we were to use this function to add 2 again, it wouldn't throw an error. It would simply have no effect on the set.
- The `list()` function can be used to convert a set to list. For example: `lst = list(my_set)` would result in a list of `[2]`.

## Booleans
- A python boolean is a true or false value that can be either `True` or `False`, or 1 or 0. They are mostly used in control flow of data.
- An example of something that returns a boolean value is a comparison operator. For example: `4>3 = True`.

## Comparison and Logical Operators
- Comparison operators in python include: `==, !=, >, >=. <, <=`.
- All comparison operators will return a boolean value. For example: `4 == 3` returns `False`.
- Comparison of strings in python is case sensitive. For example: `"Boy" == "boy"` returns `False` while `"Boy" == "Boy"` returns `True`.
- Logical operators in python include: `and`, `or`, as well as `not`. These operators can be used to chain comparison operators.
  - `and` requires that both boolean values be true for the expression to return true.
  - `or` requires that at least one boolean value to be true for the expression to return true.
  - `not` negates the boolean value of the expression.
- Example:
  ```python
  2<3 and 4<5 # true
  2>3 and 4>5 # false
  2>3 and 4<5 # false
  2<3 and 4>5 # false
  2<3 or 4<5 # true
  2>3 or 4>5 # false
  2>3 or 4<5 # true
  2<3 or 4>5 # true
  not(2<3 or 4<5) # false
  not(2>3 or 4>5) # true
  ```

## Bitwise Operators
- Bitwise operators in python work at the binary level. They manipulate individual bits of numbers.
- Bitwise operators include:
  - Bitwise AND (`&`): Returns `1` if both bits are `1`, otherwise `0`. For example:
    ```python
    12 ->   00001100
    13 -> & 00001101
            ________
            00001100 = 12
    ```
  - Bitwise OR (`|`): Returns `1` if either bit is `1`, otherwise `0`. For example:
    ```python
    12 ->   00001100
    13 -> | 00001101
            ________
            00001101 = 13
    ```
  - Bitwise XOR (`^`): Returns `1` if the bits are different, otherwise `0`. For example:
    ```python
    12 ->   00001100
    13 -> ^ 00001101
            ________
            00000001 = 1
    ```
  - Bitwise NOT (`~`): Flips all of the bits, converting `1` to `0` and `0` to `1`. For example:
    ```python
    12 -> ~ 00001100
            ________
            11110011 = -13
    ```
  - Bitwise LEFT SHIFT (`<<`): Shifts bits left by n positions (multiplies by 2^n). For example:
    ```python
    10 -> ~ 1010 << 2
            ____
            101000 = 40
    ```
  - Bitwise RIGHT SHIFT (`>>`): Shifts bits right by n positions (divides by 2^n). For example:
    ```python
    10 -> ~ 1010 >> 2
            ____
            10 = 2
    ```
  - We used four leading zeroes in the example above because we assumed a fixed bid-width of 8 bits. The number of leading zeroes in a binary number is the difference between this width and the position of the most significant bit, which is the first 1. Since the position of the most significant bit in `1100` and `1101` is 4, we used 4 leading zeroes.
  - Representing Negative Numbers in Binary (4-bit example):
    - Step 1: Convert number to binary: 6 becomes 0110
    - Step 2: Flip all of the bits: 0110 becomes 1001
    - Step 3: Add 1: 1001 + 1 = 1010
    - This means the representation depends on the bit width. It's not an absolute answer like in base 10.

## Control Flow Statements
- The three control flow statements in python are: `if`, `elif` and `else`. Control flow statements work as follows:
  ```python
  if boolean_expression_a:
    Code to execute if boolean_expression_a returns True.
  elif boolean_expression_b:
    Code to execute if boolean_expression_a returns False and boolean_expression_b returns True.
  elif boolean_expression_c:
    Code to execute if boolean_expression_a and boolean_expression_b return False and boolean_expression_c returns True.
  else:
    Code to execute by default.
  ```
  - You can have as many `elif` statements as you want, but only one `if` and one `else`.
  - The code will execute for the first boolean expression to return True, or if none return True and an `else` statement is provided, the code within the `else` statement. All statements after the first one to return True will not be evaluated.
  - The indentation is required in these statements.

## Loops
- As in other programming languages, loops in python are used to operate on iterable objects, such as lists and strings. There are `for` and `while` loops in python.
- For loop syntax:
  ```python
  for item in iterable:
    Code to execute for each item.
  ```
- For loop examples:
  ```python
  lst = [1, 2, 3]

  for item in lst:
    print(item) # Prints 1, 2, and 3 on separate lines (one line for each execution of the code block).
  ```
  ```python
  lst = [1, 2, 3, 4, 5, 6, 7, 8, 9]

  for item in lst:
    if item%2 == 0:
      print(item) # prints just the number for even numbers.
    else:
      print("{item} is an odd number".format(item)) # prints '{item} is an odd number' for odd numbers, where {item} is replaced by the value in the list.
  ```
  ```python
  phrase = "Hello World"

  for item in phrase:
    print(item) # prints each character (including spaces) in the string.
  ```
  ```python
  lst = [(1, 2), (3, 4), (5, 6), (7, 8)]

  for item in lst:
    print(item) # prints each tuple (ex: '(1, 2)') on a separate line (four prints in total).

  for (a, b) in lst:
    print(a, b) # prints the elements in each tuple separated by a space (ex: '1 2'. four prints in total).
  
  for (a, b) in lst:
    print(a) # prints the first value in each tuple
    print(b) # prints the second value in each tuple
    # There will be 8 prints in total printing 1 through 8 consecutively. The code block operates on each tuple independently.
    # This methodology can also be used when working with key-value pairs in dictionaries.
  ```
  ```python
  lst = [1, 2, 3, 4, 5, 6, 7, 8, 9]

  total_sum = 0
  for item in lst
    total_sum = total_sum + item # you can also write total_sum += item
  print("Total sum is", total_sum) # prints 'Total sum is 45' because the print statement is outside of the for loop (not indented).

  for item in lst:
    total_sum = total_sum + item
    print("Total sum is", total_sum) # prints the statement for each execution of the loop because the print statement is inside the for loop (is indented). 
  ```
- While loops in python repeat code based on a condition. Specifically, the code in a while loop will continue to execute until the provided boolean expression becomes False.
- When working with while loops, it's very important to avoid infinite loops, where the provided boolean expression never becomes False and the loop executes indefinitely.
- While loop syntax:
  ```python
  while boolean_expression:
    Code to execute as long as boolean_expression returns True
  else:
    Code to execute after boolean_expression returns False # optional and only executes once.
  ```
- While loop examples:
  ```python
  x = 0

  while x<5:
    print(x) # infinite loop because x will always be less than 5.
  
  while x<5:
    print(x) # prints 0, 1, 2, 3, and 4 (does not print 5 because 5<5 returns False).
    x = x + 1
  ```
- The `break` and `continue` keywords in python are used to interrupt the execution of a loop. `break` stops the entire future execution of the loop. `continue` skips all code below it and proceeds to the next execution of the loop. `pass` simply does nothing (does not interrupt execution of the code) and is often used as a placeholder for actual code within the loop (comments do not suffice). `pass` can also be used as a placeholder in other places, such as `if` statements. For example:
  ```python
  lst = [1, 2, 3, 4, 5]

  for item in lst:
    pass # no code within the loop is executed.
  ```
  ```python
  lst = [1, 2, 3, 4, 5]

  for item in lst:
    if item%2 == 0:
      continue # note the indentation required for both the for loop and if statement.
    print(item) # will only print odd numbers
  ```
  ```python
  x = 0
  while x < 5:
    if x > 3:
      break
    print(x) # prints 0, 1, 2 , and 3 because 4 > 3 and will cause the break statement to execute.
    x+=1
  ```

## Important Functions
- The `range()` function helps avoid needing to create a list of consecutive numbers. The syntax is `range(start, stop, step)`. Only one argument is required. For example:
  ```python
  for item in range(10):
    print(item) # prints numbers 1 through 9 because it is exclusive of the last number. The default step argument is 1 and the default start argument is 0.
  
  for item in range(3, 10):
    print(item) # prints numbers 3 through 9.
  
  for item in range(0, 10, 2) # prints even numbers.

  for item in range(10, 0, -1) # prints numbers 10 through 1 backwards. Negative number is required when start > stop.

  # range() can also be used to generate lists:
  lst = list(range(0, 10)) # creates a list of numbers 1 through 9.
  ```
- The `enumerate` function creates a series of tuples where each pair consists of an item's index in a list or string, followed by its value. For example:
  ```python
  test_string = "Hello_World"

  for item in enumerate(test_string):
    print(item) # prints (0, 'H'), (1, 'e'), (2, 'l), and so on. Each on a new line.
  ```
- The `zip()` function creates a series of tuples based on lists (typically of equal length, but not required) by placing the nth index of each list in the tuple. For example:
  ```python
  list_1 = [1, 2, 3]
  list_2 = ['a', 'b', 'c']
  list_3 = ['d', 'e', 'f']

  zipped_2_lists = list(zip(list_1, list_2)) # creates a list of [(1, 'a'), (2, 'b'), (3, 'c')]
  zipped_3_lists = list(zip(list_1, list_2, list_3)) # creates a list of [(1, 'a', 'd'), (2, 'b', 'e'), (3, 'c', 'f')]

  for i,j,k in zipped_3_lists:
    print(i,j,k) # prints the elements in each tuple separated by spaces.
    print(i) # prints the first element in each tuple.
    print(j) # prints the second element in each tuple.
    print(k) # prints the third element in each tuple.
  
  # If the lists are of unequal length, it will create tuples based on the length of the shortest list:
  list_a = [1, 2, 3, 4, 5]
  list_b = [1, 2, 3, 4]
  list_c = [1, 2, 3]

  zipped_3_lists = list(zip(list_a, list_b, list_c)) # creates a list of [(1, 1, 1), (2, 2, 2), (3, 3, 3)].
  ```
- The `in` function will check if an element is present in a list or string. For example:
  ```python
  'o' in "Hello World" # returns True.
  'a' in "Hello World" # returns False.
  1 in [1, 2, 3] # returns True.
  4 in [1, 2, 3] # returns False.
  'key' in {'key': 10} # returns True. Only checks keys in dictionaries, not values.
  'key1' {'key': 10} # returns False.
  10 in {'key': 10} # returns False.
  ```
- The `min()` and `max()` functions check the minimum and maximum numerical value in a list. For example:
  ```python
  min([2, 4, 6, 8, 10]) # returns 2
  max([2, 4, 6, 8, 10]) # returns 10
  ``` 
- The `input()` function will prompt input from the user and optionally store that string value in a variable. For example:
  ```python
  name = input('Enter name here: ') # if the user enters 'Jose' when prompted, then name will be assigned a value of 'Jose'. 
  ```
- The `randint()` function produces a random integer (requires importing a library). For example:
  ```python
  from random import randint

  number = randint(0,100) # assigns a random integer between 0 and 100 (exclusive).
  ```

## List Comprehension
- List comprehension in python provides a convenient (less readable) way to quickly create lists without using loops. For example, the following method can be used to turn a string into a list:
  ```python
  test = "Hello World"
  lst = []

  for i in test
    lst.append(i) # adds each character (including spaces) to lst.
  ```
  - The same thing can be done using  list comprehension, as follows:
  ```python
  string = "Hello World"
  lst = [item for item in string] # it's important that the for loop be within the square brackets to properly create the list.

  # instead of using the list() and range() functions to create a list, the same range function can be used with list comprehension as follows:

  lst = [i for i in range(0,10)] # creates a list of numbers 0 through 9.

  # you can also use if statements with list comprehension as follows:

  lst = [i for i in range(0,10) if i%2 == 0] # creates a list of even numbers from 0 to 9 (0, 2, 4, 6, 8).

  # you can also manipulate the variable outside of the for loop to modify the elements in the list:

  lst = [i**2 for i in range(0,10) if i%2 == 0] # creates a list of squares of even numbers from 0 to 9 (0, 4, 16, 36, 64).

  # if-else statements can also be used with list comprehension as follows (not recommended):

  lst = [i if i%2==0 else 'ODD' for i in range(0,10)] # creates a list of even numbers and the string 'ODD' replacing odd numbers.

  # using nested loops to create lists:

  lst = []

  for i in [1, 2, 3]:
    for j in [1, 10, 100]
      lst.append(i*j) # creates the following list: [1, 10, 100, 2, 20, 200, 3, 30, 300]
  
  # using nested loops with list comprehension:

  lst = [i*j for i in [1, 2, 3] for j in [1, 10 , 100]] # creates the following list: [1, 10, 100, 2, 20, 200, 3, 30, 300] 
  ```

## Functions
- Functions in python allow you to create clean, repeatable code that can be reused any number of times in your program.
- Functions are defined using the `def` keyword. The standard naming convention for functions in python is `snake_case`. For example:
  ```python
  def greet_hello():
    """
    we are greeting hello. # acts as function documentation
    """

    print("hello")
    return

  greet_hello() # executes the above function. As with loops, indentation is very important.
  ```
- Functions can also be given parameters to dynamically modify their execution:
  ```python
  def greet_hello(name):
    """
    we are greeting hello.
    """

    print("Hello " + name)
    return

  greet_hello("John") # executes the above function, printing 'Hello John'.
  greet_hello("Mike") # executes the above function, printing 'Hello Mike'.
  greet_hello("Sarah") # executes the above function, printing 'Hello Sarah'.
  ```
- The `return` keyword in a function can be used to pass a value to something else in your program, such as a variable. For example:
  ```python
  def sum_of_two_numbers(x, y):
    """
    Adds and returns x and y
    """

    return (x + y) # use parentheses to enclose return statement for enhanced readability.
  
  sum_of_two_numbers(3, 4) # returns 7.
  final_result = sum_of_two_numbers(3, 4) # returns 7 and stores it in final_result.
  ```
- The `return` keyword is not required if a function is not meant to return a value. For example:
  ```python
  def print_output(name):
    print("Hello {}".format(name))

  print_output("John") # prints 'Hello John' to the console.
  ```
- Function parameters can be given default values, making them optional. For example:
  ```python
  def print_output(name = "Mike"):
    print("Hello {}".format(name))

  print_output("John") # prints 'Hello John' to the console.
  print_output() # prints 'Hello Mike' to the console.
  ```
- Using logic within functions:
  ```python
  def is_even(number):
    return (number % 2 == 0)
  
  is_even(40) # returns True
  is_even(43) # returns False

  # alternatively, we could use two separate return statements:

  def is_even(number):
    if number % 2 == 0:
      return True
    else:
      return False
  
  is_even(48) # returns True
  is_even(57) # returns False  
  ```
  ```python
  def check_for_evens(lst)
    for i in lst:
      if i % 2 == 0:
        return True
    return False # notice the indentation here. This return statement is outside of the for loop to prevent the function from returning False when it encounters the first odd number in a list.
  
  check_for_evens([1, 2, 3, 4]) # returns True
  check_for_evens([1, 3, 5, 7]) # returns False
  ```
- If the return statement within a function is never executed, the function will simply return nothing. For example:
  ```python
  def check_for_evens(lst)
    for i in lst:
      if i % 2 == 0:
        return True
      else:
        pass
  check_for_evens([1, 2, 3, 4]) # returns True
  check_for_evens([1, 3, 5, 7]) # returns nothing
  ```
- Unpacking tuples within functions:
  ```python
  # individual elements within tuples can typically be accessed using loops as follows:

  stocks = [("AAPL", 300), ("GOOG", 400), ("MSFT". 500)]

  for stock, value in stocks:
    print(stock) # prints 'AAPL', 'GOOG', and 'MSFT'.
    print(value) # prints 300, 400, and 500.
  
  # individual elements within tuples can also be accessed with functions as follows:

  work_hours = [("Billy", 300), ("Sam", 400), ("Henry", 600)]

  def employee_of_month(work_hours):
    current_max_hours = 0
    current_employee_of_month = ""

    for name, hours in work_hours:
      if hours > current_max_hours:
        current_max_hours = hours
        current_employee_of_month = name
    
    return (current_employee_of_month, current_max_hours)

  employee_of_month(work_hours) # returns ("Henry", 600).
  employee,hours = employee_of_month(work_hours) # assigns "Henry" to employee and 600 to hours.
  ```
- A function can return multiple values as a tuple. For example:
  ```python
  def rectangle_properties(length, width):
      area = length * width
      perimeter = 2 * (length + width)
      return area, perimeter  # Returning two values

  # Storing multiple return values
  area_result, perimeter_result = rectangle_properties(5, 3)
  print(f"Area: {area_result}, Perimeter: {perimeter_result}")
  ```
- The `global` keyword can be used to refer to and modify global variables within a function. For example:
  ```python
  # Global variable
  count = 10

  def update_count():
      global count  # Declaring that we are using the global variable
      count += 5  # Modifying the global variable
  ```

### Interactions Between Functions
- In real python scripts, you will most likely have multiple functions interacting with one another to produce a final result. As an example, let's simulate the Three Cup Monte game, where you need to guess which cup an object is under after the cups have been shuffled:
  - Shuffle example:
  ```python
  from random import shuffle

  my_list = [2, 6, 5, 1, 8, 9]

  def shuffle_list(lst):
    shuffle(lst) # shuffle modifies the list in-place and returns None, it does not return a shuffled list.
  
  shuffle_list(my_list)
  print(my_list) # prints a shuffled list, such as [1, 6, 5, 2, 8, 9].
  ```
  - Implementation:
  ```python
  from random import shuffle

  my_list = ['   ', ' o ', '   ']

  def shuffle_list(lst):
    shuffle(lst)

  def check_guess():
    guess = ''

    guess = input('Pick a number from index 0, 1, or 2: ')
    while guess not in ['0', '1', '2']:
      guess = input('Pick a number from index 0, 1, or 2: ')
    
    return int(guess)
  
  def validate_guess(current_list, current_guess):

    if current_list[current_guess] == ' o ':
      return('Correct guess')
    else:
      return('Wrong guess')
  
  ## Input List
  my_list = ['   ', ' o ', '   ']

  ## Shuffle
  shuffle_list(my_list)

  ## Take Guess From User
  user_guess = check_guess()

  ## Check Guess
  validate_guess(my_list, user_guess)
  ```

### Lambda, Map, and Filter
- The `map()` function will apply a given input function to an iterable and returns an anonymous value that can be used in a loop or to construct a list. For example:
  ```python
  my_list = [1, 2, 3, 4, 5]

  def square_num(value):
    return (value ** 2) # parentheses not required, but improves readability.
  
  new_list = list(map(square_num, my_list)) # produces a list of [1, 4, 9, 16, 25] and assigns it to new_list. You cannot assign 'map(square_num, my_list)' directly to new_list.

  for i in map(square_num, my_list):
    print(i) # prints the squares of each number in the list.
  ```
- The `filter()` function will apply a given input value which **returns a boolean value** to a given input list and filter items based on those which return true when passed to the input function. For example:
  ```python
  names = ['Gloria', 'Allen', 'Bob', 'Steve']
  def check_string(string):
    if len(string)%2==0:
      return True
    else:
      return False
  
  even_strings = list(filter(check_string, names)) # returns ['Gloria'] since that was the only name with an even number of characters.
  ```
- The `lambda` keyword is used to create an anonymous function. For example:
  ```python
  def square_number(number):
    return number ** 2
  
  # the lambda equivalent of this would be:

  square = lambda: num:num**2 # both of the above functions would be called using 'square(n)'

  # lambda functions are not typically used as shown above, where you assign the function to a variable. They can be used where a function is required, but won't need to be used anywhere else. For example

  my_list = [1, 2, 3, 4]
  square_list = list(map(lambda: num:num**2, my_list)) # returns a list of [1, 4, 9, 16].
  ```
  - Lambda functions should only be used when the logic is very simple and easy to understand. You'll typically use them in place of 'one-liner' functions that just have a return statement. The return keyword is not required in the lambda function as shown above.

## Object Oriented Programming (OOP)
- OOP places more of a focus on objects, rather than functions, which improves code organization and reusability. Objects are used to combine data and behavior and classes act as the blueprint for objects.
- The `class` keyword is used to create classes in python and the standard naming convention for classes is `CamelCase`. A class is constructed as follows:
  ```python
  class Sample():
    def __init__(self, param1, param2): # this method acts as the class's constructor. The first parameter of all constructors must be 'self' because it ties the method to the class.
      self.param1 = param1
      self.param2 = param2
    
    def some_method(self):
      print(self.param1) # prints the first parameter of the class. The self keyword is used to bind the method to the class.
  
  my_sample = Sample() # creates an instance of the sample class and stores it in my_sample.
  my_sample.some_method() # calls the some_method function defined in the sample class.
  ```
- Dog class example:
  ```python
  class Dog():
    # Class Attributes (typically static values)
    species = "Mammal" # similar to a static constant in Java.

    def __init__(self, breed, name):
      self.breed = breed # this tells python to define a 'breed' attribute for the class and assign the value of 'breed' to it when a dog object is constructed. Conventionally, the attribute and parameter will have the same name, but it is not a strict requirement.
      self.name = name
    
    def bark(self, number):
      print("WOOF! I am {}. The number is {}.".format(self.name, number)) # as with the constructor, the 'self' keyword is the first parameter and is used to tie the method to the class.
  
  test_dog = Dog(breed = "Lab", name = "Sammy") # you can also just list the parameters values without equating them to the parameter names.
  test_dog.breed # returns 'Lab' because of how we defined the test_dog object.
  test_dog.name # returns 'Sammy' because of how we defined the test_dog object.
  test_dog.species # returns 'Mammal' because of how the attribute was defined in the class.
  test_dog.bark(10) # prints 'WOOF! I am Sammy. The number is 10.' because of how the method was defined in the class.
  type(test_dog) # returns __main__.dog
  ```
  - The `self.` in `self.name` of the `bark()` method is required so that python knows you're referring to the `name` attribute of the `dog` class.
  - The `number` parameter in the `bark` method did not need to be prefaced with `self.` because it is not a class attribute. This allows the method parameter to be independent of the class attribute, which will typically not change after the object is constructed.
- Circle class example:
  ```python
  class Circle():
    # Class Attributes
    pi = 3.141592654

    def __init__(self, radius = 1):
      self.radius = radius # as with other methods, you can use default parameters in the constructor
      self.area = Circle.pi * self.radius ** 2

    def circumference(self):
      return 2 * Circle.pi # self.radius 
  
  my_circle = Circle()
  my_circle.radius # returns 1 because that is the default parameter defined in the class.
  my_other_circle = Circle(5)
  my_other_circle.radius # returns 5 because that is the value passed to the constructor.
  my_circle.circumference # returns 6.28.
  ```
  - The `self.` prefix is required when referring to any class attribute. However, for static class attributes such as pi, you can also use the name of the class (i.e. `Circle.pi`)
  - The class attributes don't need to have a one-to-one relationship with the constructor arguments. You can have more attributes than arguments, such as with `self.area`.

## Inheritance and Polymorphism
- Inheritance in python, as in other programming languages, occurs when one class inherits methods and properties from another class. For example:
  ```python
  class Animal():
    def __init__(self):
      print('Creating Animal...')
    
    def who_am_i(self):
      print('I am an Animal')

    def speak(self):
      print('Animal Speaks')
  
  class Dog(Animal):
    def __init__(self):
      Animal.__init__(self) # prints 'Creating Animal...'
      print("I am a Dog!")
  
  my_dog = Dog() # prints 'Creating Animal...' and 'I am a Dog!'.
  my_dog.who_am_i() # inherits method from Animal class and prints 'I am an Animal'.
  my_dog.speak() # inherits method from Animal class and prints 'Animal Speaks'.
  ```
  - A class extends another class by placing the parent class in the parentheses next to the subclass name. In the example above, `Animal` was placed inside the parentheses next to `Dog`, so `Dog` extends `Animal`.
- When creating a subclass, you can override methods in the parent class as follows:
  ```python
  class Animal():
    def __init__(self):
      print('Creating Animal...')
    
    def who_am_i(self):
      print('I am an Animal')

    def speak(self):
      print('Animal Speaks')
  
  class Dog(Animal):
    def __init__(self):
      Animal.__init__(self) # prints 'Creating Animal...'
      print("I am a Dog!")
    
    def who_am_i(self):
      print("I am a good boy")
  
  my_dog = Dog()
  my_dog.who_am_i() # prints 'I am a good boy'.
  my_dog.speak() # still prints 'Animal Speaks' because the method has not been overridden in the subclass.
  ```
- Polymorphism in python, as in other programming languages, occurs when one class is used to create multiple subclasses. For example:
  ```python
  class Animal():
    def __init__(self):
      print('Creating Animal...')
    
    def who_am_i(self):
      print('I am an Animal')

    def speak(self):
      print('Animal Speaks')
  
  class Dog(Animal):
    def who_am_i(self):
      print("I am a Dog")

  class Cat(Animal):
    def who_am_i(self):
      print("I am a Cat")
  
  my_dog = Dog() # inherits constructor from Animal class.
  my_dog.who_am_i() # prints 'I am a Dog'.
  my_cat = Cat() # inherits constructor from Animal class.
  my_cat.who_am_i() # prints 'I am a Cat'.

  for pet in [my_dog, my_cat]:
    print(pet.who_am_i()) # prints 'I am a Dog' and 'I am a cat'.
  ```
  - Even though the `who_am_i()` method shares names across the different subclasses, it has different behavior based on which class the method is called on. This is what polymorphism is.

## Magic Methods
- Magic methods, also referred to as Dunder methods, in python are methods, such as init, that get called in the background. Other magic methods include those such as `len()` and `str()`. These can be called on generic types, such as lists and strings, but not on classes that are created, unless those methods are explicitly defined within the class. For example:
  ```python
  class Sample():
    def __init__(self):
      pass
  
  len(Sample()) # throws a type error because the Sample class has not defined a len() method.
  print(Sample()) # prints the memory address of the Sample object.
  str(Sample()) # simply encloses the memory address of the Sample object in quotes.

  class  Book():
    def __init__(self, title, author, pages):
      self.title = title
      self.author = author
      self.pages = pages

    def __str__(self):
      return "The author of {} is {}".format(self.title, self.author)
    
    def __len__(self):
      return self.pages
    
    def __del__(self):
      print("The Book is gone.")
  
  book_1 = Book("The Python Book", "Roger Wheelhouse", 100)
  book_string = str(book_1) # returns 'The author of The Python Book is Roger Wheelhouse' and assigns the value to book_string.
  print(book_1) # prints 'The author of The Python Book is Roger Wheelhouse'.
  len(book_1) # returns 100 because of the way the method was implemented in the Book class.
  del book_1 # removes book_1 from memory and prints 'The Book is gone.'
  ```
  - Note that, similar to the `__init__()` method, other magic methods such as `str()` have the same notation when being defined within a class. As with any other method, `self` is required to tie that implementation of the method to the class.
  - The `print()` method simply prints whatever gets returned by `str()` to the console, which is why we are able to print the same thing without defining `print()` in the `Book` class.
  - `del` is another magic method which removes a variable from memory. It will typically be called as: `del book_1` in the case of the `book_1` variable.

## Errors and Exceptions
- Similar to try-catch-finally in Java, python uses try-except-finally to handle errors that may occur during the execution of a program. For example:
  ```python
  try:
    result = 10 + '10' # results in a TypeError because the number will not be coerced into a string in python.
  except:
    print("You added incorrectly") # prints 'You added incorrectly' because an error was thrown during the execution of the code in the try block.
  ```
- Instead of generically catching all errors, you can catch specific errors as follows:
  ```python
  try:
    result = 10 + '10'
  except TypeError:
    print("You added incorrectly") # prints 'You added incorrectly' because a TypeError was encountered during the execution of the code in the try block.
  except:
    print("The addition was performed incorrectly") # will not print because the TypeError was caught first.
  else:
    print("Nice job adding") # will not print because the try block was not executed successfully.
  finally:
    print("This code will always run") # prints 'This code will always run' because the finally block executes regardless of whether an error is encountered in the try block.
  ```
  - The `else` block only executes when the try block executes successfully.
  - The `finally` block always executes. If an error is encountered, the `except` block for that error will be executed in addition to the `finally` block.
  - In typical data engineering code, you won't catch specific errors and will just use a generic except block to handle errors. You also typically won't use an `else` block and will use the `finally` block instead.
