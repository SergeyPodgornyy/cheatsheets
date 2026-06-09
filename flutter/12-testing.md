# Flutter — Testing

*Source: https://docs.flutter.dev/testing/overview*

## Three Test Types

| Type | Tests | Tradeoff |
| --- | --- | --- |
| **Unit** | a single function, method, or class | fastest, cheapest; no UI / no real I/O |
| **Widget** | a single widget in isolation | runs in a test environment, no device needed |
| **Integration** | a complete app (or large slice) | slowest, most expensive; needs a real device/emulator |

```dart
// Rule of thumb: many cheap unit + widget tests, fewer expensive integration tests.
// Higher confidence costs more time and maintenance — balance accordingly.
```

## Unit Tests

Use `flutter_test` (re-exports `package:test`). `test` declares a case, `group` clusters related cases, `expect` asserts.

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('Counter value increments', () {
    final counter = Counter();
    counter.increment();
    expect(counter.value, 1); // actual, expected (or matcher)
  });

  group('Counter', () { // group related tests under one label
    test('starts at zero', () {
      expect(Counter().value, 0);
    });
  });
}
```

Common matchers:

| Matcher | Passes when |
| --- | --- |
| `equals(x)` | value equals `x` (bare value also works: `expect(a, x)`) |
| `isTrue` / `isFalse` | boolean is true / false |
| `isNull` / `isNotNull` | value is / isn't null |
| `throwsException` | the callable throws |
| `greaterThan(n)` | value `> n` |

## Widget Tests

Use `testWidgets` + a `WidgetTester`. `pumpWidget` builds the widget; **finders** locate widgets; **matchers** assert how many were found.

```dart
testWidgets('MyWidget has a title and message', (WidgetTester tester) async {
  await tester.pumpWidget(const MyWidget(title: 'T', message: 'M'));

  final titleFinder = find.text('T');
  expect(titleFinder, findsOneWidget);
  expect(find.byType(FloatingActionButton), findsOneWidget);
});
```

Interaction — `tap`, then `pump` to rebuild after state changes:

```dart
await tester.tap(find.byType(FloatingActionButton));
await tester.pump(); // process the frame triggered by the tap
expect(find.text('1'), findsOneWidget);
```

### Finders

| Finder | Locates by |
| --- | --- |
| `find.text('x')` | rendered text |
| `find.byType(Widget)` | widget runtime type |
| `find.byKey(key)` | a `Key` |
| `find.byIcon(Icons.add)` | icon data |

### Match Matchers

| Matcher | Passes when |
| --- | --- |
| `findsOneWidget` | exactly one match |
| `findsNothing` | zero matches |
| `findsWidgets` | one or more matches |
| `findsNWidgets(n)` | exactly `n` matches |

## Running Tests

```console
$ flutter test
```

## Integration Tests

Drive a full, running app. Use the `integration_test` package and run on a **real device or emulator** (not a headless test runtime).

```console
$ flutter test integration_test
```

<!-- nav -->
---

← [Flutter — Theming & Animations](11-theming-animations.md) · [Index](../README.md) · [Flutter — Slivers & Advanced Scrolling](13-slivers-scrolling.md) →
<!-- nav -->
