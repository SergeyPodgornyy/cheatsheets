# Dart: Generics

*Source: https://dart.dev/language/generics*

## Why generics

Generics give type safety and cut down on duplicated code. Types in angle brackets (`<...>`) parameterize a class or method. Convention for type params: `E, T, S, K, V`.

`List` is really `List<E>`, where `E` is the element type. Declaring it lets the analyzer catch wrong-typed inserts:

```dart
var names = <String>[];
names.addAll(['Seth', 'Kathy', 'Lars']);
// names.add(42);   // Error: 42 is not a String
```

Without generics you'd need a separate `StringCache`, `IntCache`, etc.; with one `Cache<T>` the same code serves every type.

## Generic collection literals

```dart
var names       = <String>['Seth', 'Kathy', 'Lars'];               // List<String>
var uniqueNames = <String>{'Seth', 'Kathy', 'Lars'};               // Set<String>
var pages       = <String, String>{                                 // Map<String, String>
  'index.html': 'Homepage',
  'robots.txt': 'Hints for web robots',
};
var nameSet = Set<String>.of(names);          // typed constructor
var views   = SplayTreeMap<int, View>();      // parameterized generic type
```

## Generic classes

Parameterize a class with one or more type variables.

```dart
abstract class Cache<T> {
  T getByKey(String key);
  void setByKey(String key, T value);
}
```

## Generic methods

Type params on methods/functions; the return and parameter types can reuse them.

```dart
T first<T>(List<T> ts) {
  T tmp = ts[0];
  return tmp;
}
```

## Bounds with extends

Restrict a type argument with `extends`.

```dart
class Foo<T extends Object> { }   // T must be non-nullable (any non-null Object)
```

```dart
class Foo<T extends SomeBaseClass> {
  String toString() => "Instance of 'Foo<$T>'";
}

var someBaseClassFoo = Foo<SomeBaseClass>();
var extenderFoo      = Foo<Extender>();        // Extender is a subtype, so OK

var foo = Foo();                               // T defaults to the bound
print(foo);                                    // Instance of 'Foo<SomeBaseClass>'

// var foo = Foo<Object>();                    // fails analysis: Object isn't SomeBaseClass
```

## Reified generics

Dart generics are **reified**: type arguments are carried at runtime, unlike Java's type erasure.

```dart
var names = <String>[];
names.addAll(['Seth', 'Kathy', 'Lars']);
print(names is List<String>);   // true: the element type survives at runtime
```

<!-- nav -->
---

← [Dart: Inheritance, Mixins & Enums](08-inheritance-mixins.md) · [Index](README.md) · [Dart: Error Handling](10-error-handling.md) →
<!-- nav -->
