# Python — Operators

## Numeric

```python
number1 // number2   # floor (rounds down) to avoid decimal values (keeps only int)
print(5 // 2)        # 2
print(-5 // 2)       # -3

number1 ** number2   # exponential (power)
print(2 ** 3)        # 8

number1 % number2    # modulo (remainder)
print(10 % 3)        # 1
```

## Assignment

```python
a += b, a -= b, a *= b, a /= b, a //= b, a **= b, a %= b, a |= b
```

## Comparison

```python
a == b, a != b, a > b, a < b, a >= b, a <= b
```

## Logical

```python
and, or, not       # logical
is, is not         # identity (same object)
in, not in         # membership

# Python has NO `xor` keyword. For boolean XOR use `^` (bitwise, works on bools) or `!=`:
True ^ False        # True
True != False       # True
```

## Equality (==) vs identity (is)

Identity operator `is` also checks the **memory address** of variables. Sometimes Python makes a trick and allocates the same memory address for different variables for optimization processes.

```python
a: int = 1000
b: int = 1000
a == b     # True, correct check
a is b     # True, but wrong check, as memory addresses might be different

a: int = 1000
b: int = int('1000')
a is b     # False

var is None     # correct
car1 is car2    # check if 2 class instances are the same
var == None     # invalid
```

## Walrus (:=)

Walrus operator allows us to do an assignment inline with different expression.

```python
if (match := pattern.search(data)) is not None:
    # := is expression, while = is a statement
    print(result := a + b)
    details: dict := {'length': (length := len(data)), 'avg': sum / length}
```

## Comparing floats

```python
from math import isclose

a: float = .1 + .2
b: float = .3

print(f'{a} == {b}?', a == b)                  # False
print(f'{a} == {b}?', isclose(a, b, rel_tol=.001))   # True
# rel_tol (relative tolerance): 1 = 100%, 0.1 = 10%, 0.01 = 1%, etc.

a: float = 0.999
b: float = 1.000

print(f'{a} == {b}?', isclose(a, b, abs_tol=.001)) # False
print(f'{a} == {b}?', isclose(a, b, abs_tol=.002)) # True
# abs_tol (absolute tolerance): difference below it will be considered as tolerant
```

It is possible to specify both `rel_tol` and `abs_tol` — then the expression will be tolerant if **one of** these conditions is met.

## Formatting variables (old style)

```python
percentage = 50
formatted = "The success rate is %d%%." % percentage # %d is placeholder for int
print(formatted) # The success rate is 50%.

formatted = "The success rate is {}%.".format(percentage) # {} is placeholder
print(formatted) # The success rate is 50%.
```

## Formatting variables (using f-strings)

```python
big_number: int = 1_620_000_000
print(f'{big_number:.2e}') # 1.62e+09

n: int = 10000000000
print(f'{n:_}') # 10_000_000_000
print(f'{n:,}') # 10,000,000,000
# only "_" and "," are allowed as separator
number: float = 1000000.1234567
print(f'{number:,.2f}') # 1,000,000.12
percent: float = 0.5555555
print(f'{percent:.2%}') # 55.56%
print(f'{percent:.0%}') # 56%

a: float = 0.1
b: float = 0.2
print(f'{a + b =:.1f}') # a + b = 0.3

name: str = 'Sergey'
print(f'{name:10}: world')   # Sergey    : world
print(f'{name:<10}: world')  # Sergey    : world
print(f'{name:>10}: world')  #     Sergey: world
print(f'{name:^10}: world')  #   Sergey  : world
print(f'{name:•>10}: world') # ••••Sergey: world
print(f'{name:@>10}: world') # @@@@Sergey: world
# it occupies 10 spaces

name: str = 'Sergey'
print(f'{name = }')   # name = 'Sergey'
print(f'{name = !s}') # name = Sergey
# !s is string representation,
# !r is a representation of a variable,
# !a is ASCII representation

print(f'{add(5, 10) =}') # add(5, 10) = 15

now: datetime = datetime.now()
print(f'{now:%d.%m.%y}') # 22.04.2024

date_spec: str = '%d.%m.%y'
print(f'{now:{date_spec}}') # 22.04.2024
```

## Raw string (r-string)

```python
path: str = r'\Users\Sergey\Documents\file.txt'

user: str = 'Sergey'
path: str = fr'\Users\{user}\Documents\file.txt'
```

## Byte string (b-string)

```python
user: bytes = b'Sergey'
path: bytes = rb'\Users\Sergey\Documents\file.txt'

msg: bytes = fb'Hello {user}!' # Error

# converting string to bytes
msg: bytes = f'Hello {user}!'.encode()
# converting bytes to string
user: str = b'Sergey'.decode()

smile: bytes = b"😊" # Error
smile: bytes = "😊".encode()
smile: str = bytes([240, 159, 152, 138]).decode('utf-8') # 😊
```

<!-- nav -->
---

← [Python — Basics](01-basics.md) · [Index](../README.md) · [Python — Flow](03-flow.md) →
<!-- nav -->
