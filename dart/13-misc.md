# Dart — Tooling, Metadata & Docs

*Source: https://dart.dev/language/metadata*

## Doc comments

Use `///` for documentation. Wrap member names in `[brackets]` to create resolved cross-references in generated docs.

```dart
/// Use [turnOn] to turn the power on instead.
void activate() => turnOn();   // [turnOn] links to the turnOn member
```

## Metadata / annotations

An **annotation** is `@` followed by a compile-time constant (often a `const` constructor call), placed before a declaration or directive.

### Built-in

```dart
@override                       // declares an intentional supertype override
void build() { ... }

class Television {
  /// Use [turnOn] to turn the power on instead.
  @Deprecated('Use turnOn instead')   // with a message
  void activate() { turnOn(); }

  @deprecated                          // lowercase: no message
  void legacy() { }

  void turnOn() { ... }
}

@pragma('vm:prefer-inline')     // tool/compiler hints
int hot() => 42;
```

### Custom annotation

Define a class with a `const` constructor, then apply it.

```dart
class Todo {
  final String who;
  final String what;
  const Todo(this.who, this.what);   // const constructor required
}

@Todo('Dash', 'Implement this function')
void doSomething() {
  print('Do something');
}
```

### package:meta

```dart
import 'package:meta/meta.dart';

@visibleForTesting   // public only so tests can reach it; warns on other uses
void resetCacheForTest() { }
// also: @awaitNotRequired, @immutable, @sealed, ...
```

## Tooling

| Command | Description |
| --- | --- |
| `dart format .` | format all code to the standard style |
| `dart analyze` | static analysis (uses `analysis_options.yaml` lints) |
| `dart fix --apply` | apply automated fixes/migrations |
| `dart test` | run tests (package:test) |
| `dart doc .` | generate API documentation |
| `dart compile exe` | compile to a native self-contained executable |
| `dart compile js` | compile to JavaScript |
| `dart compile aot-snapshot` | compile to an AOT snapshot |

## analysis_options.yaml

Configure the analyzer and enable lint rules.

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    - prefer_final_locals
    - avoid_print

analyzer:
  exclude:
    - build/**
```

## package:test — minimal

```dart
import 'package:test/test.dart';

void main() {
  group('String', () {                 // group related tests
    test('.split() splits on the delimiter', () {
      expect('foo,bar'.split(','), equals(['foo', 'bar'])); // expect(actual, matcher)
    });
  });
}
```

## dart:core highlights

| API | Example |
| --- | --- |
| `print` | `print('hello');` — write to stdout |
| `int.parse` | `int.parse('42'); // 42` (throws `FormatException` on bad input) |
| `DateTime.now()` | `DateTime now = DateTime.now();` |
| `Duration` | `Duration d = Duration(hours: 1, minutes: 30);` |
| `Uri.parse` | `Uri.parse('https://dart.dev');` |
| `RegExp` | `var re = RegExp(r'(\d+)');` — raw string for the pattern |

```dart
DateTime now = DateTime.now();
Duration d = Duration(hours: 1, minutes: 30);
var re = RegExp(r'(\d+)');
re.firstMatch('abc123')?.group(1);   // '123'
```

<!-- nav -->
---

← [Dart — Libraries & Packages](12-libraries-packages.md) · [Index](../README.md) · [Dart — Iterables](14-iterables.md) →
<!-- nav -->
