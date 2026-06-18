# Dart — Patterns & Destructuring

*Source: https://dart.dev/language/patterns*

## What patterns do

A pattern **matches** a value, **destructures** it, or both.

- **Matching** checks shape / constant / equality / type.
- **Destructuring** breaks a value into parts and binds them to variables.

```dart
// Matching:
switch (number) {
  case 1: print('one');  // constant pattern
}
const a = 'a';
const b = 'b';
switch (obj) {
  case [a, b]: print('$a, $b');  // matches a 2-element list equal to [a, b]
}

// Destructuring:
var numList = [1, 2, 3];
var [a, b, c] = numList;  // binds a=1, b=2, c=3
print(a + b + c);         // 6

switch (list) {
  case ['a' || 'b', var c]: print(c);  // match + destructure
}
```

## Where patterns appear

### Variable declaration

Starts with `var`/`final`; destructures into new variables.

```dart
var (a, [b, c]) = ('str', [1, 2]);  // a='str', b=1, c=2
```

### Variable assignment

Assigns to **existing** variables — enables a no-temp swap.

```dart
var (a, b) = ('left', 'right');
(b, a) = (a, b);  // swap → b='left', a='right'  →  "right left"
```

### Switch & if-case

Case patterns are **refutable** (may fail to match).

```dart
switch (obj) {
  case 1:                     print('one');
  case >= first && <= last:   print('in range');
  case (var a, var b):        print('a = $a, b = $b');
  default:
}
```

### for / for-in

Destructure each element — e.g. `MapEntry` over a map.

```dart
Map<String, int> hist = {'a': 23, 'b': 100};
for (var MapEntry(key: key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
for (var MapEntry(:key, value: count) in hist.entries) { ... }  // :key shorthand
```

## Pattern types

### Logical-or `||`

Matches if **any** branch matches; share one case body.

```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false,
};
```

### Logical-and `&&`

Both subpatterns must match. **Cannot bind the same name twice.**

```dart
// case (var a, var b) && (var b, var c)  // ERROR — both bind 'b'
```

### Relational

`==` `!=` `<` `>` `<=` `>=` against a constant.

```dart
String asciiCharType(int char) {
  const space = 32;
  const zero = 48;
  const nine = 57;
  return switch (char) {
    < space            => 'control',
    == space           => 'space',
    > space && < zero  => 'punctuation',
    >= zero && <= nine => 'digit',
    _ => '',
  };
}
```

### Cast `as`

Asserts the type and binds; throws if the type is wrong.

```dart
(num, Object) record = (1, 's');
var (i as int, s as String) = record;  // i is int, s is String
```

### Null-check `?`

Matches only if **non-null**, binds the inner value as **non-nullable**.

```dart
String? maybeString = 'nullable with base type String';
switch (maybeString) {
  case var s?:  // s is non-nullable String
}
```

### Null-assert `!`

Binds the value, **throws** if it is null.

```dart
List<String?> row = ['user', null];
switch (row) {
  case ['user', var name!]:  // name is non-nullable; throws if null
}

(int?, int?) position = (2, 3);
var (x!, y!) = position;  // x, y non-nullable; throws if either is null
```

### Constant

A constant expression matches by equality. Literal collections need `const`.

```dart
case 1:               // matches the value 1
case const [a, b]:    // const needed to treat the list literal as a constant
```

### Variable

Binds the matched (or destructured) value; can be typed.

```dart
case (var a, var b):       // binds a, b
case (int a, String b):    // typed — also matches on type
```

### Identifier

A bare name in a **matching** context is a reference to a constant (not a new binding).

```dart
const c = 1;
switch (2) {
  case c:  print('match $c');   // compares against constant c
  default: print('no match');   // → no match
}
```

### Parenthesized

Controls precedence, like in expressions.

```dart
// x || y && z   ==   x || (y && z)
// (x || y) && z differs
```

### List

Matches a list; one **rest element** `...` allowed.

```dart
case [a, b]:                                  // exactly 2 elements
var [a, b, ..., c, d] = [1, 2, 3, 4, 5, 6, 7];      // a=1 b=2 c=6 d=7 (middle discarded)
var [a, b, ...rest, c, d] = [1, 2, 3, 4, 5, 6, 7];  // rest = [3, 4, 5]
```

### Map

Matches map keys; missing keys fail the match.

```dart
final {'foo': int? foo} = {};  // binds foo from key 'foo'
```

### Record

Destructures positional and named record fields.

```dart
var (myString: foo, myNumber: bar) = (myString: 'string', myNumber: 1);
var (:untyped, :int typed) = record;  // :field shorthand, optionally typed
```

### Object

Matches a type and destructures via getters.

```dart
switch (shape) {
  case Rect(width: var w, height: var h): ...
}
var Point(:x, :y) = Point(1, 2);  // :x shorthand binds getter x
```

### Wildcard `_`

Matches anything, binds nothing; can be typed to match on type only.

```dart
var [_, two, _] = list;  // keep only the middle element
switch (record) {
  case (int _, String _):
    print('First field is int and second is String.');
}
```

## Use cases

### Multiple returns

```dart
var (name, age) = userInfo(json);  // unpack a returned record
```

### Destructuring class instances

```dart
var Foo(:one, :two) = myFoo;  // bind getters one, two
```

### Validating JSON

```dart
// Unconditional destructure (throws if shape differs):
var {'user': [name, age]} = data;

// Validate + destructure in one check:
if (data case {'user': [String name, int age]}) {
  print('User $name is $age years old.');
}
```

## Sealed class exhaustive switch

Algebraic data types: a `sealed` supertype lets the compiler verify the switch covers **all** subtypes (no `default` needed). See the inheritance file for `sealed`.

```dart
sealed class Shape {}
class Square implements Shape { final double length; Square(this.length); }
class Circle implements Shape { final double radius; Circle(this.radius); }

double calculateArea(Shape shape) => switch (shape) {
  Square(length: var l) => l * l,
  Circle(radius: var r) => math.pi * r * r,
  // exhaustive: Square + Circle cover all of sealed Shape
};
```

<!-- nav -->
---

← [Dart — Functions](05-functions.md) · [Index](README.md) · [Dart — Classes](07-classes.md) →
<!-- nav -->
