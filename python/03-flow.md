# Python — Flow

## Condition

```python
if weather == 'clear':
    print('It is a nice day')
elif weather == 'cloudy':
    print('Weather could be better')
else:
    print('Unknown weather')
```

### Shorthand / Ternary

```python
# do this if TRUE else do this
print('Adult zone') if age >= 18 else print('Kids zone')

result: str = 'Adult' if age >= 18 else 'Young'
```

### In list

```python
colors = ['red', 'orange', 'green']
if "red" in colors:
    print('Color found')
```

## Match case

```python
match weather:
    case 'clear':
        print('It is a nice day')
    case 'cloudy':
        print('Weather could be better')
    case _:
        print('Unknown weather')
```

Match-case can also accept multiple arguments, that will be compared as a list:

```python
command: list[str] = input('Enter a command: ').split()

match command:
    case 'find', *images: # can also be as: ['find', *images]
        print(f'Finding: {images}...')
    case 'enlarge', image, amount:
        print(f'You enlarged {image} by {amount}x')
    case 'rename', image, new_name if len(new_name) > 3:
        print(f'{image} was renamed to {new_name}')
    case 'x' | 'delete', *images:
        print(f'Deleting: {images}...')
    case _:
        print('Command not found...')
```

## Loops

### For

```python
for i in range(5): # 0, 1, 2, 3, 4
    print(i)
else: # success block: no error or break
    print('Iteration completed')
```

### While

```python
while number > 0:
    if number > 100:
        break # stops iteration

    number -= 1
    if number % 2:
        continue # next iteration

    print(number)
else:
    print('Successfully completed all numbers')
```

### Enumerate

```python
people: list[str] = ['Anna', 'Bob', 'Chris', 'David', 'Fred']

for i, person in enumerate(people, 1): # enumerate with index starting 1
    print(f'{i} - {person}') # 1 - Anna, 2 - Bob, ...
```

## Passing block: keep it empty

`pass` acts as a **placeholder** and can be used anywhere we need to skip: *conditions, loops, functions*:

```python
def connect():
    pass
for i in range(3):
    pass
if age < 18:
    pass
```

**ellipsis** acts as a pass:

```python
def enter_club(name: str, age: int, has_id: bool) -> None:
    ...
```

## Index during iteration

Python does not store the index of original elements during iteration:

```python
people: list[str] = ['Anna', 'Bob', 'Chris', 'David', 'Fred']

for person in people:
    # ❗ Chris will never reach, as Python shifts indexes on remove
    print(f'{people.index(person)} - {person}')

    if person == 'Bob':
        people.remove(person) # shift indexes if people here
```

When there is a need to modify the list during iteration, better to use different variable:

```python
filtered: list[str] = []
for person in people:
    if person == 'Bob':
        continue

    filtered.append(person)
```

## Assertions

Assertions are used for debugging and won't be included in your final code. You can check anything with the condition, and if it is not true, then `AssertionError` will be thrown.

```python
assert lang == 'Python', 'Invalid language selected' # condition and error msg
```

Using `assert` for user input validation or critical checks is **dangerous** and considered as bad practice because assertions can be <u>completely removed</u> when running Python in optimized mode (`-O` or `-OO`). This means that if you use `assert` for essential validation, your program might behave unpredictably in production. Assertions are useful for **debugging and development**, but not for validating external input.

<!-- nav -->
---

← [Python — Operators](02-operators.md) · [Index](../README.md) · [Python — Comprehension and Unpacking](04-comprehension.md) →
<!-- nav -->
