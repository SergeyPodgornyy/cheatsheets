# Dart: Functions

*Source: https://dart.dev/language/functions*

Functions are **objects** (type `Function`); assignable to variables and passable as arguments.

## Declaration

Types are recommended but optional (omitting them is discouraged for public APIs).

```dart
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

isNoble(atomicNumber) {            // types omitted: works, but not recommended
  return _nobleGases[atomicNumber] != null;
}
```

## Arrow `=>`

Shorthand for a body that is a single expression only. `=> expr` means `{ return expr; }`.

```dart
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;

// => takes an expression, not a statement:
// bool f() => if (x) ...;  // ERROR
```

## Parameters

Required positional, optionally followed by either named or optional positional, but not both.

### Required positional

```dart
void greet(String name, String greeting) { ... }
greet('Dash', 'Hello');
```

### Named `{}`

Wrapped in `{}`; optional unless marked `required`.

```dart
void enableFlags({bool? bold, bool? hidden}) { ... }
enableFlags(bold: true, hidden: false);
```

Defaults must be compile-time constants:

```dart
void enableFlags({bool bold = false, bool hidden = false}) { ... }
enableFlags(bold: true);  // hidden defaults to false
```

`required` named (no default, must be supplied):

```dart
const Scrollbar({super.key, required Widget child});
```

Named args can be placed anywhere in the call:

```dart
repeat(times: 2, () { ... });  // named arg before the positional fn
```

### Optional positional `[]`

Wrapped in `[]`; a trailing set of positional params that may be omitted.

```dart
String say(String from, String msg, [String? device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}

say('Bob', 'Howdy');                  // 'Bob says Howdy'
say('Bob', 'Howdy', 'smoke signal');  // 'Bob says Howdy with a smoke signal'
```

With a default:

```dart
String say(String from, String msg, [String device = 'carrier pigeon']) { ... }
```

## main()

Top-level entry point; returns `void`; optional `List<String>` of args.

```dart
void main() {
  print('Hello, World!');
}

void main(List<String> arguments) {
  print(arguments);
}
// $ dart run args.dart 1 test   →  [1, test]
```

## Functions as first-class objects

Assign to variables, pass as arguments.

```dart
void printElement(int element) {
  print(element);
}
var list = [1, 2, 3];
list.forEach(printElement);  // pass a function as an argument

var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';  // assign to a variable
```

## Function types

A function's type is `ReturnType Function(params)`, including named params.

```dart
void greet(String name, {String greeting = 'Hello'}) => print('$greeting $name!');

void Function(String, {String greeting}) g = greet;
g('Dash', greeting: 'Howdy');  // Howdy Dash!
```

## Anonymous functions (lambdas / closures)

Functions without a name; common as callbacks.

```dart
const list = ['apples', 'bananas', 'oranges'];

var uppercaseList = list.map((item) {
  return item.toUpperCase();
}).toList();

var uppercaseList = list.map((item) => item.toUpperCase()).toList();  // arrow form
```

## Lexical scope

Variable scope follows the curly braces outward, resolved at write-time from the source structure.

## Lexical closures

A function captures variables from its surrounding scope, keeping them alive.

```dart
Function makeAdder(int addBy) {
  return (int i) => addBy + i;  // closes over addBy
}

var add2 = makeAdder(2);
var add4 = makeAdder(4);
assert(add2(3) == 5);
assert(add4(3) == 7);
```

## Tear-offs

Reference a function, method, or constructor without parentheses. Preferred over wrapping it in a lambda.

```dart
charCodes.forEach(print);          // function tear-off
charCodes.forEach(buffer.write);   // method tear-off

var strings = charCodes.map(String.fromCharCode);  // constructor tear-off
var buffers = charCodes.map(StringBuffer.new);      // .new tear-off

// charCodes.forEach((code) => print(code));  // avoid: lambda when a tear-off works
```

## Return values

All functions return; if no return is specified, `return null;` is implicit (the body must allow null).

```dart
foo() {}
assert(foo() == null);  // implicit return null
```

### Multiple returns via records

```dart
(String, int) foo() {
  return ('something', 42);
}
var (name, count) = foo();  // destructure with a record pattern
```

## Generators

Lazily produce a sequence of values.

```dart
// Synchronous generator → Iterable, sync*, yield
Iterable<int> naturalsTo(int n) sync* {
  int k = 0;
  while (k < n) yield k++;
}

// Asynchronous generator → Stream, async*, yield
Stream<int> asynchronousNaturalsTo(int n) async* {
  int k = 0;
  while (k < n) yield k++;
}

// Recursive: yield* delegates to another generator
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    yield* naturalsDownFrom(n - 1);
  }
}
```

## external

Declares a function whose body is implemented elsewhere (e.g. native/interop code).

```dart
external void someFunc(int i);
```

<!-- nav -->
---

← [Dart: Control Flow](04-control-flow.md) · [Index](README.md) · [Dart: Patterns & Destructuring](06-patterns-records.md) →
<!-- nav -->
