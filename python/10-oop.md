# Python — OOP

## Classes and self

`self` refers to the current instance of the class. It is a convention to call it `self`, but it can be any name.

```python
class Car:
    SPEED_LIMIT: float = 130 # class attribute to share across all instances

    # initializer / constructor
    def __init__(self, brand: str) -> None:
        self.brand = brand

    @classmethod # changing class attribute for all instances
    def city_limit(cls, limit: float) -> None:
        cls.SPEED_LIMIT = limit

    @classmethod # represents a factory method
    def from_brand(cls, brand: str) -> Self:
        return cls(brand.upper())

    def drive(self, *, speed: float) -> None:
        if speed > self.SPEED_LIMIT:
            print(f'Driving {self.brand} at {self.SPEED_LIMIT} km/h limit.')
        else:
            print(f'Driving {self.brand} at {speed} km/h.')

    def describe(self) -> None:
        print(f'Represents a car of {self.brand}')
        print(f'Content: {self.__dict__}')

bmw: Car = Car('BMW')
vw: Car = Car.from_brand('vw')
print(bmw.brand)      # BMW
print(vw.brand)       # VW
bmw.describe()

Car.SPEED_LIMIT = 120 # Changing class value, all instances will be affected
bmw.drive(speed=150)  # Driving BMW at 120 km/h limit.

bmw.SPEED_LIMIT = 170 # Changing only instance value
bmw.drive(speed=150)  # Driving BMW at 170 km/h.
vw.drive(speed=150)   # Driving VW at 120 km/h.

bmw.city_limit(50)    # Changing class value, all instances will be affected
bmw.drive(speed=150)  # Driving BMW at 50 km/h limit.
vw.drive(speed=150)   # Driving VW at 50 km/h limit.
```

## Inheritance and annotations

```python
class Cabrio(Car):
    def __init__(self, brand: str) -> None:
        # super() refers to the parent class
        super().__init__(brand)

    @override # from typing import override
    def describe(self) -> None:
        print(f'This is a cabrio represented by {self.brand}')

    @staticmethod # method can be called with instance or on class directly
    def info() -> None:
        print('It represents cabrio car type.')
```

## Abstraction

```python
from abc import ABC, abstractmethod

class Appliance(ABC):
    def __init__(self, version: int) -> None:
        self.version = version
        self.is_running = False

    @abstractmethod
    def turn_on(self) -> None:
        ...

    @abstractmethod
    def turn_off(self) -> None:
        ...

# instance of Appliance can't be created directly, because it has abstract methods
```

## Protocol

Protocol is a parent class that requires all inherited instances to have their own implementation of methods inside the protocoled class:

```python
from typing import Protocol

class Printable(Protocol):
    pages: int

    def print(self) -> None:
        ...

class Book(Printable):
    pages: int

    def __init__(self, title: str) -> None:
        self.title = title

    def print(self) -> None:
        print(f'Printing book "{self.title}" ...')

def printer(printable: Printable) -> None:
    printable.print()
```

## Singleton

To use a singleton pattern, we would need to use the `__new__` method, which is always called before the initializer (`__init__`). Singleton `__new__` method always has a return and makes sure that we only have one instance of class running at any given moment of our program. Also whatever the signature for the variables and keyword arguments are in the `__new__`, you are going to have to include them in the initializer.

```python
class Connection:
    __instance = None

    def __new__(cls, *args, **kwargs) -> Self:
        if cls.__instance is None:
            print(f'Connecting...')
            cls.__instance = super().__new__(cls)

        return cls.__instance

    def __init__(self) -> None:
        print(f'Connected!')
```

## Name mangling (private access modifier)

All attributes, properties or methods with leading **2 underscores** within the class pretended to be **private** and will be replaced by Python interpreter to `_<class>__<method>`. Everything with leading 1 underscore pretended to be protected, but not verified by interpreter.

```python
class Car:
    __TYPE: str = 'fuel'

    def __init__(self, brand: str) -> None:
        self.__brand = brand

    def __describe(self) -> None:
        print(f'{self.__TYPE} car represented by {self.__brand}')

bmw: Car = Car('BMW')

# everything within class is private, but can be access with name mangling
print(bmw._Car__TYPE)
print(bmw._Car__brand)
bmw._Car__describe()
```

While it makes attribute names harder to access, it is still possible to access them if one knows the mangled name. For instance, `_ClassName__attr` can be used to access the attribute directly. Overuse of name mangling can make the code harder to read and understand. If you use dynamic class names (e.g., using `type()` to create classes), name mangling can become less predictable and harder to manage.

## Dunder methods

Also known as magic methods or special methods, are predefined methods in Python that have <u>double underscores</u> (or "**dunders**") at the beginning and end of their names. These methods provide a way to define specific behaviors for built-in operations or functionalities in Python classes.

```python
def __len__(self) -> int: # to call len() function on instance
def __add__(self, other: Self) -> Self: # when add 2 instances with '+'
def __repr__(self) -> str: # string representation of an instance with repr()
def __str__(self) -> str: # string representation of an instance
def __eq__(self, other: Self) -> bool: # compares 2 objects with '=='
    return self.__dict__ == self.__dict__
```

The `dir()` function on the object returns all properties and methods (including magic methods inherited by a class) of the specified object, without the values. (e.g. `__abs__`, `__add__`, `__bool__`, `__class__`, `__eq__`, `__hash__`, `__init__`, `__new__`, `__repr__`, `__str__`, …)

More: https://www.geeksforgeeks.org/dunder-magic-methods-python/

## Context manager

Context managers allow you to allocate and release resources precisely when you want to. The most widely used example of context managers is the `with` statement. Suppose you have two related operations which you'd like to execute as a pair, with a block of code in between. Context managers allow you to do specifically that.

At the very least a context manager has an `__enter__` and `__exit__` method defined, and just by defining them, we can use our new class in a `with` statement.

```python
class File(object):
    def __init__(self, path, mode):
        self.file = open(path, mode)

    def __enter__(self):
        return self.file

    def __exit__(self, type, value, traceback):
        if type is not None:
            print('Error happened')
        self.file.close()

with File('demo.txt', 'r') as file:
    pass
```

## Data classes

The sole purpose of `dataclass` is to hold and represent data. To create a data class, first, we need to annotate it as `@dataclass` and we need to create a class as normal.

```python
from dataclasses import dataclass

@dataclass
class Coin:
    name: str
    value: float
    id: str # with dataclasses we can use built-in names, while with normal not

bitcoin: Coin = Coin('Bitcoin', 10_000, 'BTC')
```

When we create mutable defaults, the `field` becomes very important. We cannot simply pass a list or a set or a dictionary because those are mutable, we need to use `default_factory`. Also, `field` allows you to set default in any order of properties, while normal equal sign allows it only at the end. You can also add methods in data classes as usual.

```python
from dataclasses import dataclass, field, InitVar

@dataclass
class Fruit:
    name: str
    grams: float = field(default=0)
    price: float
    locations: list[str] = field(default_factory=list)
    eatable: bool = True
    is_rare: InitVar[bool | None] = None # init-only variable
    total_price: float = field(init=False)

    # by providing InitVar as a type annotation for the field
    # Python consider it as a part of initializer, not a part of a class
    def __post_init(self, is_rare: bool | None) -> None:
        if is_rare:
            self.price *= 2
        # computed property would be better for this example
        # as post initializer runs only once
        self.total_price = (self.grams / 1000) * self.price

    def describe(self) -> None:
        print(f'{self.grams}g of {self.name} costs ${self.total_price}')
```

We can create a method that will act as a virtual property, if we will use `@property` annotation.

```python
from dataclasses import dataclass, property

@dataclass
class Fruit:
    name: str
    grams: float
    price: float

    @property
    def total_price(self) -> float:
        return (self.grams / 1000) * self.price
```

## Getters and setters

`@property` annotation can be used as a getter to property. But for the setter, we need to use `@name.setter` annotation. In addition, we also have `@name.getter` and `@name.deleter` annotations. But it is important that the `name` should be the same as the property accessor method.

```python
class Fruit:
    def __init__(self, name: str, calories: float) -> None:
        self.__name = name
        self.__calories = calories

    @property # can be also @name.getter
    def name(self) -> str:
        return self.__name

    @name.setter
    def name(self, value: str) -> None:
        self.__name = value

    @kcal.getter
    def kcal(self) -> float:
        return self.__calories

    @kcal.setter
    def kcal(self, value: float) -> None:
        self.__calories = value
```

## Enums

Enums are a safe way of defining a fixed set of constant values that helps drastically with reducing errors that we get from the input. To use enums in Python, we need to import them from the corresponding package:

```python
from enums import Enum

class Gender(Enum):
    MALE: str = 'male'
    FEMALE: str = 'female'

gender: Gender = Gender.MALE

print(gender)          # Gender.MALE
print(gender.value)    # male
print(gender.name)     # MALE
print(Gender('male'))  # Gender.MALE
other: Gender = Gender('other') # ValueError: 'other' is not valid Gender

if gender == Gender.MALE:
    print('Welcome to the boys club')
```

## Memoization of class methods

We cannot add `@cache` decorator in classes, as we need to use a different decorator:

```python
from functools import cached_property

class DataSet:
    def __init__(self, data: list[float]) -> None:
        self._data = data

    @cached_property
    def exists(self, value: float) -> bool:
        return value in self._data

ds: DataSet = DataSet([1.5, 2.25, 13.55])

del ds.exists # AttributeError: 'DataSet' object has no attribute 'exists'
ds.exists(5.56)
del ds.exists # clears the cache
```

## Monkey patching

Python allows to add/replace methods in the class during runtime. This can be useful when attaching our custom implementation to existing remote libraries:

```python
import pandas as pd

def get_foo_cols(self):
    """Get a list of column names containing the string 'foo'."""

    return [x for x in self.columns if 'foo' in x]

pd.DataFrame.get_foo_cols = get_foo_cols # monkey-patch the DataFrame class

df = pd.DataFrame([list(range(4))], columns=["A", "foo", "foozball", "bar"])
df.get_foo_cols()
del pd.DataFrame.get_foo_cols # remove the new method
```

If you're using name-mangling (prefixing attributes with a double-underscore, which alters the name) you'll have to name-mangle manually if you do this.

Reference: https://stackoverflow.com/a/27466499/5004569

<!-- nav -->
---

← [Python — AsyncIO](09-async.md) · [Index](../README.md) · [Python — Exceptions](11-exceptions.md) →
<!-- nav -->
