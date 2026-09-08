# Dart: Types & Collections

*Source: https://dart.dev/language/built-in-types*

## Numbers

`int` (up to 64 bits) and `double` (64-bit IEEE 754) are both subtypes of `num`.

```dart
var x = 1;               // int
var hex = 0xDEADBEEF;    // int, hex literal
var y = 1.1;             // double
var exponents = 1.42e5;  // double: 142000.0

num x = 1;
x += 2.5;                // OK: num holds both int and double

double z = 1;            // == double z = 1.0 (int literal coerced to double if target is double)
```

### parse / toString

```dart
var one = int.parse('1');                        // 1
var onePointOne = double.parse('1.1');           // 1.1
String oneAsString = 1.toString();               // '1'
String piAsString = 3.14159.toStringAsFixed(2);  // '3.14'
```

### Digit separators

Underscores improve readability; ignored by the compiler.

```dart
var n1 = 1_000_000;
var n2 = 0.000_000_000_01;
var n3 = 0x00_14_22_01_23_45;
var n4 = 555_123_4567;
var n5 = 100__000_000__000_000;  // multiple underscores allowed
```

### Bitwise

```dart
assert((3 << 1) == 6);   // shift left
assert((3 | 4) == 7);    // OR
assert((3 & 4) == 0);    // AND
```

`num` provides `+ - / * abs() ceil() floor()`; more in `dart:math`. Number literals are compile-time constants:

```dart
const msPerSecond = 1000;
const secondsUntilRetry = 5;
const msUntilRetry = secondsUntilRetry * msPerSecond; // const arithmetic
```

## Strings

A `String` is a sequence of **UTF-16 code units**. Single or double quotes.

```dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

### Interpolation

Use `$identifier` for a simple identifier, `${expression}` for anything else.

```dart
var s = 'string interpolation';
'Dart has $s, which is very handy.';
'${s.toUpperCase()} is very handy!';   // STRING INTERPOLATION is very handy!
```

### Concatenation

Adjacent string literals concatenate; `+` also works.

```dart
var s1 = 'String ' 'concatenation' " works even over line breaks.";
var s2 = 'The + operator ' + 'works, as well.';
```

### Multi-line & raw

```dart
var s1 = '''
You can create
multi-line strings like this one.
''';
var s2 = """This is also a
multi-line string.""";

var raw = r'In a raw string, not even \n gets special treatment.';
```

### const strings

Interpolated values in a `const` string must themselves be const (null / numeric / string / boolean).

```dart
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';
const validConstString = '$aConstNum $aConstBool $aConstString';
// const invalid = '$aNum $aBool $aString'; // ERROR: non-const interpolated values
```

`==` on strings checks **code-unit-sequence equivalence**.

### Common methods

| Method | Example |
| --- | --- |
| `length` | `'abc'.length` → `3` |
| `toUpperCase()` | `'abc'.toUpperCase()` → `'ABC'` |
| `toLowerCase()` | `'ABC'.toLowerCase()` → `'abc'` |
| `trim()` | `'  hi  '.trim()` → `'hi'` |
| `split(pat)` | `'a,b,c'.split(',')` → `['a','b','c']` |
| `substring(s, [e])` | `'hello'.substring(1, 3)` → `'el'` |
| `contains(p)` | `'hello'.contains('ell')` → `true` |
| `replaceAll(f, r)` | `'a-b-c'.replaceAll('-', '_')` → `'a_b_c'` |
| `startsWith(p)` | `'hello'.startsWith('he')` → `true` |
| `indexOf(p)` | `'hello'.indexOf('l')` → `2` (`-1` if absent) |
| `padLeft(w, [pad])` | `'7'.padLeft(3, '0')` → `'007'` |

## Booleans

Only `true` and `false` have type `bool`. There is no truthy/falsy, so you must check explicitly.

```dart
var fullName = '';
assert(fullName.isEmpty);          // check emptiness explicitly

var hitPoints = 0;
assert(hitPoints == 0);            // check zero explicitly

var unicorn = null;
assert(unicorn == null);           // check null explicitly

var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);      // check NaN explicitly

// if ('') {}   // ERROR: a String is not a bool
```

## Runes & grapheme clusters

`runes` are the **Unicode code points** of a string. `\uXXXX` for 4-digit code points, `\u{...}` otherwise. Many user-perceived characters (emoji, flags) span multiple code points, so use the `characters` package for grapheme clusters.

```dart
import 'package:characters/characters.dart';

var hi = 'Hi 🇩🇰';
print(hi.characters.last); // 🇩🇰 (one grapheme cluster, not raw code units)

var heart = '♥';      // ♥
var laugh = '\u{1f606}';   // 😆, needs braces for a non-4-digit value
```

## Object / dynamic / Null / Never

- `Object` is the superclass of all Dart objects except `Null`.
- `dynamic` disables static checks; member access resolves at runtime.
- `Null` is the type of `null`.
- `Never` has no values; it is the type of an expression that never completes (e.g. always throws).
- Other special types: `Enum`, `Future`/`Stream`, `Iterable`, `void`.

## Lists

Zero-based, with `.length` and `[]` subscript.

```dart
var list = [1, 2, 3];
var planes = ['Car', 'Boat', 'Plane'];
assert(list.length == 3);
assert(list[1] == 2);
list[1] = 1;             // mutate by index

var constantList = const [1, 2, 3];
// constantList[1] = 1;  // ERROR: const list is immutable
```

Trailing comma is allowed.

## Sets

Unordered collection of unique items. `add()` / `addAll()`, `.length`.

```dart
var halogens = {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};

var names = <String>{};  // empty typed set
// Set<String> names = {}; // also works
// var names = {};         // GOTCHA: this is a MAP, not a set!

var elements = <String>{};
elements.add('fluorine');
elements.addAll(halogens);

final constantSet = const {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};
```

**Gotcha:** `{}` defaults to `Map<dynamic, dynamic>` (map literals came first). Always type empty set literals as `<T>{}`.

## Maps

Key → value pairs with unique keys. `[]=` to add, `[]` to read; missing key returns `null`.

```dart
var gifts = {'first': 'partridge', 'second': 'turtledoves', 'fifth': 'golden rings'};
var nobleGases = {2: 'helium', 10: 'neon', 18: 'argon'};

var gifts2 = Map<String, String>();
gifts2['first'] = 'partridge';
assert(gifts2['first'] == 'partridge');
assert(gifts2['fifth'] == null);   // missing key → null (no error)
gifts2['fourth'] = 'calling birds';
assert(gifts2.length == 2);   // 'first' + 'fourth'

final constantMap = const {2: 'helium', 10: 'neon', 18: 'argon'};
```

## Records

Anonymous, immutable, aggregate types: fixed-size, heterogeneous, and typed. They use structural typing, so the shape determines the type. Fields have getters, no setters. Auto-generated `==` and `hashCode`: equal if same shape and values; named-field *order* doesn't affect equality. (Records are also covered in the patterns file.)

```dart
var record = ('first', a: 2, b: true, 'last'); // mixed positional + named

(String, int) record2;
record2 = ('A string', 123);          // positional fields

({int a, bool b}) record3;
record3 = (a: 123, b: true);          // named fields
```

Named fields are part of the type; positional field names are documentation only:

```dart
({int a, int b}) recordAB = (a: 1, b: 2);
({int x, int y}) recordXY = (x: 3, y: 4);
// recordAB = recordXY;  // ERROR: named fields differ → different types

(int a, int b) posAB = (1, 2);
(int x, int y) posXY = (3, 4);
posAB = posXY;           // OK: positional names are just documentation
```

Field access: positional via `$1`, `$2`, ...; named by name.

```dart
var record = ('first', a: 2, b: true, 'last');
print(record.$1); // 'first'
print(record.a);  // 2
print(record.b);  // true
print(record.$2); // 'last'  ($N counts only positional fields)
```

Multiple returns + destructuring:

```dart
(String name, int age) userInfo(Map<String, dynamic> json) {
  return (json['name'] as String, json['age'] as int);
}

var (name, age) = userInfo(json);        // positional destructuring
final (:name, :age) = userInfo(json);    // named destructuring

typedef ButtonItem = ({String label, Icon icon, void Function()? onPressed});
```

## Spread `...` / `...?`

```dart
var a = [1, 2, null, 4];
var items = [0, ...a, 5];        // [0, 1, 2, null, 4, 5]

// Null-aware spread, skips a null collection:
List<int>? maybe = null;
var b = [1, null, 3];
var merged = [0, ...?maybe, ...?b, 4]; // [0, 1, null, 3, 4]
// var x = [...maybe];           // ERROR: nullable spread needs ...?
```

## Collection-if / collection-for / null-aware element

```dart
// collection-if:
var includeItem = true;
var items = [0, if (includeItem) 1, 2, 3];        // [0, 1, 2, 3]
var items2 = [0, if (name == 'orange') 1 else 10, 2, 3];

// if-case inside a collection:
Object data = 123;
var typeInfo = [
  if (data case int i) 'Data is an integer: $i',
  if (data case String s) 'Data is a string: $s',
];

// collection-for:
var numbers = [2, 3, 4];
var squares = [1, for (var n in numbers) n * n, 7]; // [1, 4, 9, 16, 7]
var counted = [1, for (var x = 5; x > 2; x--) x, 7]; // [1, 5, 4, 3, 7]

// nested (list-comprehension style):
var nums = [1, 2, 3, 4, 5, 6, 7];
var evens = [0, for (var n in nums) if (n.isEven) n, 8]; // [0, 2, 4, 6, 8]

// null-aware element (leaf): `?expr` drops the slot when expr is null:
int? absentValue = null;
int? presentValue = 3;
var list = [1, ?absentValue, ?presentValue, absentValue, 5]; // [1, 3, null, 5]
```

<!-- nav -->
---

← [Dart: Basics](01-basics.md) · [Index](README.md) · [Dart: Operators](03-operators.md) →
<!-- nav -->
