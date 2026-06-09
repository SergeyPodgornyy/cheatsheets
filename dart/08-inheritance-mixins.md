# Dart — Inheritance, Mixins & Enums

*Source: https://dart.dev/language/extend*

## extends + super

Subclass with `extends`; call the overridden member with `super`.

```dart
class SmartTelevision extends Television {
  void turnOn() {
    super.turnOn();          // call superclass implementation
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
}
```

## @override

Mark overrides with `@override`. Rules for a valid override:

- Return type must be the **same or a subtype** (narrower).
- Each parameter type must be the **same or a supertype** (wider).
- Same number of positional parameters.

```dart
class SmartTelevision extends Television {
  @override
  set contrast(num value) { /* ... */ }
}
```

### covariant

Use `covariant` to **narrow** a parameter type in an override (you take responsibility for the type soundness).

```dart
class Cat extends Animal {
  @override
  void chase(covariant Mouse x) { /* ... */ }   // narrower than Animal.chase(Animal)
}
```

## Abstract classes

Can't be instantiated; can be extended or implemented. Often contain abstract methods.

```dart
abstract class Vehicle {
  void moveForward(int meters);   // abstract method
}
// Vehicle v = Vehicle();   // ERROR — can't construct an abstract class

class Car extends Vehicle {
  @override
  void moveForward(int meters) { /* ... */ }
}
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) { /* ... */ }
}
```

## Implicit interfaces + implements

**Every class implicitly defines an interface** containing all its members. `implements` provides that interface *without* inheriting implementation — the implementing class must define **all** members.

```dart
class Point {
  final x, y;
  Point(this.x, this.y);
}
class MockPoint implements Point {   // must implement x, y, and every member
  @override
  final x = 0;
  @override
  final y = 0;
}
```

Implement multiple interfaces with a comma:

```dart
class C implements A, B { /* must define all members of A and B */ }
```

## noSuchMethod

Override `noSuchMethod` to react to missing members; otherwise a `NoSuchMethodError` is thrown.

```dart
class A {
  @override
  void noSuchMethod(Invocation invocation) {
    print('You tried to use a non-existent member: '
        '${invocation.memberName}');
  }
}
```

## Mixins

Reuse code across multiple class hierarchies. Declare with `mixin`, apply with `with` (comma-separated for multiple). Mixins **can't have `extends`** and **can't declare generative constructors**.

```dart
class Musician extends Performer with Musical { }

class Maestro extends Person with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}

mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

### Abstract members in a mixin

The using class must define them.

```dart
mixin Musician {
  void playInstrument(String instrumentName);   // abstract
  void playPiano() { playInstrument('Piano'); }
  void playFlute() { playInstrument('Flute'); }
}
class Virtuoso with Musician {
  @override
  void playInstrument(String instrumentName) {
    print('Plays the $instrumentName beautifully');
  }
}
```

### on clause

Restricts which types can use the mixin and sets the `super` type — use only when the mixin needs a `super` call.

```dart
class Musician {
  musicianMethod() { print('Playing music!'); }
}
mixin MusicalPerformer on Musician {
  performerMethod() {
    print('Performing music!');
    super.musicianMethod();   // available because of `on Musician`
  }
}
class SingerDancer extends Musician with MusicalPerformer { }
```

### mixin class

Usable both as a class and as a mixin.

```dart
mixin class Musician { }
class Novice with Musician { }      // used as a mixin
class Novice extends Musician { }   // used as a class
```

## Extension methods

Add functionality to existing types without modifying them. Resolved against the **static type** of the receiver — so they fail on `dynamic`.

```dart
extension NumberParsing on String {
  int parseInt() => int.parse(this);
  double parseDouble() => double.parse(this);
}

print('42'.parseInt());            // 42 — after importing the extension
dynamic d = '2';
print(d.parseInt());               // Runtime NoSuchMethodError — dynamic receiver
```

Unnamed extension (library-private):

```dart
extension on String {
  bool get isBlank => trim().isEmpty;
}
```

Generic extension:

```dart
extension MyFancyList<T> on List<T> {
  int get doubleLength => length * 2;
  List<T> operator -() => reversed.toList();
  List<List<T>> split(int at) => [sublist(0, at), sublist(at)];
}
```

**Conflict resolution:** `hide` on import, explicit application `NumberParsing('42').parseInt()`, or an import prefix.

## Extension types

A **zero-cost wrapper**: a compile-time-only interface over an existing **representation type**. No wrapper object is allocated — it's erased at runtime. Use to give a type a disciplined interface without the cost of a real wrapper class.

```dart
extension type IdNumber(int id) {        // representation type int, accessed via `id`
  operator <(IdNumber other) => id < other.id;   // expose only the ops that make sense
  // no `+` declared → addition is a compile error for IdNumber
}

var safeId = IdNumber(42424242);
safeId < IdNumber(42424241);   // OK — wrapped < operator
// safeId + 10;                // ERROR — no + operator
// int x = safeId;             // ERROR — not assignable to int
int x = safeId as int;         // OK — runtime cast to the representation type
```

By default the representation type's members are **not** exposed — declare each one you want.

**Opaque** (no `implements`) = a brand-new type, hides the representation. **Transparent** (`implements`) = also exposes the representation type's members:

```dart
extension type NumberT(int value) implements int {
  // inherits all int members PLUS anything declared here
  NumberT get i => this;
}
int v = NumberT(2);   // OK — transparent, assignable to int
```

**Gotcha — unsafe abstraction:** at runtime the type is the representation type, so `is`/`as` see through it.

```dart
var n = NumberE(1);
if (n is int) print('yes');   // prints — runtime type IS int
```

## Enhanced enums

An enum is a special class that auto-extends `Enum` and is **sealed** (no subclassing, implementing, mixing in, or instantiating).

```dart
enum Color { red, green, blue }

final favoriteColor = Color.blue;
if (favoriteColor == Color.blue) { print('Your favorite color is blue!'); }

assert(Color.red.index == 0);     // .index — declaration order
assert(Color.green.index == 1);

List<Color> colors = Color.values;   // .values — all values in order
assert(colors[2] == Color.blue);

print(Color.blue.name);           // 'blue' — .name is the value's identifier
```

`switch` on an enum (warns if not all values are handled):

```dart
switch (aColor) {
  case Color.red:   print('Red as roses!');
  case Color.green: print('Green as grass!');
  default:          print(aColor);
}
```

Enhanced enums add fields, a const constructor, methods, and can implement interfaces:

```dart
enum Vehicle implements Comparable<Vehicle> {
  car(tires: 4, passengers: 5, carbonPerKilometer: 400),
  bus(tires: 6, passengers: 50, carbonPerKilometer: 800),
  bicycle(tires: 2, passengers: 1, carbonPerKilometer: 0);   // instances first, ≥1

  const Vehicle({
    required this.tires,
    required this.passengers,
    required this.carbonPerKilometer,
  });

  final int tires;
  final int passengers;
  final int carbonPerKilometer;

  int get carbonFootprint => (carbonPerKilometer / passengers).round();
  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
print(Vehicle.car.carbonFootprint);
```

**Rules:** instance vars must be `final`; all generative constructors `const`; a factory may only return a known instance; can't override `index`, `hashCode`, or `==`; can't declare a member named `values`; all instances declared first (at least one).

## Class modifiers

Modifiers go before `class`/`mixin`. Set: `abstract`, `base`, `final`, `interface`, `sealed`, `mixin`. Only `base` may precede a `mixin`. Not allowed on enum/typedef/extension/extension type.

### abstract

Can't be constructed (in any library); CAN be extended and implemented. Often has abstract methods.

```dart
abstract class Vehicle { void moveForward(int meters); }
// Vehicle myVehicle = Vehicle();   // Error
class Car extends Vehicle { @override void moveForward(int meters) {/*...*/} }
class MockVehicle implements Vehicle { @override void moveForward(int meters) {/*...*/} }
```

### base

Disallows **implement** outside the library. Can construct and extend. Subtypes must be `base`/`final`/`sealed`.

```dart
base class Vehicle { void moveForward(int meters) {/*...*/} }
base class Car extends Vehicle {/*...*/}
// base class MockVehicle implements Vehicle {...}   // ERROR in another library
```

### interface

Others can **implement** but not **extend**. Can construct. `abstract interface` = pure interface (implement only; abstract members allowed).

```dart
interface class Vehicle { void moveForward(int meters) {/*...*/} }
// class Car extends Vehicle {...}                   // ERROR in another library
class MockVehicle implements Vehicle {/*...*/}
```

### final

Closes the hierarchy: no extend AND no implement outside the library (includes `base` effects).

```dart
final class Vehicle {/*...*/}
// extend or implement in another library → ERROR
```

### sealed

Known, enumerable subtypes for **exhaustive switch**. Implicitly abstract (can't construct; may have a factory). Subclasses are NOT implicitly abstract.

```dart
sealed class Vehicle {}
class Car extends Vehicle {}
class Truck implements Vehicle {}
class Bicycle extends Vehicle {}

// Vehicle myVehicle = Vehicle();   // ERROR — sealed is implicitly abstract
Vehicle myCar = Car();              // OK
```

### Summary table

| Modifier  | Construct | Extend (outside) | Implement (outside) | Mix in |
|-----------|-----------|------------------|---------------------|--------|
| none      | yes | yes | yes | yes (mixin) |
| abstract  | no  | yes | yes | - |
| base      | yes | yes | no  | - |
| interface | yes | no  | yes | - |
| final     | yes | no  | no  | - |
| sealed    | no  | no  | no  | - |

### Combining order & disallowed combos

```text
(abstract)? (base | interface | final | sealed)? (mixin)? class
```

Disallowed: `abstract` + `sealed` (sealed is already abstract); `interface`/`final`/`sealed` + `mixin` (would allow mixing in).

<!-- nav -->
---

← [Dart — Classes](07-classes.md) · [Index](../README.md) · [Dart — Generics](09-generics.md) →
<!-- nav -->
