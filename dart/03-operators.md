# Dart: Operators

*Source: https://dart.dev/language/operators*

## Precedence

Highest → lowest. Within a row, operators associate left-to-right unless noted.

| Description | Operators |
| --- | --- |
| postfix | `expr++` `expr--` `()` `[]` `?[]` `.` `?.` `!` |
| prefix | `-expr` `!expr` `~expr` `++expr` `--expr` `await expr` |
| multiplicative | `*` `/` `%` `~/` |
| additive | `+` `-` |
| shift | `<<` `>>` `>>>` |
| bitwise AND | `&` |
| bitwise XOR | `^` |
| bitwise OR | `\|` |
| relational & type-test | `>=` `>` `<=` `<` `as` `is` `is!` |
| equality | `==` `!=` |
| logical AND | `&&` |
| logical OR | `\|\|` |
| if-null | `??` |
| conditional | `expr ? expr : expr` |
| cascade | `..` `?..` |
| assignment | `=` `*=` `/=` `+=` `-=` `&=` `^=` ... |
| spread | `...` `...?` |

## Arithmetic

```dart
2 + 3 == 5;
5 - 2 == 3;
2 * 3 == 6;
5 / 2 == 2.5;            // / always returns a double
5 ~/ 2 == 2;             // ~/ integer division (truncates)
5 % 2 == 1;              // remainder
-(5);                    // unary minus
```

Increment / decrement: prefix returns the value *after* the change, postfix *before*:

```dart
var a = 0, b;
b = ++a;   // a=1, b=1   (pre-increment: change then use)
a = 0;
b = a++;   // a=1, b=0   (post-increment: use then change)
b = --a;   // a=0, b=0
a = 0;
b = a--;   // a=-1, b=0
```

## Equality & relational

```dart
==  !=  >  <  >=  <=
```

How `==` works: if either operand is `null`, returns `true` only when both are null; otherwise it invokes the `==` method on the left operand.

```dart
1 == 1;          // true
1 == 2;          // false
null == null;    // true
null == 1;       // false
```

Use `identical(a, b)` for object identity (Dart's equivalent of Python's `is`: same instance, not just equal value).

```dart
var a = [1, 2], b = [1, 2];
a == b;              // depends on List's == (false here: different instances)
identical(a, b);     // false: distinct objects
identical(a, a);     // true
```

## Type test

```dart
obj is  T            // true if obj implements T's interface
obj is! T            // negation
obj as  T            // cast, throws if obj isn't a T (or is null)
```

```dart
(employee as Person).firstName = 'Bob';   // cast then use; throws on null/wrong type

if (employee is Person) {                  // safe: promotes inside the block
  employee.firstName = 'Bob';
}
```

## Assignment

```dart
a = value;               // plain assignment
b ??= value;             // assign only if b is currently null
```

Compound assignment: `a op= b` is equivalent to `a = a op b`.

| | | | | |
| --- | --- | --- | --- | --- |
| `*=` | `/=` | `+=` | `-=` | `~/=` |
| `%=` | `<<=` | `>>=` | `>>>=` | `&=` |
| `^=` | `\|=` | `??=` | | |

```dart
var a = 2;
a *= 3;   // a = a * 3 → 6
```

## Logical

```dart
!expr            // NOT
expr1 && expr2   // AND (short-circuits)
expr1 || expr2   // OR  (short-circuits)

if (!done && (col == 0 || col == 3)) {
  // ...
}
```

## Bitwise & shift

```dart
&  |  ^  ~expr   // AND, OR, XOR, NOT(complement)
<<  >>  >>>      // shift left, shift right, unsigned (zero-fill) shift right

final value = 0x22;
final bitmask = 0x0f;
assert((value & bitmask) == 0x02);   // AND
assert((value | bitmask) == 0x2f);   // OR
assert((value ^ bitmask) == 0x2d);   // XOR
assert((value << 4) == 0x220);       // shift left
```

## Conditional expressions

Two operators that replace many `if-else` statements.

```dart
// ternary: condition ? then : else
var visibility = isPublic ? 'public' : 'private';

// ?? is if-null: left if non-null, else right
String playerName(String? name) => name ?? 'Guest';
```

## Cascade `..` and `?..`

Run a sequence of operations on the same object without repeating the receiver. The cascade expression evaluates to the *object*, not the last call's result.

```dart
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;
// equivalent to: var paint = Paint(); paint.color = ...; paint.strokeCap = ...; ...

// ?.. is the null-shorting cascade: use on the FIRST operation when the target may be null:
querySelector('#confirm')
  ?..text = 'Confirm'
  ..classes.add('important');
```

**Gotcha:** a cascade on a void-returning expression isn't allowed (e.g. you can't cascade off `print()`).

## Null-aware

```dart
obj?.member          // conditional access: null if obj is null (no throw)
list?[index]         // conditional index
expr ?? other        // if-null: expr, or other if expr is null
x ??= value;         // assign value only if x is currently null
expr!                // null assertion: throws at runtime if expr is null
```

```dart
String? name;
name?.length         // null  (short-circuits the whole access)
name ?? 'Guest'      // 'Guest'
name!.length         // throws, name is null
```

## Spread

Part of collection-literal syntax: `...` expands a collection, `...?` skips a null one. See Types & Collections for full examples.

```dart
var combined = [0, ...listA, ...?maybeListB, 5];
```

<!-- nav -->
---

← [Dart: Types & Collections](02-types-collections.md) · [Index](README.md) · [Dart: Control Flow](04-control-flow.md) →
<!-- nav -->
