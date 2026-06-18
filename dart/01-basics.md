# Dart — Basics

*Source: https://dart.dev/language*

Dart is **statically typed** with **sound null safety**: every variable has a type (inferred or declared), and the compiler guarantees a non-nullable variable never holds null. Compiles to **native machine code**, **JavaScript**, and **WebAssembly (Wasm)**. Supports both **AOT** (ahead-of-time, fast startup) and **JIT** (just-in-time, hot reload during development) compilation.

## `main()` and hello world

Every app needs a top-level `main()` — the entry point.

```dart
void main() {
  print('Hello, World!');
}

// Command-line args via List<String>:
void main(List<String> args) {
  print(args); // e.g. [one, two] for: dart run prog.dart one two
}
```

## Variables

Style guide recommends **`var`** over type annotations for local variables.

```dart
var name = 'Bob';        // inferred String
String name = 'Bob';     // explicit type
Object name = 'Bob';     // override inference — widen to Object
```

**`dynamic`** disables static type checking (calls resolved at runtime); **`Object`** keeps checks but accepts any value.

```dart
dynamic anything = 'text';
anything = 42;           // OK — type can change, no static check
anything.foo();          // compiles; throws at RUNTIME if no such member

Object boxed = 'text';
// boxed.length;         // ERROR — Object has no `length`; need cast first
```

## `final` vs `const`

**`final`** = set once at runtime. **`const`** = compile-time constant (implicitly `final`).

```dart
final name = 'Bob';
final String nickname = 'Bobby';
name = 'Alice';          // ERROR: a final variable can only be set once

const bar = 1000000;
const double atm = 1.01325 * bar; // compile-time computed

final now = DateTime.now();  // OK — runtime value
const now = DateTime.now();  // ERROR — not a compile-time constant
```

A **`final` object's fields can still change**; a **`const` object and its fields are deeply immutable**.

### const collections

```dart
var foo = const [];
final bar = const [];
const baz = [];          // equivalent to `const []`

foo = [1, 2, 3];         // OK — was const [], but `foo` itself isn't const
baz = [42];              // ERROR: constant variables can't be assigned a value
```

`const` works with `as`, `is`, and spread inside the constant context:

```dart
const Object i = 3;
const list = [i as int];                       // typecast
const map = {if (i is int) i: 'int'};          // is + collection if
const set = {if (list is List<int>) ...list};  // spread
```

## `late`

**`late`** = non-nullable variable initialized *after* declaration, or lazy initialization.

```dart
late String description;
void main() {
  description = 'Feijoada!';
  print(description);    // Feijoada!
}
// Reading a `late` var before assignment → runtime error.

// With initializer → runs lazily on FIRST use (handy for expensive calls):
late String temperature = readThermometer(); // readThermometer() not called yet
```

## Null safety basics

Append **`?`** to make a type nullable. Non-nullable variables must be initialized before use.

```dart
int? lineCount;          // nullable — defaults to null
assert(lineCount == null);

int count;               // non-nullable — must be set before first read
String? name = null;     // explicitly nullable
```

Null-handling operators:

```dart
String? name;

name!                    // null assertion — throws at runtime if name is null
name ?? 'Guest'          // if-null — value of name, or 'Guest' if null
name ??= 'Guest';        // assign 'Guest' only if name is currently null
name?.length             // conditional access — null if name is null (no throw)
```

## Wildcard `_`

A **non-binding placeholder**: the initializer still runs, but the value isn't accessible. Multiple `_` are allowed in the same scope.

```dart
main() {
  var _ = 1;
  int _ = 2;             // both fine — no name collision
}

for (var _ in list) {}             // ignore loop value
try { throw '!'; } catch (_) {     // ignore caught error
  print('oops');
}

class T<_> {}                      // unused type parameter
void genericFunction<_>() {}
list.where((_) => true);           // ignore callback arg
```

## Comments

```dart
// Single-line comment.

/* Block comment —
   spans multiple lines. */

/// Doc comment. Supports markdown; used by `dart doc`.
/// Refer to elements in [brackets] to link them in generated docs.
void greet() {}
```

<!-- nav -->
---

← [Home](../README.md) · [Index](README.md) · [Dart — Types & Collections](02-types-collections.md) →
<!-- nav -->
