# Dart — Error Handling

*Source: https://dart.dev/language/error-handling*

## Exception vs Error

Dart has two predefined hierarchies, both with subtypes. All exceptions are **unchecked**: no method declares what it throws, and the compiler never forces a catch.

```dart
// Error  -> a programming BUG. Should crash, NOT be caught. Fix the code.
//   e.g. ArgumentError, RangeError, StateError, UnimplementedError, AssertionError
// Exception -> a recoverable runtime condition you can reasonably handle.
//   e.g. FormatException, IOException, TimeoutException
```

You may `throw` any non-null object, but production code should throw `Exception`/`Error` types (not raw strings).

```dart
throw FormatException('Expected at least 1 section');
throw 'Out of llamas!';   // legal — arbitrary object, but discouraged
```

An **uncaught** error suspends the current isolate and usually terminates the program.

## throw

`throw` is an **expression**, so it works in arrow functions and `??`/ternary positions.

```dart
void distanceTo(Point other) => throw UnimplementedError(); // throw as expression

final name = input ?? throw ArgumentError('name required'); // in ?? position
```

## try / on / catch

`on` selects the **type**; `catch` binds the thrown **object**.

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {        // on  -> match by type, no object needed
  buyMoreLlamas();
}
```

### catch (e) vs catch (e, s)

```dart
try {
  // ...
} on Exception catch (e) {
  print('Exception details:\n $e');         // e = the thrown object
} catch (e, s) {
  print('Exception details:\n $e');          // e = object
  print('Stack trace:\n $s');                // s = StackTrace
}
```

## Multiple on clauses

First matching type wins — order **specific → general**. A bare `catch` is the catch-all and must come last.

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {                   // most specific first
  buyMoreLlamas();
} on Exception catch (e) {                     // any Exception
  print('Unknown exception: $e');
} catch (e) {                                  // anything (incl. non-Exception)
  print('Something really unknown: $e');
}
```

## rethrow

Partially handle, then propagate the **same** error (preserving the stack trace) to an outer handler.

```dart
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++);                             // runtime type error
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow;                                   // pass it on, unchanged
  }
}
```

## finally

Always runs — whether or not an exception was thrown, and after any matching `catch`.

```dart
try {
  breedMoreLlamas();
} finally {
  cleanLlamaStalls();        // runs even if breedMoreLlamas() throws
}

try {
  breedMoreLlamas();
} catch (e) {
  print('Error: $e');        // handle first
} finally {
  cleanLlamaStalls();        // then always clean up
}
```

## assert

`assert(condition, [message])` throws `AssertionError` when the condition is false. **Dev-only**: stripped in production — neither the condition nor the message argument is evaluated.

```dart
assert(text != null);
assert(number < 100);
assert(urlString.startsWith('https'),
    'URL ($urlString) should start with "https".');   // optional message
```

```console
$ dart run --enable-asserts main.dart   # enable asserts on the command line
```

Enabled in Flutter debug mode; ignored in production (e.g. `dart compile exe`, release builds).

## Common built-in types

| Type | Meaning |
| --- | --- |
| `FormatException` | string/data not in expected format — `int.parse('x')` |
| `UnimplementedError` | method/operation not yet implemented (an `Error`) |
| `ArgumentError` | argument out of allowed range/shape — `ArgumentError.value(x)` |
| `RangeError` | index/value outside valid range — `list[10]` on shorter list |
| `StateError` | object in wrong state for the call — `iterable.first` on empty |
| `AssertionError` | failed `assert` (dev-only) |

<!-- nav -->
---

← [Dart — Generics](09-generics.md) · [Index](README.md) · [Dart — Async (Future, Stream, Isolate)](11-async.md) →
<!-- nav -->
