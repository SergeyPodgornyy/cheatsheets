# Python — Comprehension and Unpacking

## Comprehension: Inline iteration with condition

```python
# it reads as [result for element in list if condition is True]
people: list[str] = ['James', 'Charlotte', 'Stephany', 'Mario', 'Sandra']

# long_names: list[str] = []
# for person in people:
#     if len(person) > 7:
#         long_names.append(person)

long_names: list[str] = [p for p in people if len(p) > 7]
print(f'Long names: {long_names}')
```

```python
def upper_everything(elements: list[str]) -> list[str]:
    return [element.upper() for element in elements]
```

```python
dict_comprehension = {i: i*i for x in range(10)}
list_comprehension = [x*x for x in range(10)]
set_comprehension = {i%3 for x in range(10)}
yield_comprehension = (2*x+5 for x in range(10**20)) # Generator
```

## Slicing

```python
numbers: list[int] = [1, 2, 3, 4, 5, 6]
print(numbers[0:3])   # [1, 2, 3] - index from 0 to 3 (exclusive)
print(numbers[:3])    # [1, 2, 3] - index until 3 (exclusive)
print(numbers[3:6])   # [4, 5, 6] - index from 3 to 6 (exclusive)
print(numbers[3:])    # [4, 5, 6] - index from 3 onwards
print(numbers[-1])    # 6 - last element from the list, as it goes backward

numbers[0:2] = [7, 8]
print(numbers)        # [7, 8, 3, 4, 5, 6]
```

### Steps

```python
numbers: list[int] = [1, 2, 3, 4, 5, 6]
print(numbers[0:4:2]) # [1, 3] - index from 0 to 4 (exclusive) with step 2
print(numbers[::2])   # [1, 3, 5] - all values with step 2
print(numbers[4:0])   # [] - without negative step is nothing (as it doesn't make sense)
print(numbers[4:0:-1])# [5, 4, 3, 2] - index from 4 to 0 (exclusive) with step 1
print(numbers[4:0:-2])# [5, 3] - index from 4 to 0 (exclusive) with step 2
```

### Reverse

```python
numbers: list[int] = [1, 2, 3, 4, 5]
print(numbers[::-1]) # [5, 4, 3, 2, 1]

def is_palindrome(text: str) -> bool:
    return text == text[::-1]
```

## Unpacking

```python
inputs = ['John', 'Smith', 'United States', 'blue', 'brown', 29]
first_name, last_name, *_, age = inputs

data = [1, 2, 3, 4, 5]
print(*data)                  # 1 2 3 4 5
print(*data, sep=",", end=".")# 1,2,3,4,5.

a, *b, c = 'Python'
print(a, b, c) # P ['y' 't' 'h' 'o'] n

*_, = 'Python'
print(_) # ['P' 'y' 't' 'h' 'o' 'n']

numbers: list[int] = [1, 2, 3, 4, 5]
params: dict[str, str] = {'sep': '-', 'end': '.'}
print(*numbers, **params)
```

<!-- nav -->
---

← [Python — Flow](03-flow.md) · [Index](README.md) · [Python — Functions](05-functions.md) →
<!-- nav -->
