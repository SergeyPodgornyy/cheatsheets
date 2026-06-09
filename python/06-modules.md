# Python — Modules / Packages / Libraries

**Modules** are separate **files**.

A **package** is a collection of modules in the **directory**. There should be `__init__.py` to initialize the package (but it can be empty). It will be run as soon as we import the package. It is the only way to tell Python that it is a package and not a folder.

**Library** refers to the collection of modules and packages that can be used to perform specific tasks.

```python
from package import module
import package.module
import module          # if in the same folder
import package         # ❌ it won't import the functionality we are looking for

import helper
helper.count()

import helper as hp
hp.count()

from helper import count, help
count()
help()

from helper import * # be aware of overwrites from other modules
count()
```

## Main module execution

Whenever we import something, it is going to run all the code from the imported module. So for safety reasons, it is advisable to add all executed code into the condition, when it was called directly on a purpose:

```python
if __name__ == '__main__':
    ...  # run the block when the script was used directly (not imported).
         # important for every module that can be used by another script
```

`__name__` is used to determine which file is currently running.

## Random

```python
from random import choice, choices, sample

names: list[str] = ['Bob', 'George', 'Anna', 'Sophia']

winner: str = choice(names)                         # Sophia
winners: list[str] = choices(names, k=2)            # ['Bob', 'Bob']
unique_winners: list[str] = sample(names, k=2)      # ['Bob', 'Sophia']
```

## Partial

```python
from functools import partial

def specification(country: str, area: str, name: str) -> None:
    pass

# instead of this
specification('us', 'game', 'Oblivion')
specification('us', 'game', 'Skyrim')

# use this
us_game_spec: partial = partial(specification, 'us', 'game')
us_game_spec('Oblivion')
us_game_spec('Skyrim')
```

## Performance benchmarks

Performing benchmarks in Python is probably one of the hardest things to get right. It is easy to get biased, so it is good to perform plenty of tests, swap the order of functions around, change the test size, and use the `repeat` function.

The `warmup` ensures that the interpreter is performing at peak performance. It is recommended to warm up the code before actually performing tests.

```python
from timeit import timeit

stmt: str = 'list(range(1000))'
warmup: float = timeit(stmt=stmt, number=100_000)
time: float = timeit(stmt=stmt, number=100_000)
print(f'Execution time: {time:.3f} sec.')
```

The `repeat` function is probably the safest way to get back consistent and unbiased results when you are using the `timeit` module. The `number` argument defines how many time the snippet of code will run. Also, when using `repeat` function, we do not need to insert a warm up, because by repeating this function many times, we are bringing the interpreter up to speed.

```python
from timeit import repeat

stmt: str = 'set(range(1000))'
time: float = min(repeat(stmt=stmt, repeat=5, number=100_000))
print(f'Execution time: {time:.3f} sec.')
```

When the code snippet requires a setup, we need to provide it as an extra `setup` argument, and it will run only once during the entirety of our test.

```python
power: float = timeit(stmt='a**b', setup='a, b = 10, 3')
math_power: float = timeit(stmt='pow(10, 3)', setup='from math import pow')
```

If we want to use a function from the global scope, we can pass globals as an argument:

```python
def factorial(num: int) -> int:
    return num * factorial(num - 1) if num else 1

speed: float = timeit('factorial(5)', globals=globals())
```

<!-- nav -->
---

← [Python — Functions](05-functions.md) · [Index](../README.md) · [Python — File Handling](07-file-handling.md) →
<!-- nav -->
