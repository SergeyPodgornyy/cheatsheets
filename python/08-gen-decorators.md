# Python: Generators, Decorators, Memoization

## Generators

Generators are incredibly memory-efficient compared to lists, tuples and most other iterables. Any function that `yield`s a value turns into a generator. The function `next` retrieves the next value from the the iterator. Generators are exhaustive, which means once we retrieve a value, it disappears from that generator.

```python
from typing import Generator

def values() -> Generator:
    for i in range(1, 5): # 1, 2, 3, 4, 5
        yield i

numbers: Generator = values()
print(next(number))   # 1
print(next(number))   # 2
print(next(number))   # 3
print(list(number))   # [4, 5]
print(list(number))   # []
print(next(number))   # StopIteration error
```

## Decorators

All functionality that belongs to the decorator, should be inside the `wrapper`. To use the decorator, we just need to annotate the function with the decorator function name.

```python
import time
from typing import Callable
from functools import wraps

def with_execution_time(func: Callable) -> Callable:
    """Print time how long it executes the function."""

    # to get the actual name and docstring from the function we use
    # we need to use wraps decorator on our wrapper
    @wraps(func)
    def wrapper(*args, **kwargs) -> None:
        start_time: float = time.perf_counter()
        func(*args, **kwargs)
        end_time: float = time.perf_counter()

        print(f'Execution time: {end_time - start_time:.3f} sec.')

    return wrapper

@with_execution_time # adding a decorator to the function
def calculate() -> None:
    pass

calculate.__name__ # wraps helps to return the actual name of wrapped function
calculate.__doc__ # wraps helps to return actual docstring of wrapped function
```

Decorators can also accept arguments, but we need to perform a little bit of inception, which means we will go one layer deeper. So instead of just having one inner function, we will have two inner functions.

```python
from typing import Callable, Any
from functools import wraps

def repeat(count: int) -> Callable:
    """Repeat the function call x amount of times.."""

    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            value: Any = None
            for _ in range(count):
                value = func(*args, **kwargs)

            return value

        return wrapper

    return decorator

@repeat(number=3) # kwargs can be args, as usual
def ping(url: str) -> str:
    return 'pong'
```

## Memoization

We can store the function output with the same input arguments in the cache:

```python
from functools import cache

@cache
def ping(url: str) -> str:
    return f'response from {url}'

ping('google.com')
ping.cache_info() # CacheInfo(hits=0, misses=1, maxsize=None, currsize=1)
ping.cache_clear() # clears an entire cache for the function
```

Memoization and a cache can be extremely useful in recursions:

```python
@cache
def fibonacci(num: int) -> int:
    if num < 2:
        return num

    return fibonacci(num - 1) + fibonacci(num - 2)

@cache
def factorial(num: int) -> int:
    return num * factorial(num - 1) if num else 1
```

<!-- nav -->
---

← [Python: File Handling](07-file-handling.md) · [Index](README.md) · [Python: AsyncIO](09-async.md) →
<!-- nav -->
