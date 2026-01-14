# Python Basics

> **What it is:** Python is a high-level, interpreted programming language known for its readability and versatility. It's widely used for automation, scripting, web development, data analysis, and system administration.

## Data Types

```python
# Variables (no type declaration needed)
name = "John"           # String
age = 25                # Integer
price = 19.99           # Float
is_active = True        # Boolean
nothing = None          # None type

# Type conversion
str(123)                # "123"
int("123")              # 123
float("19.99")          # 19.99
```

## Strings

```python
text = "Hello, World!"
print(len(text))              # 13
print(text.upper())           # HELLO, WORLD!
print(text.lower())           # hello, world!
print(text.replace("World", "Python"))
print(text.split(", "))       # ['Hello', 'World!']

# Slicing
text = "Python"
print(text[0])         # P
print(text[-1])        # n
print(text[0:3])       # Pyt
print(text[::-1])      # nohtyP (reversed)

# f-strings
name = "Alice"
print(f"{name} is {age} years old")
```

## Lists

```python
fruits = ["apple", "banana", "cherry"]
fruits.append("orange")        # Add to end
fruits.insert(1, "mango")      # Insert at index
fruits.remove("banana")        # Remove by value
popped = fruits.pop()          # Remove and return last

# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
```

## Dictionaries

```python
user = {"name": "John", "age": 30}
print(user["name"])           # John
print(user.get("name"))       # John (safer)
user["phone"] = "123-456"     # Add key

for key, value in user.items():
    print(f"{key}: {value}")
```

## Loops

```python
# For loop
for fruit in fruits:
    print(fruit)

for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

for i in range(5):
    print(i)

# While loop
count = 0
while count < 5:
    print(count)
    count += 1
```

## Functions

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# *args and **kwargs
def sum_all(*args):
    return sum(args)

def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# Lambda
square = lambda x: x ** 2
```

## File Operations

```python
# Read file
with open("file.txt", "r") as f:
    content = f.read()

# Write file
with open("output.txt", "w") as f:
    f.write("Hello, World!\n")

# Read line by line
with open("file.txt", "r") as f:
    for line in f:
        print(line.strip())
```

## Practical Examples

### Count Words

```python
def count_words(filename):
    with open(filename, "r") as f:
        return len(f.read().split())
```

### Word Frequency

```python
from collections import Counter

def word_frequency(filename):
    with open(filename, "r") as f:
        words = f.read().lower().split()
        return Counter(words)

freq = word_frequency("document.txt")
for word, count in freq.most_common(10):
    print(f"{word}: {count}")
```

### Extract IPs from Log

```python
import re

def extract_ips(filename):
    pattern = r'\b(?:\d{1,3}\.){3}\d{1,3}\b'
    with open(filename, "r") as f:
        return set(re.findall(pattern, f.read()))
```

### List Files with Sizes

```python
import os

def list_files(directory):
    for root, dirs, files in os.walk(directory):
        for filename in files:
            filepath = os.path.join(root, filename)
            size_mb = os.path.getsize(filepath) / (1024 * 1024)
            print(f"{size_mb:.2f} MB  {filepath}")
```

## Quick Reference

```python
# File operations
open("file", "r")   # Read
open("file", "w")   # Write
open("file", "a")   # Append

# String methods
s.upper(), s.lower(), s.strip()
s.split(), s.replace(), s.find()

# List methods
l.append(), l.pop(), l.sort()

# Common imports
import os, sys, re, json, csv
from collections import Counter
```

---

*Part of the [Programming Documentation](README.md)*

