# Comprehensive Python Reference Guide

## 1. BASICS & SYNTAX

### 1.1 Variables & Data Types
```python
# Variables (dynamic typing)
x = 5                    # int
y = 3.14                 # float
name = "Python"          # str
is_valid = True          # bool
nothing = None           # NoneType

# Type checking and conversion
type(x)                  # Returns <class 'int'>
isinstance(x, int)       # True
int("123")               # Convert to int
float(5)                 # Convert to float
str(123)                 # Convert to string
bool(0)                  # False (0, "", [], None are falsy)
```

### 1.2 Operators
```python
# Arithmetic
+ - * / // % **          # Add, Sub, Mul, Div, FloorDiv, Mod, Power

# Comparison
== != < > <= >=          # Equal, NotEqual, Less, Greater, LessEq, GreaterEq

# Logical
and or not               # Logical operators

# Assignment
= += -= *= /= //= %= **= # Assignment operators

# Identity & Membership
is, is not               # Identity operators
in, not in               # Membership operators

# Bitwise
& | ^ ~ << >>            # AND, OR, XOR, NOT, Left shift, Right shift

# Walrus operator (Python 3.8+)
if (n := len(data)) > 10:  # Assign and use in same expression
    print(f"List has {n} elements")
```

### 1.3 Input/Output
```python
# Input
name = input("Enter name: ")        # Returns string
age = int(input("Enter age: "))     # Convert to int

# Output
print("Hello", "World")             # Hello World
print("Value:", x, sep="-")         # Value-5
print("Line", end="")               # No newline
print(f"Name: {name}, Age: {age}")  # f-string (Python 3.6+)
print("Pi: {:.2f}".format(3.14159)) # Format string
```

## 2. DATA STRUCTURES

### 2.1 Strings
```python
# Creation
s = "Hello"
s = 'World'
s = """Multi
line"""
s = r"Raw\nstring"       # Raw string (ignores escape)

# String methods
s.upper()                # HELLO
s.lower()                # hello
s.strip()                # Remove whitespace
s.split(',')             # Split into list
s.replace('a', 'b')      # Replace characters
s.startswith('He')       # True
s.endswith('lo')         # True
s.find('l')              # Index of first occurrence (returns -1 if not found)
s.count('l')             # Count occurrences
s.isdigit()              # Check if all digits
s.isalpha()              # Check if all alphabetic
s.join(['a', 'b'])       # Join list with separator

# Indexing & Slicing
s[0]                     # First character
s[-1]                    # Last character
s[1:4]                   # Slice from index 1 to 3
s[::2]                   # Every 2nd character
s[::-1]                  # Reverse string

# Formatting
f"{name} is {age}"       # f-string
"{} {}".format(a, b)     # format method
"%s %d" % (name, age)    # Old style (deprecated)
```

### 2.2 Lists
```python
# Creation
lst = [1, 2, 3]
lst = list(range(5))     # [0, 1, 2, 3, 4]
lst = [x**2 for x in range(5)]  # List comprehension

# Methods
lst.append(4)            # Add to end
lst.extend([5, 6])       # Add multiple elements
lst.insert(0, 0)         # Insert at index
lst.remove(2)            # Remove first occurrence
lst.pop()                # Remove and return last
lst.pop(0)               # Remove and return at index
lst.clear()              # Remove all
lst.index(3)             # Find index of value
lst.count(2)             # Count occurrences
lst.sort()               # Sort in place
lst.reverse()            # Reverse in place
sorted(lst)              # Return sorted copy
len(lst)                 # Length
max(lst)                 # Maximum value
min(lst)                 # Minimum value
sum(lst)                 # Sum of elements

# Slicing
lst[1:3]                 # Elements 1 to 2
lst[::2]                 # Every 2nd element
lst[::-1]                # Reverse list

# List comprehension
[x**2 for x in range(10)]                    # Squares
[x for x in range(10) if x % 2 == 0]         # Even numbers
[x if x > 0 else 0 for x in [-1, 2, -3]]     # With condition
[[i+j for j in range(3)] for i in range(3)]  # Nested
```

### 2.3 Tuples
```python
# Creation (immutable)
tup = (1, 2, 3)
tup = 1, 2, 3            # Without parentheses
tup = (1,)               # Single element tuple
empty = ()               # Empty tuple

# Operations
tup[0]                   # Access by index
tup.count(2)             # Count occurrences
tup.index(3)             # Find index
len(tup)                 # Length
a, b, c = tup            # Unpacking
a, *rest = tup           # Extended unpacking

# Named tuples
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
p.x, p.y                 # Access by name
```

### 2.4 Sets
```python
# Creation (unique, unordered)
s = {1, 2, 3}
s = set([1, 2, 2, 3])    # {1, 2, 3}
empty_set = set()        # Empty set (not {})

# Methods
s.add(4)                 # Add element
s.update([5, 6])         # Add multiple
s.remove(2)              # Remove (raises error if not found)
s.discard(2)             # Remove (no error)
s.pop()                  # Remove and return arbitrary element
s.clear()                # Remove all

# Set operations
a | b                    # Union
a & b                    # Intersection
a - b                    # Difference
a ^ b                    # Symmetric difference
a.union(b)               # Union method
a.intersection(b)        # Intersection method
a.issubset(b)            # Check subset
a.issuperset(b)          # Check superset

# Frozen set (immutable)
fs = frozenset([1, 2, 3])
```

### 2.5 Dictionaries
```python
# Creation
d = {'a': 1, 'b': 2}
d = dict(a=1, b=2)
d = dict([('a', 1), ('b', 2)])

# Access & Modification
d['a']                   # Get value (raises KeyError if not found)
d.get('a')               # Get value (returns None if not found)
d.get('c', 0)            # Get with default value
d['c'] = 3               # Add/update key
d.setdefault('d', 4)     # Set if key doesn't exist
d.update({'e': 5})       # Update multiple

# Methods
d.keys()                 # Get all keys
d.values()               # Get all values
d.items()                # Get all (key, value) pairs
d.pop('a')               # Remove and return value
d.popitem()              # Remove and return last (key, value)
d.clear()                # Remove all
len(d)                   # Number of items
'a' in d                 # Check if key exists

# Dictionary comprehension
{x: x**2 for x in range(5)}
{k: v for k, v in d.items() if v > 0}

# Default dict
from collections import defaultdict
dd = defaultdict(int)    # Default value 0 for missing keys
dd['a'] += 1             # No KeyError

# OrderedDict (maintains insertion order, though regular dicts do too in Python 3.7+)
from collections import OrderedDict
od = OrderedDict([('a', 1), ('b', 2)])
```

## 3. CONTROL FLOW

### 3.1 Conditional Statements
```python
# if-elif-else
if condition:
    pass
elif other_condition:
    pass
else:
    pass

# Ternary operator
x = value_if_true if condition else value_if_false

# Match statement (Python 3.10+)
match value:
    case 1:
        print("One")
    case 2 | 3:
        print("Two or Three")
    case _:
        print("Other")
```

### 3.2 Loops
```python
# For loop
for i in range(5):           # 0 to 4
    print(i)

for i in range(1, 10, 2):    # 1, 3, 5, 7, 9
    print(i)

for item in collection:
    print(item)

for i, item in enumerate(collection):  # With index
    print(i, item)

for k, v in dict.items():    # Iterate dictionary
    print(k, v)

# While loop
while condition:
    pass

# Loop control
break                        # Exit loop
continue                     # Skip to next iteration
else:                        # Executes if loop completes normally
    print("Loop finished")

# Nested loops with labels (using exception)
try:
    for i in range(10):
        for j in range(10):
            if condition:
                raise StopIteration
except StopIteration:
    pass
```

## 4. FUNCTIONS

### 4.1 Function Basics
```python
# Basic function
def greet(name):
    return f"Hello, {name}"

# Default arguments
def greet(name="World"):
    return f"Hello, {name}"

# Multiple return values
def get_stats(data):
    return min(data), max(data), sum(data)

# Keyword arguments
def func(a, b, c):
    pass
func(a=1, c=3, b=2)

# Variable arguments
def func(*args):             # Tuple of positional args
    for arg in args:
        print(arg)

def func(**kwargs):          # Dict of keyword args
    for key, val in kwargs.items():
        print(key, val)

def func(*args, **kwargs):   # Both
    pass

# Positional-only and keyword-only (Python 3.8+)
def func(pos_only, /, standard, *, kw_only):
    pass
func(1, 2, kw_only=3)        # pos_only must be positional
```

### 4.2 Lambda Functions
```python
# Anonymous functions
square = lambda x: x**2
add = lambda x, y: x + y

# Common use with map, filter, sorted
list(map(lambda x: x**2, [1, 2, 3]))      # [1, 4, 9]
list(filter(lambda x: x > 0, [-1, 2, -3])) # [2]
sorted(data, key=lambda x: x['age'])       # Sort by key
```

### 4.3 Scope & Closures
```python
# Global vs Local
x = 10                       # Global
def func():
    global x                 # Modify global
    x = 20

def outer():
    x = 10
    def inner():
        nonlocal x           # Modify enclosing scope
        x = 20
    inner()
    return x

# Closures
def multiplier(n):
    def multiply(x):
        return x * n
    return multiply

times2 = multiplier(2)
times2(5)                    # 10
```

### 4.4 Decorators
```python
# Basic decorator
def decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@decorator
def say_hello():
    print("Hello")

# Decorator with arguments
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("Hello")

# Class decorators
class Decorator:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        print("Before")
        return self.func(*args, **kwargs)

# Common built-in decorators
@staticmethod                # Method doesn't need self
@classmethod                 # Method receives class as first arg
@property                    # Makes method accessible as attribute
```

### 4.5 Generators
```python
# Generator function
def count_up_to(n):
    i = 0
    while i < n:
        yield i
        i += 1

for num in count_up_to(5):
    print(num)

# Generator expression
gen = (x**2 for x in range(10))
next(gen)                    # Get next value

# Generator methods
gen.send(value)              # Send value to generator
gen.close()                  # Close generator
gen.throw(Exception)         # Throw exception
```

## 5. OBJECT-ORIENTED PROGRAMMING

### 5.1 Classes & Objects
```python
# Basic class
class Person:
    species = "Human"        # Class variable
    
    def __init__(self, name, age):
        self.name = name     # Instance variable
        self.age = age
    
    def greet(self):         # Instance method
        return f"Hi, I'm {self.name}"
    
    @classmethod
    def from_birth_year(cls, name, year):  # Class method
        return cls(name, 2025 - year)
    
    @staticmethod
    def is_adult(age):       # Static method
        return age >= 18

# Create object
p = Person("Alice", 30)
p.greet()

# Attributes
p.name                       # Access
p.age = 31                   # Modify
hasattr(p, 'name')           # Check if exists
getattr(p, 'name', 'Unknown') # Get with default
setattr(p, 'age', 32)        # Set attribute
delattr(p, 'age')            # Delete attribute
```

### 5.2 Inheritance
```python
# Single inheritance
class Student(Person):
    def __init__(self, name, age, grade):
        super().__init__(name, age)  # Call parent constructor
        self.grade = grade
    
    def greet(self):         # Override method
        return f"Hi, I'm {self.name}, grade {self.grade}"

# Multiple inheritance
class Teacher(Person):
    def teach(self):
        pass

class TeachingAssistant(Student, Teacher):
    pass

# Check inheritance
isinstance(obj, Class)       # Check if instance of class
issubclass(Child, Parent)    # Check if subclass

# Method Resolution Order (MRO)
Class.__mro__                # View MRO
Class.mro()                  # Method form
```

### 5.3 Special Methods (Magic/Dunder)
```python
class MyClass:
    def __init__(self):              # Constructor
        pass
    
    def __str__(self):               # str(obj), print(obj)
        return "String representation"
    
    def __repr__(self):              # repr(obj), debugging
        return "MyClass()"
    
    def __len__(self):               # len(obj)
        return 0
    
    def __getitem__(self, key):      # obj[key]
        return self.data[key]
    
    def __setitem__(self, key, val): # obj[key] = val
        self.data[key] = val
    
    def __delitem__(self, key):      # del obj[key]
        del self.data[key]
    
    def __iter__(self):              # iter(obj), for loops
        return iter(self.data)
    
    def __call__(self):              # obj()
        return "Called"
    
    def __eq__(self, other):         # obj == other
        return self.data == other.data
    
    def __lt__(self, other):         # obj < other
        return self.data < other.data
    
    def __add__(self, other):        # obj + other
        return MyClass(self.data + other.data)
    
    def __enter__(self):             # Context manager: with obj:
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        pass                         # Cleanup

# Comparison operators
__eq__  ==    __ne__  !=
__lt__  <     __le__  <=
__gt__  >     __ge__  >=

# Arithmetic operators
__add__  +    __sub__  -    __mul__  *    __truediv__  /
__floordiv__ //  __mod__ %  __pow__ **
__and__ &  __or__ |  __xor__ ^

# Unary operators
__neg__  -x   __pos__  +x   __invert__  ~x
```

### 5.4 Encapsulation
```python
class BankAccount:
    def __init__(self):
        self.public = "Public"
        self._protected = "Protected"    # Convention: single underscore
        self.__private = "Private"       # Name mangling: double underscore
    
    @property
    def balance(self):                   # Getter
        return self._balance
    
    @balance.setter
    def balance(self, value):            # Setter
        if value >= 0:
            self._balance = value
    
    @balance.deleter
    def balance(self):                   # Deleter
        del self._balance

# Access
acc = BankAccount()
acc.public                               # OK
acc._protected                           # OK (but convention says don't)
acc.__private                            # Error
acc._BankAccount__private                # OK (name mangling)
```

### 5.5 Abstract Classes
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def sound(self):
        pass
    
    @abstractmethod
    def move(self):
        pass

class Dog(Animal):
    def sound(self):
        return "Woof"
    
    def move(self):
        return "Run"

# Cannot instantiate abstract class
# animal = Animal()  # Error
dog = Dog()          # OK
```

### 5.6 Data Classes (Python 3.7+)
```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: int
    y: int
    z: int = 0                           # Default value

@dataclass(order=True, frozen=True)      # Comparable, immutable
class Person:
    name: str
    age: int
    email: str = field(default="", repr=False)  # Field options
    
    def __post_init__(self):             # After initialization
        self.email = self.email.lower()
```

## 6. FILE HANDLING

### 6.1 Reading & Writing Files
```python
# Basic file operations
f = open('file.txt', 'r')    # Open for reading
content = f.read()           # Read entire file
f.close()                    # Close file

# Context manager (recommended)
with open('file.txt', 'r') as f:
    content = f.read()       # Automatically closes

# File modes
'r'   # Read (default)
'w'   # Write (overwrite)
'a'   # Append
'x'   # Exclusive creation (fails if exists)
'b'   # Binary mode
't'   # Text mode (default)
'+'   # Read and write

# Reading methods
f.read()                     # Read entire file
f.read(10)                   # Read 10 characters
f.readline()                 # Read one line
f.readlines()                # Read all lines into list

# Writing methods
f.write("text")              # Write string
f.writelines(list)           # Write list of strings

# Iteration
with open('file.txt') as f:
    for line in f:           # Memory efficient
        print(line.strip())
```

### 6.2 File & Directory Operations
```python
import os
import shutil
from pathlib import Path

# OS module
os.getcwd()                  # Get current directory
os.chdir('/path')            # Change directory
os.listdir('.')              # List directory contents
os.mkdir('dir')              # Create directory
os.makedirs('dir/subdir')    # Create nested directories
os.remove('file.txt')        # Delete file
os.rmdir('dir')              # Delete empty directory
os.rename('old', 'new')      # Rename file/directory
os.path.exists('path')       # Check if exists
os.path.isfile('path')       # Check if file
os.path.isdir('path')        # Check if directory
os.path.getsize('file')      # Get file size
os.path.join('dir', 'file')  # Join path components
os.path.split('/path/file')  # Split into (dir, file)
os.path.splitext('file.txt') # Split into (name, ext)
os.path.abspath('.')         # Get absolute path

# Shutil module
shutil.copy('src', 'dst')    # Copy file
shutil.copytree('src', 'dst') # Copy directory
shutil.move('src', 'dst')    # Move file/directory
shutil.rmtree('dir')         # Delete directory tree

# Pathlib (modern approach)
path = Path('file.txt')
path.exists()                # Check if exists
path.is_file()               # Check if file
path.is_dir()                # Check if directory
path.read_text()             # Read file content
path.write_text('content')   # Write to file
path.mkdir(parents=True)     # Create directory
path.unlink()                # Delete file
path.parent                  # Parent directory
path.name                    # File name
path.stem                    # File name without extension
path.suffix                  # File extension
path.glob('*.txt')           # Find files matching pattern
path.rglob('*.txt')          # Recursive glob
```

### 6.3 Working with CSV & JSON
```python
import csv
import json

# CSV reading
with open('data.csv') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

# CSV with dictionaries
with open('data.csv') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['column_name'])

# CSV writing
with open('data.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Name', 'Age'])
    writer.writerows([['Alice', 30], ['Bob', 25]])

# CSV writing with dictionaries
with open('data.csv', 'w', newline='') as f:
    fieldnames = ['Name', 'Age']
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerow({'Name': 'Alice', 'Age': 30})

# JSON reading
with open('data.json') as f:
    data = json.load(f)

# JSON from string
data = json.loads('{"name": "Alice"}')

# JSON writing
with open('data.json', 'w') as f:
    json.dump(data, f, indent=4)

# JSON to string
json_str = json.dumps(data, indent=4)
```

## 7. EXCEPTION HANDLING

### 7.1 Try-Except-Finally
```python
# Basic exception handling
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# Multiple exceptions
try:
    result = int("abc")
except (ValueError, TypeError) as e:
    print(f"Error: {e}")

# Catch all exceptions
try:
    risky_operation()
except Exception as e:
    print(f"An error occurred: {e}")

# Finally block (always executes)
try:
    f = open('file.txt')
    data = f.read()
except FileNotFoundError:
    print("File not found")
finally:
    f.close()  # Always closes

# Else block (executes if no exception)
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Error")
else:
    print("Success")
finally:
    print("Cleanup")
```

### 7.2 Raising Exceptions
```python
# Raise exception
raise ValueError("Invalid value")

# Re-raise current exception
try:
    something()
except Exception:
    log_error()
    raise  # Re-raises the exception

# Custom exceptions
class CustomError(Exception):
    def __init__(self, message):
        self.message = message
        super().__init__(self.message)

raise CustomError("Something went wrong")
```

### 7.3 Common Built-in Exceptions
```python
Exception               # Base class for all exceptions
AttributeError          # Attribute not found
IOError                 # I/O operation failed
ImportError             # Import failed
IndexError              # Index out of range
KeyError                # Key not found in dictionary
KeyboardInterrupt       # User interrupted execution
MemoryError             # Out of memory
NameError               # Variable not found
OSError                 # Operating system error
RuntimeError            # Generic runtime error
StopIteration           # Iterator has no more items
SyntaxError             # Invalid syntax
TypeError               # Wrong type
ValueError              # Right type, wrong value
ZeroDivisionError       # Division by zero
FileNotFoundError       # File doesn't exist
PermissionError         # No permission to access
```

### 7.4 Context Managers
```python
# Using with statement
with open('file.txt') as f:
    content = f.read()

# Custom context manager (class)
class MyContext:
    def __enter__(self):
        print("Entering")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Exiting")
        return False  # False: propagate exceptions

# Custom context manager (function)
from contextlib import contextmanager

@contextmanager
def my_context():
    print("Setup")
    yield
    print("Cleanup")

with my_context():
    print("Inside context")
```

## 8. MODULES & PACKAGES

### 8.1 Importing
```python
# Import entire module
import math
math.sqrt(16)

# Import with alias
import numpy as np
np.array([1, 2, 3])

# Import specific items
from math import sqrt, pi
sqrt(16)

# Import all (not recommended)
from math import *

# Import from package
from package.module import function

# Relative imports (within package)
from . import module          # Same directory
from .. import module         # Parent directory
from ..package import module  # Sibling package
```

### 8.2 Creating Modules
```python
# mymodule.py
def greet(name):
    return f"Hello, {name}"

PI = 3.14159

class MyClass:
    pass

# Usage in another file
import mymodule
mymodule.greet("Alice")
```

### 8.3 Creating Packages
```python
# Directory structure:
# mypackage/
#   __init__.py
#   module1.py
#   module2.py
#   subpackage/
#     __init__.py
#     module3.py

# __init__.py (runs when package is imported)
from .module1 import function1
from .module2 import function2
__all__ = ['function1', 'function2']

# Usage
import mypackage
from mypackage import function1
from mypackage.subpackage import module3
```

### 8.4 Important Built-in Modules
```python
# Math module
import math
math.sqrt(16)      # Square root
math.ceil(4.3)     # Ceiling
math.floor(4.7)    # Floor
math.factorial(5)  # Factorial
math.gcd(12, 8)    # Greatest common divisor
math.sin(math.pi)  # Trigonometric functions
math.log(10)       # Natural logarithm
math.e, math.pi    # Constants

# Random module
import random
random.random()              # Random float [0.0, 1.0)
random.randint(1, 10)        # Random int [1, 10]
random.choice([1, 2, 3])     # Random element
random.shuffle(lst)          # Shuffle list in place
random.sample(lst, 3)        # Random sample without replacement
random.seed(42)              # Set random seed

# Datetime module
from datetime import datetime, date, time, timedelta
now = datetime.now()
today = date.today()
dt = datetime(2025, 10, 27, 14, 30)
dt.strftime('%Y-%m-%d %H:%M:%S')  # Format to string
datetime.strptime('2025-10-27', '%Y-%m-%d')  # Parse from string
dt + timedelta(days=7)       # Add time

# Collections module
from collections import Counter, defaultdict, deque, namedtuple
Counter([1, 1, 2, 3])        # Count occurrences
defaultdict(int)             # Dict with default values
deque([1, 2, 3])            # Double-ended queue
namedtuple('Point', ['x', 'y'])  # Named tuple

# Itertools module
import itertools
itertools.count(10)          # Infinite counter
itertools.cycle([1, 2, 3])   # Infinite cycle
itertools.repeat(5, 3)       # Repeat element
itertools.chain([1, 2], [3, 4])  # Chain iterables
itertools.combinations([1, 2, 3], 2)  # Combinations
itertools.permutations([1, 2, 3], 2)  # Permutations
itertools.product([1, 2], [3, 4])     # Cartesian product

# Functools module
from functools import reduce, partial, lru_cache
reduce(lambda x, y: x + y, [1, 2, 3, 4])  # Reduce
partial(func, arg1)          # Partial function application
@lru_cache(maxsize=128)      # Memoization decorator

# Re module (Regular expressions)
import re
re.match(r'pattern', string)     # Match at beginning
re.search(r'pattern', string)    # Search anywhere
re.findall(r'pattern', string)   # Find all matches
re.sub(r'pattern', 'repl', str)  # Replace matches
re.split(r'pattern', string)     # Split by pattern
re.compile(r'pattern')           # Compile pattern

# Os module (covered in File Handling section)
# Sys module
import sys
sys.argv                     # Command line arguments
sys.exit()                   # Exit program
sys.path                     # Module search path
sys.version                  # Python version

# Time module
import time
time.time()                  # Current timestamp
time.sleep(2)                # Sleep for seconds
time.strftime('%Y-%m-%d')    # Format time

# Copy module
import copy
copy.copy(obj)               # Shallow copy
copy.deepcopy(obj)           # Deep copy

# Pickle module (serialization)
import pickle
pickle.dump(obj, file)       # Serialize to file
pickle.dumps(obj)            # Serialize to bytes
pickle.load(file)            # Deserialize from file
pickle.loads(bytes)          # Deserialize from bytes
```

## 9. ADVANCED CONCEPTS

### 9.1 List/Dict/Set Comprehensions
```python
# List comprehension
[x**2 for x in range(10)]
[x for x in range(10) if x % 2 == 0]
[x if x > 0 else 0 for x in numbers]
[[i*j for j in range(3)] for i in range(3)]  # Nested

# Dict comprehension
{x: x**2 for x in range(5)}
{k: v for k, v in dict.items() if v > 0}

# Set comprehension
{x**2 for x in range(10)}
{x for x in range(10) if x % 2 == 0}

# Generator expression
(x**2 for x in range(10))    # Memory efficient
```

### 9.2 Iterators & Iterables
```python
# Iterator protocol
class MyIterator:
    def __init__(self, max):
        self.max = max
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < self.max:
            self.current += 1
            return self.current
        raise StopIteration

# Using iterators
iterator = iter([1, 2, 3])
next(iterator)               # Get next value
list(iterator)               # Convert to list

# Iterator tools
zip([1, 2], ['a', 'b'])      # [(1, 'a'), (2, 'b')]
enumerate(['a', 'b'])        # [(0, 'a'), (1, 'b')]
reversed([1, 2, 3])          # [3, 2, 1]
sorted([3, 1, 2])            # [1, 2, 3]
map(func, iterable)          # Apply function
filter(func, iterable)       # Filter elements
all([True, True, False])     # False (all truthy?)
any([False, False, True])    # True (any truthy?)
```

### 9.3 Map, Filter, Reduce
```python
from functools import reduce

# Map - apply function to all items
list(map(lambda x: x**2, [1, 2, 3]))  # [1, 4, 9]
list(map(str.upper, ['a', 'b']))      # ['A', 'B']

# Filter - keep items where function returns True
list(filter(lambda x: x > 0, [-1, 2, -3, 4]))  # [2, 4]

# Reduce - accumulate result
reduce(lambda x, y: x + y, [1, 2, 3, 4])  # 10
reduce(lambda x, y: x * y, [1, 2, 3, 4])  # 24

# Multiple iterables with map
list(map(lambda x, y: x + y, [1, 2, 3], [4, 5, 6]))  # [5, 7, 9]
```

### 9.4 *args and **kwargs
```python
# Variable positional arguments
def func(*args):
    for arg in args:
        print(arg)
func(1, 2, 3)

# Variable keyword arguments
def func(**kwargs):
    for key, val in kwargs.items():
        print(f"{key}: {val}")
func(a=1, b=2, c=3)

# Both together
def func(*args, **kwargs):
    print(args)              # Tuple
    print(kwargs)            # Dict
func(1, 2, a=3, b=4)

# Unpacking in function calls
nums = [1, 2, 3]
func(*nums)                  # Unpack list as args

params = {'a': 1, 'b': 2}
func(**params)               # Unpack dict as kwargs
```

### 9.5 Type Hints (Python 3.5+)
```python
from typing import List, Dict, Tuple, Set, Optional, Union, Any, Callable

# Basic type hints
def greet(name: str) -> str:
    return f"Hello, {name}"

# Collections
def process(nums: List[int]) -> int:
    return sum(nums)

def lookup(data: Dict[str, int]) -> int:
    return data['key']

# Multiple types
def parse(value: Union[int, str]) -> int:
    return int(value)

# Optional (can be None)
def find(data: List[int], target: int) -> Optional[int]:
    return data.index(target) if target in data else None

# Any type
def process(data: Any) -> Any:
    return data

# Callable
def apply(func: Callable[[int, int], int], x: int, y: int) -> int:
    return func(x, y)

# Type aliases
Vector = List[float]
def scale(vector: Vector, scalar: float) -> Vector:
    return [x * scalar for x in vector]

# Generic types
from typing import TypeVar, Generic
T = TypeVar('T')
class Stack(Generic[T]):
    def push(self, item: T) -> None:
        pass
    def pop(self) -> T:
        pass
```

### 9.6 Asynchronous Programming
```python
import asyncio

# Async function
async def fetch_data():
    await asyncio.sleep(1)   # Non-blocking sleep
    return "Data"

# Running async functions
asyncio.run(fetch_data())

# Multiple async tasks
async def main():
    task1 = asyncio.create_task(fetch_data())
    task2 = asyncio.create_task(fetch_data())
    result1 = await task1
    result2 = await task2

# Async iteration
async def async_generator():
    for i in range(5):
        await asyncio.sleep(0.1)
        yield i

async def consume():
    async for value in async_generator():
        print(value)

# Async context manager
class AsyncResource:
    async def __aenter__(self):
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        pass

# Gathering multiple tasks
results = await asyncio.gather(task1, task2, task3)

# Waiting with timeout
try:
    result = await asyncio.wait_for(fetch_data(), timeout=1.0)
except asyncio.TimeoutError:
    print("Timeout")
```

### 9.7 Context Variables (Python 3.7+)
```python
from contextvars import ContextVar

# Thread-safe context variable
request_id: ContextVar[str] = ContextVar('request_id', default='')

def process_request():
    request_id.set('12345')
    handle_request()

def handle_request():
    print(request_id.get())  # Access context variable
```

### 9.8 Metaclasses
```python
# Metaclass - class of a class
class Meta(type):
    def __new__(cls, name, bases, dct):
        # Modify class creation
        dct['class_id'] = id(cls)
        return super().__new__(cls, name, bases, dct)

class MyClass(metaclass=Meta):
    pass

# Using __init_subclass__ (simpler alternative)
class Base:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.subclass_name = cls.__name__
```

### 9.9 Descriptors
```python
# Descriptor protocol
class Descriptor:
    def __get__(self, obj, objtype=None):
        return self.value
    
    def __set__(self, obj, value):
        if value < 0:
            raise ValueError("Must be positive")
        self.value = value
    
    def __delete__(self, obj):
        del self.value

class MyClass:
    attr = Descriptor()

# Built-in descriptors: property, staticmethod, classmethod
```

### 9.10 Memory Management
```python
import gc
import sys

# Garbage collection
gc.collect()                 # Force garbage collection
gc.disable()                 # Disable automatic GC
gc.enable()                  # Enable automatic GC
gc.get_count()               # Get collection counts

# Reference counting
sys.getrefcount(obj)         # Get reference count

# Weak references
import weakref
ref = weakref.ref(obj)       # Create weak reference
obj = ref()                  # Dereference (returns None if collected)

# Slots (memory optimization)
class Point:
    __slots__ = ['x', 'y']   # Only allow these attributes
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

## 10. STANDARD LIBRARY HIGHLIGHTS

### 10.1 String Processing
```python
import string

# Constants
string.ascii_lowercase       # 'abcdefghijklmnopqrstuvwxyz'
string.ascii_uppercase       # 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
string.ascii_letters         # Lower + upper
string.digits                # '0123456789'
string.punctuation           # All punctuation characters

# String template
from string import Template
t = Template('Hello, $name!')
t.substitute(name='Alice')

# Text wrapping
import textwrap
textwrap.wrap(text, width=70)        # Wrap text
textwrap.fill(text, width=70)        # Wrap and join
textwrap.dedent(text)                # Remove indentation
```

### 10.2 Data Serialization
```python
# JSON (covered earlier)
import json

# Pickle (covered earlier)
import pickle

# Marshal (internal Python objects)
import marshal
marshal.dumps(obj)
marshal.loads(bytes)

# Shelve (persistent dictionary)
import shelve
with shelve.open('mydata') as db:
    db['key'] = value
    value = db['key']
```

### 10.3 Compression
```python
import gzip
import zipfile
import tarfile

# Gzip
with gzip.open('file.gz', 'wt') as f:
    f.write('content')
with gzip.open('file.gz', 'rt') as f:
    content = f.read()

# Zip
with zipfile.ZipFile('archive.zip', 'w') as z:
    z.write('file.txt')
with zipfile.ZipFile('archive.zip', 'r') as z:
    z.extractall('destination')
    z.extract('file.txt')

# Tar
with tarfile.open('archive.tar.gz', 'w:gz') as tar:
    tar.add('file.txt')
with tarfile.open('archive.tar.gz', 'r:gz') as tar:
    tar.extractall('destination')
```

### 10.4 Logging
```python
import logging

# Basic configuration
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log',
    filemode='w'
)

# Logging levels
logging.debug('Debug message')
logging.info('Info message')
logging.warning('Warning message')
logging.error('Error message')
logging.critical('Critical message')

# Custom logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Handler
handler = logging.FileHandler('app.log')
handler.setLevel(logging.DEBUG)

# Formatter
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger.addHandler(handler)
```

### 10.5 Command Line Arguments
```python
import sys
import argparse

# Sys.argv (simple)
sys.argv                     # List of command line arguments
# python script.py arg1 arg2
# sys.argv = ['script.py', 'arg1', 'arg2']

# Argparse (advanced)
parser = argparse.ArgumentParser(description='Program description')

# Positional arguments
parser.add_argument('input', help='Input file')

# Optional arguments
parser.add_argument('-o', '--output', help='Output file')
parser.add_argument('-v', '--verbose', action='store_true')
parser.add_argument('-n', '--number', type=int, default=1)
parser.add_argument('--choice', choices=['a', 'b', 'c'])

# Parse arguments
args = parser.parse_args()
print(args.input)
print(args.output)
print(args.verbose)
```

### 10.6 Threading & Multiprocessing
```python
import threading
import multiprocessing
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# Threading (for I/O-bound tasks)
def worker():
    print("Working")

thread = threading.Thread(target=worker)
thread.start()
thread.join()                # Wait for completion

# Thread with arguments
thread = threading.Thread(target=worker, args=(arg1, arg2))

# Thread lock
lock = threading.Lock()
with lock:
    # Critical section
    pass

# Thread pool
with ThreadPoolExecutor(max_workers=5) as executor:
    future = executor.submit(func, arg)
    result = future.result()
    
    # Map function over data
    results = executor.map(func, data)

# Multiprocessing (for CPU-bound tasks)
def worker(x):
    return x * x

process = multiprocessing.Process(target=worker, args=(5,))
process.start()
process.join()

# Process pool
with ProcessPoolExecutor(max_workers=4) as executor:
    results = executor.map(worker, range(10))

# Shared memory
from multiprocessing import Value, Array
shared_value = Value('i', 0)     # Shared integer
shared_array = Array('i', [1, 2, 3])  # Shared array

# Queue for inter-process communication
from multiprocessing import Queue
queue = Queue()
queue.put(item)
item = queue.get()
```

### 10.7 Testing
```python
import unittest
import doctest

# Unittest
class TestMath(unittest.TestCase):
    def setUp(self):
        # Run before each test
        self.x = 5
    
    def tearDown(self):
        # Run after each test
        pass
    
    def test_addition(self):
        self.assertEqual(2 + 2, 4)
        self.assertTrue(5 > 3)
        self.assertFalse(5 < 3)
        self.assertIn(1, [1, 2, 3])
        self.assertIsNone(None)
        self.assertRaises(ValueError, func, arg)
    
    @unittest.skip("Skipping this test")
    def test_skip(self):
        pass

# Run tests
if __name__ == '__main__':
    unittest.main()

# Doctest (tests in docstrings)
def add(a, b):
    """
    Add two numbers.
    
    >>> add(2, 3)
    5
    >>> add(-1, 1)
    0
    """
    return a + b

doctest.testmod()            # Run all doctests

# Assert statements
assert condition, "Error message"
assert x > 0, "x must be positive"
```

### 10.8 HTTP Requests (urllib)
```python
import urllib.request
import urllib.parse

# GET request
response = urllib.request.urlopen('https://api.example.com')
data = response.read()
decoded = data.decode('utf-8')

# POST request
data = urllib.parse.urlencode({'key': 'value'}).encode()
req = urllib.request.Request('https://api.example.com', data=data)
response = urllib.request.urlopen(req)

# Headers
req = urllib.request.Request('https://api.example.com')
req.add_header('User-Agent', 'Python')

# URL encoding
encoded = urllib.parse.quote('Hello World')  # 'Hello%20World'
decoded = urllib.parse.unquote('Hello%20World')  # 'Hello World'
```

### 10.9 Email
```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Create message
msg = MIMEMultipart()
msg['From'] = 'sender@example.com'
msg['To'] = 'recipient@example.com'
msg['Subject'] = 'Test Email'

body = 'Email content'
msg.attach(MIMEText(body, 'plain'))

# Send email
server = smtplib.SMTP('smtp.gmail.com', 587)
server.starttls()
server.login('sender@example.com', 'password')
server.send_message(msg)
server.quit()
```

### 10.10 Database (SQLite)
```python
import sqlite3

# Connect to database
conn = sqlite3.connect('database.db')
cursor = conn.cursor()

# Create table
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY,
        name TEXT,
        age INTEGER
    )
''')

# Insert data
cursor.execute("INSERT INTO users VALUES (1, 'Alice', 30)")
cursor.executemany("INSERT INTO users VALUES (?, ?, ?)", [
    (2, 'Bob', 25),
    (3, 'Charlie', 35)
])

# Query data
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()        # Get all rows
row = cursor.fetchone()         # Get one row

# Query with parameters (prevent SQL injection)
cursor.execute("SELECT * FROM users WHERE age > ?", (25,))

# Commit and close
conn.commit()
conn.close()

# Context manager
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    rows = cursor.fetchall()
```

## 11. BEST PRACTICES & TIPS

### 11.1 PEP 8 Style Guide
```python
# Naming conventions
variable_name                # Snake case for variables/functions
CONSTANT_NAME                # Upper case for constants
ClassName                    # Pascal case for classes
_private_var                 # Single underscore for internal use
__private_var                # Double underscore for name mangling
_                            # Throwaway variable

# Indentation: 4 spaces (not tabs)

# Maximum line length: 79 characters

# Imports at top, grouped in order:
# 1. Standard library
# 2. Third-party
# 3. Local application
import os
import sys

import numpy as np

from mypackage import mymodule

# Two blank lines between top-level definitions
# One blank line between methods

# Whitespace
x = 1                        # Spaces around operators
func(x, y)                   # No space before parentheses
lst[0]                       # No space before brackets
{'key': 'value'}             # Space after colon in dicts

# Comparisons
if x is None:                # Use 'is' for None
if x:                        # Implicit bool check
if len(lst) == 0:            # Explicit length check
if not lst:                  # Pythonic empty check
```

### 11.2 Pythonic Idioms
```python
# Swap variables
a, b = b, a

# Multiple assignment
x, y, z = 1, 2, 3

# Conditional expression
x = a if condition else b

# String concatenation (use join for many strings)
' '.join(['Hello', 'World'])

# Check multiple conditions
if x in (1, 2, 3):           # Instead of x == 1 or x == 2 or x == 3

# Default dictionary values
d.get('key', default_value)

# Enumerate instead of range(len())
for i, item in enumerate(lst):
    pass

# Zip for parallel iteration
for a, b in zip(list1, list2):
    pass

# Dictionary merging (Python 3.9+)
d = {**dict1, **dict2}       # Or dict1 | dict2

# Unpacking
first, *middle, last = [1, 2, 3, 4, 5]

# Context managers for cleanup
with open('file.txt') as f:
    pass

# List comprehension over map/filter
[x**2 for x in range(10) if x % 2 == 0]

# Set for membership testing
if x in {1, 2, 3}:           # Faster than list

# String formatting (use f-strings)
f"{name} is {age} years old"

# Chained comparisons
if 0 < x < 10:               # Instead of 0 < x and x < 10

# Use any/all for multiple conditions
if any(condition for item in items):
    pass
if all(condition for item in items):
    pass
```

### 11.3 Performance Tips
```python
# Use list comprehension instead of loops
[x**2 for x in range(1000)]  # Faster than for loop with append

# Use generators for large datasets
(x**2 for x in range(1000000))  # Memory efficient

# Use set for membership testing
s = set(lst)                 # O(1) lookup vs O(n) for list

# Use dict for counting
from collections import Counter
Counter(lst)                 # Better than manual dict

# Use local variables (faster than global)
def func():
    local_var = global_var   # Access global once
    # Use local_var

# Use __slots__ for many instances
class Point:
    __slots__ = ['x', 'y']

# String concatenation
''.join(strings)             # Faster than += in loop

# Use built-in functions (implemented in C)
sum(lst)                     # Faster than manual loop
max(lst)
min(lst)

# Profile code
import cProfile
cProfile.run('function()')

# Timing code
import timeit
timeit.timeit('func()', number=1000)
```

### 11.4 Common Patterns
```python
# Singleton pattern
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Factory pattern
class AnimalFactory:
    def create_animal(self, animal_type):
        if animal_type == 'dog':
            return Dog()
        elif animal_type == 'cat':
            return Cat()

# Observer pattern
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def notify(self):
        for observer in self._observers:
            observer.update()

# Strategy pattern
class Context:
    def __init__(self, strategy):
        self._strategy = strategy
    
    def execute(self):
        self._strategy.do_algorithm()

# Decorator pattern (using functions)
def decorator(func):
    def wrapper(*args, **kwargs):
        # Before
        result = func(*args, **kwargs)
        # After
        return result
    return wrapper
```

### 11.5 Debugging
```python
# Print debugging
print(f"Debug: {variable}")

# Using pdb (Python debugger)
import pdb
pdb.set_trace()              # Set breakpoint

# Debugger commands:
# n (next) - execute next line
# s (step) - step into function
# c (continue) - continue execution
# p variable - print variable
# l - list source code
# q - quit debugger

# Assertions for debugging
assert condition, "Error message"

# Logging (better than print)
import logging
logging.debug("Debug info")

# Traceback
import traceback
try:
    risky_code()
except Exception:
    traceback.print_exc()

# Inspect module
import inspect
inspect.signature(func)      # Function signature
inspect.getsource(func)      # Source code
inspect.getmembers(obj)      # Object members
```

### 11.6 Virtual Environments
```bash
# Create virtual environment
python -m venv myenv

# Activate (Windows)
myenv\Scripts\activate

# Activate (Unix/Mac)
source myenv/bin/activate

# Deactivate
deactivate

# Install packages
pip install package_name

# Requirements file
pip freeze > requirements.txt
pip install -r requirements.txt

# List installed packages
pip list

# Uninstall package
pip uninstall package_name
```

### 11.7 Common Pitfalls
```python
# Mutable default arguments (DON'T DO THIS)
def bad_func(lst=[]):        # BUG: list is shared across calls
    lst.append(1)
    return lst

def good_func(lst=None):     # Correct way
    if lst is None:
        lst = []
    lst.append(1)
    return lst

# Late binding closures
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]         # [2, 2, 2] not [0, 1, 2]

funcs = [lambda i=i: i for i in range(3)]  # Fix
[f() for f in funcs]         # [0, 1, 2]

# Modifying list while iterating
for item in lst:
    if condition:
        lst.remove(item)     # BUG: skips elements

for item in lst[:]:          # Iterate over copy
    if condition:
        lst.remove(item)

# Or use list comprehension
lst = [item for item in lst if not condition]

# is vs ==
a = [1, 2, 3]
b = [1, 2, 3]
a == b                       # True (same value)
a is b                       # False (different objects)

# Integer caching
a = 256
b = 256
a is b                       # True (integers -5 to 256 are cached)

a = 257
b = 257
a is b                       # False (above cache range)
```

## 12. QUICK REFERENCE CHEAT SHEET

### Data Types
- `int`, `float`, `str`, `bool`, `None`
- `list`, `tuple`, `set`, `frozenset`, `dict`

### Common Methods
- **Strings**: `upper()`, `lower()`, `strip()`, `split()`, `join()`, `replace()`, `find()`, `format()`
- **Lists**: `append()`, `extend()`, `insert()`, `remove()`, `pop()`, `sort()`, `reverse()`, `index()`, `count()`
- **Dicts**: `get()`, `keys()`, `values()`, `items()`, `update()`, `pop()`, `setdefault()`

### Built-in Functions
```python
len(), max(), min(), sum(), abs(), round(), sorted(), reversed()
range(), enumerate(), zip(), map(), filter()
any(), all(), type(), isinstance(), hasattr(), getattr(), setattr()
int(), float(), str(), list(), dict(), set(), tuple()
open(), print(), input(), help(), dir()
```

### Keywords
```python
and, or, not, is, in
if, elif, else
for, while, break, continue
def, return, yield, lambda
class, self, super
import, from, as
try, except, finally, raise
with, as
True, False, None
pass, assert, del
global, nonlocal
async, await
```

This comprehensive reference covers the essential Python topics for quick reference and memorization. Keep it handy for your coding journey!