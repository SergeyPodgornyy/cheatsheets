# Python — Functions

## Scope

An inner scope can access variables from the outer scope, but it cannot reassign a value to it. We need to use a `global` keyword to overwrite the variable from the global scope. Everything in the <u>outermost layer</u> is part of the global scope.

An outer scope cannot access variables from the inner scope of any block.

```python
number: int = 10

def nothing() -> None:
    print(number)        # 10 - variable from outer scope
    number: int = 100    # inner variable
    print(number)        # 100 - variable from inner scope reassigned to
                         #     the same name

def reset() -> None:
    global number
    number: int = 0
    print(number)        # 0 - variable from the global scope

nothing()  # does not affect the number variable
reset()    # change number variable
```

To access variables from the outer scope, we need to use `nonlocal` keyword.

```python
def outer() -> None:
    name: str = 'Tom'
    age: int = 20

    def inner() -> None:
        nonlocal name, age
        name: str = 'Jerry'
        age: int = 10

    inner()
    print(name, age) # Jerry, 10
```

## Arguments and parameters

Normal arguments should always be before keyword (named) arguments:

```python
def greet(name: str, lang: str, default: str = 'Hello') -> None:
    ...

greet('Markus', lang='de', default='Hallo')
greet(name='Mario', lang='it', default='Ciao')
greet('Mykola', lang='ua')
```

## Lambda (Anonymous functions)

Lambdas are just nameless functions that you can create and use on a spot. To create a lambda, we need to use the `lambda` keyword, followed by the parameters that we want this lambda to use.

```python
add = lambda a, b: a + b

names: list[str] = ['Sarah', 'Bob', 'Samantha', 'John']
sorted_names: list[str] = sorted(names, key=lambda x: len(x))
```

## Default argument values

```python
# WRONG: mutable default is shared across calls
def append(n, l=[]):
    l.append(n)
    return l

l1 = append(0)  # [0]
l2 = append(1)  # [0, 1] oops

# CORRECT: use None as a sentinel
def append(n, l=None):
    if l is None:
        l = []
    l.append(n)
    return l

l1 = append(0)  # [0]
l2 = append(1)  # [1]
```

## Rest and required arguments (args and kwargs)

```python
def join_text(*strings: str, sep: str) -> str:
    # strings is a tuple
    return sep.join(strings)
```

Everything passed after `*args` must be a keyword argument:

```python
def greet(greeting: str, *people: str, ending: str = '!') -> None:
    for person in people:
        print(f'{greeting}, {person}{ending}')
greet('Hello', 'Sarah', 'Emily', 'John', ending=' 🎉')
```

`*args` must come before `**kwargs` (keyword args), and you can't put anything after `**kwargs`:

```python
def func(*args: int, default: int, **kwargs: int) -> None:
    # kwargs is a dict
    ...
func(1, 2, default=20, a=1, b=2)
```

Example:

```python
def order_pizza(size, *toppings, **details):
    print(f"Ordered a {size} pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")
    print("\nDetails of the order are:")
    for key, value in details.items():
        print(f"- {key}: {value}")

order_pizza("large", "pepperoni", "olives", delivery=True, tip=5)
# Ordered a large pizza with the following toppings:
# - pepperoni
# - olives
#
# Details of the order are:
# - delivery: True
# - tip: 5
```

## Forcing argument signature

Slash `/` forces arguments to be **positional** arguments, so everything **before** slash cannot be a keyword argument:

```python
def search(query: str, /, filters: dict) -> None:
    # query cannot be passed as kwarg; filters can be both arg or kwarg
    pass

search('Query', {})
search('Query', filters={})

def get(internal: str, /) -> None:
    pass

get('something')
```

Asterisk `*` forces everything that comes **after** to be a **keyword** argument:

```python
def full_name(first: str, last: str, *, sep: str) -> str:
    # sep must be kwarg, but first and last can be both arg and kwarg
    return sep.join([first, last])

full_name('Max', 'Mustermann', sep=' ')
full_name(first='Max', last='Mustermann', sep=' ')
```

Slash and asterisk can be used together:

```python
def connect(url: str, /, *, secure: bool) -> None:
    pass
connect("mysql:host=localhost;port=3306", secure=False)

def greet(greeting: str, /, name: str, *, salutation: str) -> str:
    pass
greet("Hello", "John", salutation="Mr.")
greet("Hi", name="John", salutation="Dr.")
```

## Built-in functions

Every single function that does not return something, does implicitly return `None`.

| Function | Description |
| --- | --- |
| `print()` | prints to STDOUT (by default): `sep=' '` is a separator between args, `end='\n'` is the ending of the print, and the `file` is an object to print output. `print("Adam", "John", sep=' and ', end="!\n")` |
| `enumerate()` | enumerates list with indexes. `enumerate(['A', 'B', 'C'], start=1)` → `[(1, 'A'), (2, 'B'), (3, 'C')]` |
| `id()` | returns the memory address of the variable |
| `zip()` | combines lists together: `zip(numbers, letters)` stops early at shortest list. `zip(a, b, strict=True)` raises ValueError on length mismatch. Accepts more than 2 args. |
| `round()` | rounds (up or down) numbers: `round(265.55389999, 2)` → `265.56`; `round(265.55389999, 0)` → `266.0`; `round(265.55389999, -2)` → `300` |
| `range()` | creates an iterable range of numbers: `range(5)` → 0..4; `range(1, 6)` → 1..5; `range(0, 10, 2)` → 0,2,4,6,8; `range(-5, 0)`; `range(0, -5, -1)` |
| `slice()` | creates a slice of sliceable object: `slice(0, 3)`; `slice(None, None, -1)` → `[::-1]` |
| `filter()` | applies filter function with an object (memory-efficient): `filter(lambda n: n % 2 == 0, numbers)` |
| `map()` | maps a function to an iterable (memory-efficient): `map(lambda n: n * 2, numbers)`; stops early at shortest when multiple iterables |
| `sorted()` | sorts elements ascending (default): `sorted([1, 10, 7, 12])`; `sorted(names, reverse=True)`; `sorted(names, key=lambda x: len(x))`; `sorted(chars, key=str.lower)`. Sorts strings by ASCII (A=65, a=97). `.sort()` sorts in-place without return |
| `all()` | checks all conditions together (True if all truthy) |
| `any()` | checks if at least one condition passed |
| `type()` | returns a type of variable |
| `isinstance()` | checks variable is an instance of type: `isinstance(3.14, int \| float)` |
| `callable()` | checks is a variable of callable type |
| `repr()` | represents a string value of the variable |
| `dir()` | returns all properties and methods of the variable (object) |
| `globals()` | returns everything visible in the global scope as a dictionary: variables, classes, packages… |
| `locals()` | returns everything visible in the local scope |
| `eval()` | evaluate the string expression as a code and return the result: `eval('1 + 2 + 10')`. Anything that is not an expression won't work: `eval('x = 10')` → SyntaxError |
| `exec()` | executes the string expression as a code, thus extremely dangerous |
```

<!-- nav -->
---

← [Python — Comprehension and Unpacking](04-comprehension.md) · [Index](../README.md) · [Python — Modules / Packages / Libraries](06-modules.md) →
<!-- nav -->
