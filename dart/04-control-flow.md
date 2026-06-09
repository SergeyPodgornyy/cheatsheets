# Dart — Control Flow

*Source: https://dart.dev/language/branches*

## if / else if / else

Condition **must be `bool`** (no truthy/falsy coercion).

```dart
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}

if (1) { ... }  // ERROR — condition must be bool, not int
```

## if-case

Matches a **single pattern**; variables bound in the pattern come into scope in the body.

```dart
if (pair case [int x, int y]) return Point(x, y);  // matches & destructures

if (pair case [int x, int y]) {
  print('Was coordinate array $x,$y');  // x, y in scope here
} else {
  throw FormatException('Invalid coordinates.');
}
```

## for (C-style)

```dart
for (var i = 0; i < 5; i++) {
  message.write('!');
}
```

### Closure-capture gotcha

Dart captures the index **value** per iteration (JS captures the variable → would print `2 2`).

```dart
var callbacks = [];
for (var i = 0; i < 2; i++) {
  callbacks.add(() => print(i));
}
for (final c in callbacks) {
  c();  // prints 0 then 1  — Dart captures the value, not the variable
}
```

## for-in

Iterates any `Iterable`.

```dart
for (var candidate in candidates) {
  candidate.interview();
}
```

### for-in with pattern

Destructure each element inline.

```dart
for (final Candidate(:name, :yearsExperience) in candidates) {
  print('$name has $yearsExperience of experience.');
}
```

## forEach

```dart
var collection = [1, 2, 3];
collection.forEach(print);  // 1 2 3  — print torn off as the callback
```

## while / do-while

`while` checks **before** the body; `do-while` checks **after** (body runs at least once).

```dart
while (!isDone()) {
  doSomething();
}

do {
  printLine();
} while (!atEndOfPage());
```

## break / continue

`break` exits the loop; `continue` skips to the next iteration.

```dart
while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}

for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;  // skip rest of this iteration
  }
  candidate.interview();
}
```

Iterable alternative (often cleaner than break/continue):

```dart
candidates.where((c) => c.yearsExperience >= 5).forEach((c) => c.interview());
```

## Labels

`identifier:` before a loop; lets `break`/`continue` target an **outer** loop.

```dart
outerLoop:
for (var i = 1; i <= 3; i++) {
  for (var j = 1; j <= 3; j++) {
    print('i = $i, j = $j');
    if (i == 2 && j == 2) {
      break outerLoop;  // exits both loops; continue outerLoop; also valid
    }
  }
}
```

## switch statement

Each `case` is a **pattern**. Non-empty cases do **NOT** fall through — no `break` needed; they end implicitly (or with `continue`/`throw`/`return`). Use `default` or wildcard `_` for no match.

```dart
var command = 'OPEN';
switch (command) {
  case 'CLOSED':   executeClosed();
  case 'PENDING':  executePending();
  case 'APPROVED': executeApproved();
  case 'DENIED':   executeDenied();
  case 'OPEN':     executeOpen();
  default:         executeUnknown();
}
```

### Empty-case fallthrough + continue label

Empty cases **do** fall through. Use `break` to stop. Use a label + `continue` for non-sequential fallthrough.

```dart
switch (command) {
  case 'OPEN':
    executeOpen();
    continue newCase;  // jumps to the newCase label
  case 'DENIED':       // empty case — falls through
  case 'CLOSED':
    executeClosed();   // runs for DENIED and CLOSED
  newCase:
  case 'PENDING':
    executeNowClosed();  // runs for OPEN and PENDING
}
```

## switch expression

Produces a **value**; usable wherever an expression is allowed (not at the start of an expression statement). Differences vs the statement: no `case` keyword, body is a single expression, `=>` instead of `:`, cases comma-separated (trailing comma ok), every case needs a body (no fallthrough), `default` is written only as `_`.

```dart
var x = switch (y) { ... };

token = switch (charCode) {
  slash || star || plus || minus => operator(charCode),  // logical-or cases
  comma || semicolon            => punctuation(charCode),
  >= digit0 && <= digit9        => number(),              // relational + and
  _                             => throw FormatException('Invalid'),
};
```

## Exhaustiveness

Compile-time error if a value could match **no** case. `default`/`_` makes any switch exhaustive. Enums and **sealed** types are fully enumerable → exhaustive without a default.

```dart
sealed class Shape {}
class Square implements Shape { final double length; Square(this.length); }
class Circle implements Shape { final double radius; Circle(this.radius); }

double calculateArea(Shape shape) => switch (shape) {
  Square(length: var l) => l * l,
  Circle(radius: var r) => math.pi * r * r,
  // no default needed — Square + Circle cover all of sealed Shape
};
```

See the inheritance file for `sealed` classes.

## Guard clauses with `when`

A `when` boolean is evaluated **after** the pattern matches. If the guard is `false`, control **proceeds to the next case** (it does NOT exit the switch — unlike an `if` inside a case body).

```dart
switch (something) {
  case somePattern when some || boolean || expression:
    body;
}

var value = switch (something) {
  somePattern when some || boolean || expression => body,
};

if (something case somePattern when some || boolean || expression) {
  body;
}
```

```dart
// guard false → falls to next case (vs. `if`-in-body which would exit switch)
switch (pair) {
  case (int a, int b) when a > b: print('First element greater');
  case (int a, int b):            print('First element not greater');
}
```

## assert

`assert(condition, [message])` — throws `AssertionError` if the condition is `false`.

```dart
assert(text != null);
assert(number < 100, 'number must be < 100');
assert(urlString.startsWith('https'), 'URL ($urlString) should start with "https".');
```

**Dev-only**: assertions run in development/tests and are **stripped in production** — never rely on them for runtime validation.

<!-- nav -->
---

← [Dart — Operators](03-operators.md) · [Index](../README.md) · [Dart — Functions](05-functions.md) →
<!-- nav -->
