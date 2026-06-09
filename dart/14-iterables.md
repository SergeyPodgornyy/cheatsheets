# Dart — Iterables

*Source: https://dart.dev/codelabs/iterables*

An **`Iterable`** is a collection of elements accessed sequentially. `List` and `Set` are `Iterable`; `Map` exposes iterables via `.entries`, `.keys`, `.values`. `Iterable` is abstract — you can't instantiate it directly, and it has **no `[]` operator** (index access isn't guaranteed efficient). Most methods are **lazy** — the work runs only when iterated.

## Accessing elements

```dart
Iterable<int> iterable = [1, 2, 3];
int value = iterable.elementAt(1);   // 2 — by index
iterable.first;                       // 1  (throws StateError if empty)
iterable.last;                        // 3  (can be slow; throws if empty)
[1].single;                           // the sole element (throws if not exactly 1)

for (final element in iterable) {     // idiomatic sequential access
  print(element);
}
```

`firstWhere` / `singleWhere` — first (or sole) match; throw `StateError` if none and no `orElse`:

```dart
var found = items.firstWhere(
  (item) => item.length > 10,
  orElse: () => 'None!',   // fallback instead of throwing
);
var sole = items.singleWhere((e) => e.startsWith('M') && e.contains('a'));
```

## Checking conditions

```dart
items.any((item) => item.contains('a'));    // true if AT LEAST ONE matches
items.every((item) => item.length >= 5);    // true if ALL match
items.contains('Mac');                        // membership

bool anyUnder18(Iterable<User> users) => users.any((u) => u.age < 18);
bool everyOver13(Iterable<User> users) => users.every((u) => u.age > 13);
```

## Filtering

```dart
var evens = numbers.where((n) => n.isEven);   // lazy — keeps matching elements
var strings = items.whereType<String>();      // keep only elements of a type

const numbers = [1, 3, -2, 0, 4, 5];
numbers.takeWhile((n) => n != 0);   // (1, 3, -2) — elements BEFORE first match
numbers.skipWhile((n) => n != 0);   // (0, 4, 5)  — from first non-match onward
numbers.take(2);                    // (1, 3)     — first 2
numbers.skip(2);                    // (-2, 0, 4, 5)
```

## Mapping & expanding

```dart
Iterable<int> tens = numbers.map((n) => n * 10);          // transform each
Iterable<String> strs = numbers.map((n) => n.toString());

// expand — flatten: each element maps to 0+ elements
[1, 2, 3].expand((n) => [n, n * 10]); // (1, 10, 2, 20, 3, 30)
```

## Reducing

```dart
// reduce — combine to a single value of the SAME type (throws if empty)
[1, 2, 3].reduce((a, b) => a + b);            // 6

// fold — like reduce but with a seed; works on empty, allows a different type
[1, 2, 3].fold(0, (sum, n) => sum + n);       // 6
<int>[].fold(0, (sum, n) => sum + n);         // 0 — safe on empty
['a', 'b'].fold('', (acc, s) => acc + s);     // 'ab'
```

## Other common methods

```dart
list.forEach(print);              // run a function per element (eager)
iterable.toList();                // materialize to a List
iterable.toSet();                 // to a Set (dedupes)
['a', 'b', 'c'].join('-');        // 'a-b-c'
[3, 1, 2].sort();                 // in-place on a List (not lazy)
[1, 2].followedBy([3, 4]);        // (1, 2, 3, 4) — lazy concatenation
iterable.length;                  // count
iterable.isEmpty / .isNotEmpty;
```

**Lazy gotcha:** `map`/`where` don't run until iterated. Chaining them stays lazy until a terminal op (`toList`, `forEach`, `reduce`, `first`, ...) pulls the values:

```dart
var pipeline = numbers.where((n) => n.isEven).map((n) => n * 10);
// nothing computed yet
var result = pipeline.toList();   // NOW it runs
```

<!-- nav -->
---

← [Dart — Tooling, Metadata & Docs](13-misc.md) · [Index](../README.md) · [Flutter — Basics](../flutter/01-basics.md) →
<!-- nav -->
