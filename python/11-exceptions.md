# Python — Exceptions

## Catching

```python
try:
    input: str = input('Number of participant to split the bill: ')
    part: float = price / int(input)
except ValueError:
    print('Invalid number entered')
except ZeroDivisionError as e:
    print('Dividing by zero not allowed')
except Exception as e:
    print(f'Unexpected error: {type(e)} - {e}')
else: # it is a success block, but avoid it as it is not an obvious block
    print('Success! No errors encountered')
finally: # always executed, can be handy with closing connections
    print('Program completed')
```

## Raising

```python
def validate(age: int) -> None:
    if age < 0:
        raise ValueError(f'Value {age} is not valid age')
```

## Custom exceptions

To define custom exceptions, inherit from `Exception`. To add support for extra argument(s) to a custom exception, define an `__init__()` method with a variable number of arguments.

```python
class MyProjectError(Exception):
    """A base class for MyProject exceptions."""
    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args)
        self.custom_kwarg = kwargs.get('custom_kwarg')

    # return hints how to reconstruct (unpickle) the object in case
    # it cannot be pickled automatically. It may contain an object reference
    # and parameters with which it will be called to create an initial
    # version of the object, object's state, etc.
    def __reduce__(self) -> None:
        return MyProjectError, ()
```

Reference: https://stackoverflow.com/a/60465422/5004569

## Built-in exceptions list

List of all pre-defined built-in exceptions: https://docs.python.org/3.12/library/exceptions.html, below is the list of the most useful:

```text
BaseException
 ├── GeneratorExit
 ├── KeyboardInterrupt: raised when the user hits the interrupt key (normally Ctrl-C or Delete)
 └── Exception
      ├── ArithmeticError
      │    ├── FloatingPointError
      │    ├── OverflowError
      │    └── ZeroDivisionError
      ├── AssertionError
      ├── AttributeError: raised when an attribute reference or assignment fails
      ├── BufferError: raised when a buffer-related operation cannot be performed
      ├── EOFError
      ├── LookupError: raised when a key or index used on a mapping or sequence is invalid
      │    ├── IndexError
      │    └── KeyError
      ├── MemoryError: raised when an operation runs out of memory but the situation may still be rescued
      ├── OSError
      │    ├── BlockingIOError
      │    ├── ConnectionError
      │    ├── FileExistsError
      │    ├── FileNotFoundError
      │    ├── IsADirectoryError
      │    ├── NotADirectoryError
      │    ├── PermissionError
      │    └── TimeoutError
      ├── RuntimeError
      ├── StopIteration: raised by built-in function next() and an iterator's __next__() method
      ├── SystemError
      ├── TypeError: raised when an operation or function is applied to an object of inappropriate type
      ├── ValueError
      └── Warning
```

<!-- nav -->
---

← [Python — OOP](10-oop.md) · [Index](../README.md) · [Python — Dependencies](12-dependencies.md) →
<!-- nav -->
