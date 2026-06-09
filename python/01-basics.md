# Python — Basics

## Language

Python is an **interpreted** language: it executes line by line and doesn't need to be compiled. Python is a **dynamically typed** language — it allows us to freely use and change variable types during runtime; types of the variables are going to be checked during the runtime, not at the compile time. We are also not required to explicitly give the data type. All type errors will be detected only during the execution later.

Python provides us with **high-level** abstractions between the programmer and the computer. It makes it easier to program by making the code more human-readable.

## Execution

To run Python code, we need to run it from the terminal as:

```console
$ python main.py # python3 for Linux
```

We can also run Python in optimized mode, which removes all assertions:

```console
$ python -O main.py
```

Python also can be run in aggressive optimized mode, which also strips docstring:

```console
$ python -OO main.py
```

It can be checked in Python script if it is running with normal or optimized mode from the Python file:

```python
print(__debug__) # True or False, where __debug__ == True is default and normal mode
```

## PEP

**Python Enhancement Proposals**, known as PEPs, give developers a guide on how Python should be written. PEP-8 represents the Style Guide for Python Code.

## Variables

**Snake case** is a naming convention for variables in Python. Also, Python is a case-sensitive language and `data` is not the same as `DATA`.

```python
snake_case = 'Hello World'
```

Everything that <u>returns</u> a value becomes an **expression**. On the other side, the **statement** <u>does not return</u> anything.

## Data types

```python
# Numeric types
number = -100
also_num = (10)
percent = 1.50
imaginary = 9j
PI = 3.14 # constants supposed to be uppercase, but not validated as const
big_number = 1_000_000_000
exp_big_number = 1e9
exp_big_float = 1e-10

# Boolean type
is_connected = True
has_money = False

print(True == 1)        # True
print(False == 0)       # True
print(True + True)      # 2

empty = None

# String type
text = 'Hello World'
name = "Sergey"
escaped = "Bob says \"Hello\" to everyone"
new_line = "Bob\t21\nCharlie\t25\n"
heredoc = """This is a multi-line string
It can also be wrapped with single quotes
"""

# Sequence types
number = [1, 2, 3, 4]      # list
coordinates = (2.5, 1.0)   # tuple
coordinates = 2.5, 1.0     # tuple (comma creates a tuple, not parenthesis)
# list is mutable, while tuple is immutable

# Mapping type
users = {'Mario': 1, 'Luigi': 2} # dict
# dict (dictionary) is a representation of a hash-table

# Set types
raffle = {1, 10, 25, 50}
frozen = frozenset({1, 2, 3})
# set is mutable, to create an immutable set, use frozenset.
```

There are also more specific types such as `bytes`, `bytearray` or `memoryview`.

## Multiple assignments

```python
first, last = 1, 2                # 1 2
first, last = [1, 2]              # 1 2
first, last = 'SP'               # S P
first, *others = [1, 2, 3, 4]    # 1 [2, 3, 4]

# semicolons at the end of the line are optional and not recommended by PEP,
# but Python will allow us to do this, especially like this below
a = 1; b = 2; c = 3;
```

### Swapping variables

```python
a, b = b, a
```

### Deleting

To delete a variable from the memory, or an element from the list, `del` keyword should be used:

```python
del data
del data[1]
del data['mario']
```

## Type annotation (type hints)

Type hints can be defined for better code, but they are ignored by the Python compiler.

```python
number: int = 7
text: str = 'Hello world!'
print(type(text)) # <class 'str'>
```

### Type conversion

```python
txt: str = '100'
num: int = 500

print(txt + str(num)) # 100500
print(int(txt) + num) # 600
```

### Custom type

```python
type number = int | float | None
amount: number = fetch()
```

## Strings

```python
hidden: str = '*' * 10 # **********
```

## Lists — collection

```python
empty: list = []
empty: list = list()
big: list[int] = [0] * num         # efficiently create list of fixed size with 0
people: list[str] = ['Bob', 'James', 'Tom']
people.append('Jeremy')            # ['Bob', 'James', 'Tom', 'Jeremy']
people.remove('Bob')               # ['James', 'Tom', 'Jeremy']
people.pop()                       # pop last element: ['James', 'Tom']
people[0] = 'Charlotte'            # ['Charlotte', 'Tom']
people.insert(1, 'Timothy')        # ['Charlotte', 'Timothy', 'Tom']
people.extend(['Phil', 'Sofia'])   # ['Charlotte', 'Timothy', 'Tom', 'Phil', 'Sofia']
people.clear()                     # []
people += ['Mario', 'Luigi']       # ['Mario', 'Luigi']
people.sort()                      # ['Luigi', 'Mario']
people.reverse()                   # ['Mario', 'Luigi']
```

## Tuples — immutable collection

```python
empty: tuple = tuple()
empty: tuple = ()
one: tuple = (1,)
another: tuple = 1,
# tuples are immutable, but allow duplicates and mixed types
mix: tuple = 1, 'Bob'
print(mix[0])    # 1
print(mix[-1])   # Bob

coordinates: tuple[float, float] = (1.5, 1.5,)
coordinates.count(1.5)  # 2 -> count how many times value appear in tuple
coordinates.index(1.5)  # 0 -> find the first position of value in tuple
coordinates[0] = 2.5    # TypeError: 'tuple' object does not support item assignment
```

## Sets — collection of unique values

```python
# set can't have duplicates, but values are unordered
elements: set = {99, True, 'Bob', True}
print(elements)            # {True, 'Bob', 99}
print(elements[0])         # TypeError: we cannot get index as order is random
elements.add('James')      # {99, True, 'James', 'Bob'}
elements.remove('Bob')     # {99, True, 'James'}
elements.remove('John')    # KeyError
elements.discard('John')   # discard is same as remove, but do not throw an error
# update accepts any collection type: list, tuple, set
elements.update(['Bob', 'Tom']) # {99, True, 'James', 'Bob', 'Tom'}
# pop removes a random element from set, because it is unordered
elements.pop()             # {99, True, 'James', 'Tom'}
elements.clear()           # set()

elements = elements.union({7, False}) # merge 2 sets
elements = elements | {7, False}       # also combines 2 sets

# returns duplicates from 2 sets
elements.intersection_update({12, False, 'James'}) # {False}
new_set = elements.intersection({12, False, 'James'})

# returns unique difference
elements.symmetric_difference_update({12, False, 'James'}) # {12, 'James'}
new_set = elements.symmetric_difference({12, False, 'James'})

empty: set = set()
# unchangeable frozenset object
things: frozenset = frozenset({1, 1, 2, 3, 3})
print(things) # frozenset({1, 2, 3})
```

## Dictionaries — hash-table

```python
empty: dict = dict()
empty: dict = {}
people: dict = {'mario': 1, 'luigi': 2}

print(people['unknown'])              # KeyError: 'unknown'
print(people.get('unknown', 0))       # 0 (None, if 0 not set as second arg)
print(people)                         # {'mario': 1, 'luigi': 2}
print(people.keys())                  # dict_keys(['mario', 'luigi'])
print(people.values())                # dict_values([1, 2])
print(people.items())                 # dict_items([('mario', 1), ('luigi', 2)])
print(people.setdefault('unknown', 0))# 0
print(people)                         # {'mario': 1, 'luigi': 2, 'unknown': 0}
people['unknown'] = 7                 # {'mario': 1, 'luigi': 2, 'unknown': 7}
people.pop('unknown')                 # {'mario': 1, 'luigi': 2}
del people['luigi']                   # {'mario': 1}
people.update({'marco': 3})           # {'mario': 1, 'marco': 3}
people.popitem()                      # {'mario': 1} - pop the last item
people.clear()                        # {}
```

### Merge dictionaries

```python
d1 = {'name': 'Sergey', 'age': 31}
d2 = {'name': 'Sergey', 'city': 'Munich'}
merged = {**d1, **d2}
print(merged) # {'name': 'Sergey', 'age': 31, 'city': 'Munich'}
```

## None

```python
user: dict | None = people.get('unknown')
```

## Truthy and Falsy

```python
# everything below is considered as Falsy
empty: bool = False
empty: int = 0
empty: float = 0.0
empty: str = ''
empty: list = []
empty: tuple = ()
empty: set = set()
empty: dict = {}
empty: range = range(0)
empty: complex = complex(0j)
empty: None = None
```

<!-- nav -->
---

← [Home](../README.md) · [Index](../README.md) · [Python — Operators](02-operators.md) →
<!-- nav -->
