# Dart — Classes

*Source: https://dart.dev/language/classes*

All classes except `Null` descend from **`Object`**; every class except `Object?` has exactly one superclass (**mixin-based inheritance**).

## Class declaration & instance variables

Non-nullable instance vars must be initialized at declaration; nullable ones default to `null`.

```dart
class Point {
  double? x;     // initially null
  double? y;     // initially null
  double z = 0;  // non-nullable → must init at declaration
}
```

Every instance variable generates an **implicit getter**. Non-`final` vars (and `late final` without an initializer) also generate an **implicit setter**.

```dart
var point = Point();
point.x = 4;            // uses the implicit setter
assert(point.x == 4);   // uses the implicit getter
assert(point.y == null);
```

**Gotcha — `this` in initializers:** a non-`late` initializer can't access `this`; a `late` one can.

```dart
class Point {
  double? x = initialX;       // OK — no this
  // double? y = this.x;      // ERROR — non-late initializer can't use this
  late double? z = this.x;    // OK — late initializer can access this
  Point(this.x, this.y);
}
```

`final` instance variables are set once — at declaration, via a constructor param, or in an initializer list.

```dart
class ProfileMark {
  final String name;
  final DateTime start = DateTime.now();
  ProfileMark(this.name);
  ProfileMark.unnamed() : name = '';
}
```

## Accessing members

Use `.` to access; use `?.` to short-circuit on a null receiver.

```dart
var p = Point(2, 2);
assert(p.y == 2);
double distance = p.distanceTo(Point(4, 4));
var a = p?.y;   // null-safe: if p is null, a is null (no exception)
```

`a.runtimeType` gives the object's type — prefer `is` over `runtimeType` in production.

## Constructors

### Default constructor

Generative, no args, no name — provided automatically if you declare none. Subclasses do **not** inherit the superclass's named constructors.

### Generative + initializing formals

`this.x` assigns the argument to the instance variable before the body runs.

```dart
class Point {
  double x;
  double y;
  Point(this.x, this.y);   // initializing formals — no body needed
}
```

### Named constructors

```dart
class Point {
  final double x;
  final double y;
  Point(this.x, this.y);
  Point.origin() : x = xOrigin, y = yOrigin;   // Point.origin()
}
```

### Const constructors

For compile-time-constant objects; **all** instance vars must be `final`.

```dart
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);
  final double x, y;
  const ImmutablePoint(this.x, this.y);
}
```

`const` constructors don't always create constants — used outside a const context they create a regular instance.

### Redirecting constructors

Body is empty; delegates to another constructor via `this(...)`.

```dart
class Point {
  double x, y;
  Point(this.x, this.y);
  Point.alongXAxis(double x) : this(x, 0);   // redirects to Point(x, 0)
}
```

### Factory constructors

Use `factory` to return a cached instance or a subtype — can't access `this`. Common for **caching / singletons**.

```dart
class Logger {
  final String name;
  bool mute = false;
  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }
  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }
  Logger._internal(this.name);   // private generative constructor

  void log(String msg) { if (!mute) print(msg); }
}

var logger = Logger('UI');       // returns cached instance for 'UI'
logger.log('Button clicked');
```

### Optional & named params with defaults

```dart
class PointB {
  final double x;
  final double y;
  PointB(this.x, this.y);
  PointB.optional([this.x = 0.0, this.y = 0.0]);   // optional positional
}

class PointC {
  double x;
  double y;
  PointC.named({this.x = 1.0, this.y = 1.0});      // named with defaults
}
final pointC = PointC.named(x: 2.0, y: 2.0);
```

### Initializer lists

Run before the body; RHS can't access `this`. Use for `assert`, computed `final` fields, and setting `final` vars.

```dart
// Set finals from JSON, then run a body:
Point.fromJson(Map<String, double> json)
    : x = json['x']!,
      y = json['y']! {
  print('In Point.fromJson(): ($x, $y)');
}

// Assert in an initializer list:
Point.withAssert(this.x, this.y) : assert(x >= 0) { /* ... */ }
```

```dart
class Point {
  final double x;
  final double y;
  final double distanceFromOrigin;
  // Compute a final field in the initializer list:
  Point(double x, double y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
}
```

### Super calls

Call a non-default superclass constructor with `super.named(...)` in the initializer list.

```dart
class Person {
  String? firstName;
  Person.fromJson(Map data) { print('in Person'); }
}
class Employee extends Person {
  Employee.fromJson(Map data) : super.fromJson(data) { print('in Employee'); }
}
// Prints: in Person  then  in Employee
```

### Super parameters

Forward params straight to `super` (can't combine with redirecting constructors).

```dart
class Vector2d {
  final double x;
  final double y;
  Vector2d(this.x, this.y);
}
class Vector3d extends Vector2d {
  final double z;
  Vector3d(super.x, super.y, this.z);   // forwards x, y to super
}

// Named super params:
class Vector3dNamed extends Vector2d {
  final double z;
  Vector3dNamed.yzPlane({required super.y, required this.z}) : super.named(x: 0);
}
```

### Constructor tear-offs

Reference a constructor without calling it via `.new`.

```dart
var strings = charCodes.map(String.fromCharCode);   // named constructor tear-off
var buffers = charCodes.map(StringBuffer.new);       // unnamed constructor tear-off
```

### Execution order

```text
1. initializer list           (RHS can't access this)
2. superclass no-arg constructor
3. main class constructor body
```

Super-constructor args are evaluated before invocation (may be expressions, but can't access `this`).

## Getters & setters

Computed properties via `get` / `set`.

```dart
class Rectangle {
  double left, top, width, height;
  Rectangle(this.left, this.top, this.width, this.height);

  double get right => left + width;
  set right(double value) => left = value - width;
  double get bottom => top + height;
  set bottom(double value) => top = value - height;
}
```

## Static variables & static methods

Class-wide; **static variables are lazily initialized** (only on first use).

```dart
class Queue {
  static const initialCapacity = 16;
}
assert(Queue.initialCapacity == 16);
```

Static methods have no `this`; they can access static vars and are invoked on the class.

```dart
class Point {
  double x, y;
  Point(this.x, this.y);
  static double distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}
var distance = Point.distanceBetween(a, b);   // call on the class
```

## Instance methods

Access instance variables and `this`.

```dart
class Point {
  final double x;
  final double y;
  Point(this.x, this.y);
  double distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }
}
```

## Operator overloading

Overridable operators: `< > <= >= == ~ - + / ~/ * % | ^ & << >>> >> []= []`.

```dart
class Vector {
  final int x, y;
  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  @override
  bool operator ==(Object other) =>
      other is Vector && x == other.x && y == other.y;

  @override
  int get hashCode => Object.hash(x, y);   // override == → also override hashCode
}
```

## Abstract methods

Body replaced by `;` — allowed only in abstract classes and mixins.

```dart
abstract class Doer {
  void doSomething();   // abstract — no body
}
class EffectiveDoer extends Doer {
  void doSomething() { /* provide implementation */ }
}
```

<!-- nav -->
---

← [Dart — Patterns & Destructuring](06-patterns-records.md) · [Index](README.md) · [Dart — Inheritance, Mixins & Enums](08-inheritance-mixins.md) →
<!-- nav -->
