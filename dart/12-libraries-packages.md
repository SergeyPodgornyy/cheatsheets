# Dart — Libraries & Packages

*Source: https://dart.dev/language/libraries*

## Imports

Every Dart file is a **library**. Use `import` to bring in others by URI scheme.

```dart
import 'dart:js_interop';            // dart:    -> built-in SDK libraries
import 'package:test/test.dart';     // package: -> packages from pub
import 'src/utils.dart';             // relative -> files in your own package
```

### Prefix with `as`

Disambiguate conflicting names.

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

Element element1 = Element();         // from lib1
lib2.Element element2 = lib2.Element(); // from lib2, namespaced
```

### show / hide

Import only part of a library's API.

```dart
import 'package:lib1/lib1.dart' show foo;   // only foo
import 'package:lib2/lib2.dart' hide foo;   // everything except foo
```

### Deferred (lazy) load

Defer loading a library until it's first needed (smaller startup; web).

```dart
import 'package:greetings/hello.dart' deferred as hello;

Future<void> greet() async {
  await hello.loadLibrary();   // returns a Future; loads once even if called again
  hello.printGreeting();
}
// NOTE: can't reference deferred-library TYPES in the importing file.
// Not supported by the `dart` tool for non-web (Flutter has deferred components).
```

### Library privacy

Dart has **no** `public`/`private`/`protected` keywords. An identifier with a leading underscore `_` is **library-private**.

```dart
int _counter = 0;            // private to its library (file)
class _Cache { }             // private class

/// A really great test library.
@TestOn('browser')
library;                     // optional library directive (annotations/docs)
```

## pubspec.yaml

The package manifest at the project root.

```yaml
name: newtify
description: >-
  Have you been turned into a newt?  Would you like to be?
  This package can help. It has all of the
  newt-transmogrification functionality you have been looking
  for.
version: 1.2.3
homepage: https://example-pet-store.com/newtify
documentation: https://example-pet-store.com/newtify/docs

environment:
  sdk: '^3.2.0'

dependencies:
  efts: ^2.0.4
  transmogrify: ^0.4.0

dev_dependencies:
  test: '>=1.15.0 <2.0.0'
```

| Field | Meaning |
| --- | --- |
| `name` | required; lowercase + underscores `[a-z0-9_]`, a valid Dart identifier |
| `version` | three dot-separated numbers, optional `+build` / `-prerelease` (semver) |
| `description` | required to publish; ~60-180 chars of plain text |
| `environment` | SDK constraint **required** — omitting it fails `dart pub get` |
| `dependencies` | runtime dependencies |
| `dev_dependencies` | dependencies only needed during development (tests, lints) |
| `dependency_overrides` | temporarily force a specific version (don't publish with these) |

Flutter SDK constraint:

```yaml
environment:
  sdk: ^3.2.0
  flutter: '>=3.22.0'

dependencies:
  flutter:
    sdk: flutter        # SDK package, not from pub.dev
```

## pub commands

Pattern: `dart pub <subcommand>` (or `flutter pub <subcommand>` for Flutter apps).

| Command | Description |
| --- | --- |
| `dart pub get` | retrieve dependencies (uses lock file if present) |
| `dart pub add <pkg>` | add a package dependency |
| `dart pub remove <pkg>` | remove a package dependency |
| `dart pub upgrade` | newest versions honoring constraints (ignores lock file) |
| `dart pub outdated` | check out-of-date deps + upgrade advice |
| `dart pub downgrade` | lowest allowed versions (test the lower range) |
| `dart pub deps` | list all dependencies used |
| `dart pub publish` | upload the package to pub.dev |
| `dart pub global activate <pkg>` | make a package globally runnable |
| `dart run <pkg>:<cmd>` | run a command-line app/script |

```console
$ dart pub get      # non-Flutter package
$ flutter pub get   # Flutter package
```

## dart:convert — JSON

```dart
import 'dart:convert';
```

### Decode: JSON string → Dart object

```dart
var jsonString = '''
  [
    {"score": 40},
    {"score": 80}
  ]
''';

var scores = jsonDecode(jsonString);   // dynamic; List/Map/num/String/bool/null
assert(scores is List);
var firstScore = scores[0];
assert(firstScore is Map);
assert(firstScore['score'] == 40);
```

### Encode: Dart object → JSON string

```dart
var scores = [
  {'score': 40},
  {'score': 80},
  {'score': 100, 'overtime': true, 'special_guest': null},
];
var jsonText = jsonEncode(scores);
// '[{"score":40},{"score":80},{"score":100,"overtime":true,"special_guest":null}]'
```

Only `int`/`double`/`String`/`bool`/`null`/`List`/`Map` (string keys) are directly encodable. For other types, define a `toJson()` method (the encoder calls it) or pass a `toEncodable` function as the 2nd arg.

```dart
class Point {
  final int x, y;
  Point(this.x, this.y);
  Map<String, dynamic> toJson() => {'x': x, 'y': y}; // encoder calls this
}
jsonEncode(Point(1, 2)); // {"x":1,"y":2}
```

### UTF-8

```dart
var funnyWord = utf8.decode(utf8Bytes);                    // bytes -> String
Uint8List encoded = utf8.encode('Îñţérñåţîöñåļîžåţîờñ');     // String -> bytes

// Stream decode, line by line:
var lines = utf8.decoder.bind(inputStream).transform(const LineSplitter());
```

<!-- nav -->
---

← [Dart — Async (Future, Stream, Isolate)](11-async.md) · [Index](README.md) · [Dart — Tooling, Metadata & Docs](13-misc.md) →
<!-- nav -->
