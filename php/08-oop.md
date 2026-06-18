# PHP — OOP

*Source: https://www.php.net/manual/en/language.oop5.php*

## Classes & `$this`

A **class** bundles typed properties and methods. `$this` is the current instance inside non-static methods.

```php
class Point
{
    public int $x = 0;     // typed property with default
    public int $y = 0;

    public function move(int $dx, int $dy): void
    {
        $this->x += $dx;   // $this = current instance
        $this->y += $dy;
    }
}

$p = new Point();
$p->move(3, 4);
echo $p->x; // 3
```

## `new` & chaining

```php
$p = new Point();          // parentheses optional when no args: new Point;
$p = new Point()->move(1, 1);  // chain a method off `new` without wrapping parens
```

## Constructor & property promotion

**Property promotion** declares + assigns a property straight from the constructor signature — no separate property line, no `$this->x = $x`.

```php
class PromotedPoint
{
    public function __construct(
        public int $x = 0,        // declared AND assigned
        protected int $y = 0,
        private int $z = 0,
    ) {
    }
}

$p = new PromotedPoint(1, 2, 3);
echo $p->x; // 1
```

### Classic constructor

```php
class A
{
    private int $x;
    public function __construct(int $x = 0)
    {
        $this->x = $x;          // manual assignment
    }
}
```

## Readonly properties

Initialized once, only from within the **declaring scope** (usually the constructor). Any later write is an error.

```php
class User
{
    public function __construct(public readonly string $name)
    {
    }
}

$u = new User("Alice");
echo $u->name; // Alice
// $u->name = "Bob"; // Error: Cannot modify readonly property User::$name
```

## Visibility

| Modifier    | Access scope                          |
|-------------|---------------------------------------|
| `public`    | everywhere                            |
| `protected` | declaring class + subclasses          |
| `private`   | declaring class only                  |

Applies to **both** properties and methods.

```php
class Acct
{
    public int $id;
    protected float $balance;   // visible to subclasses
    private string $pin;        // this class only
}
```

## Asymmetric visibility

Read and write visibility can differ. Write (`set`) may be **narrower** than read; get must not be narrower than set.

```php
class Example
{
    public protected(set) string $name;  // public read, protected write

    public function __construct(string $name)
    {
        $this->name = $name;             // OK — inside the class
    }
}

$e = new Example("x");
echo $e->name;       // OK — public read
// $e->name = "y";   // Error: cannot write (protected set) from outside
```

## Property hooks

Add `get`/`set` logic to a property. A **virtual** hook has no backing store (computed each access); a `set` hook can transform the value or write to its own backing value explicitly.

```php
class Person
{
    public string $fullName {
        get => $this->firstName . ' ' . $this->lastName;  // virtual, computed — no backing field
    }

    public string $firstName {
        set => ucfirst(strtolower($value));   // transform on write; $value = assigned value
    }

    public string $lastName {
        set {
            if (strlen($value) < 2) {
                throw new \InvalidArgumentException('Too short');
            }
            $this->lastName = $value;         // explicit write to backing value
        }
    }
}

$p = new Person();
$p->firstName = 'peter';
echo $p->firstName;          // "Peter"  (transformed on set)
$p->lastName = 'Peterson';
echo $p->fullName;           // "Peter Peterson"  (computed by get)
```

## Static properties & methods

Belong to the class, not instances. Access via `self::` (inside) or `ClassName::` (outside).

```php
class Counter
{
    public static int $count = 0;
    public static function inc(): void
    {
        self::$count++;
    }
}

Counter::inc();
echo Counter::$count; // 1
```

## Class constants

Constants take visibility modifiers. `final const` cannot be overridden by a subclass.

```php
class Config
{
    const VERSION = "1.0";       // implicitly public
    public const MAX = 100;
    final const FIXED = 42;      // subclass cannot redefine
}

echo Config::VERSION; // 1.0
echo Config::MAX;     // 100
```

## Inheritance

`extends` for single inheritance; `parent::` calls the overridden parent method.

```php
class Animal
{
    public function speak(): string
    {
        return "...";
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        return "Woof " . parent::speak();   // calls Animal::speak()
    }
}

echo (new Dog())->speak(); // "Woof ..."
```

## Abstract classes

Cannot be instantiated. **Abstract methods** declare a signature only; subclasses must implement them.

```php
abstract class Shape
{
    abstract public function area(): float;            // no body
    public function describe(): string
    {
        return "Area: " . $this->area();               // calls subclass impl
    }
}

class Circle extends Shape
{
    public function __construct(private float $r)
    {
    }
    public function area(): float
    {
        return 3.14159 * $this->r ** 2;
    }
}

// new Shape(); // Error: Cannot instantiate abstract class Shape
echo (new Circle(2))->describe(); // "Area: 12.56636"
```

## Interfaces

Define a contract of method signatures. A class can `implements` **many** interfaces.

```php
interface Drawable
{
    public function draw(): string;
}
interface Sizable
{
    public function size(): int;
}

class Box implements Drawable, Sizable
{
    public function draw(): string
    {
        return "[]";
    }
    public function size(): int
    {
        return 1;
    }
}
```

## Traits

Horizontal reuse — copy methods into a class via `use`.

```php
trait Greetable
{
    public function greet(): string
    {
        return "Hello from " . static::class;  // late static binding
    }
}

class Service
{
    use Greetable;
}

echo (new Service())->greet(); // "Hello from Service"
```

## Late static binding

`$this` = current instance. `self::` = the **defining** class (resolved early, at compile time). `static::` = the **called** class (resolved late, at runtime).

```php
class Base
{
    public static function create(): static  // returns called class
    {
        return new static();
    }
    public static function who()
    {
        return "Base";
    }
    public static function test()
    {
        echo self::who();    // always "Base"   — defining class
        echo static::who();  // runtime class   — called class
    }
}

class Derived extends Base
{
    public static function who()
    {
        return "Derived";
    }
}

Derived::test(); // prints "Base" then "Derived"
```

## `final`

Prevents extension / overriding.

```php
final class NoExtend  // cannot be extended
{
}

class Locker
{
    final public function lock()  // cannot be overridden
    {
    }
}
```

## Clone & clone-with

`clone` makes a **shallow copy**. `__clone()` customizes the copy (e.g. deep-copy object properties). `clone($obj, [...])` clones then overwrites named properties.

```php
$b = clone $a;                       // shallow copy
$c = clone($a, ['x' => 10]);         // clone, then set x = 10 on the copy

class Node
{
    public function __clone(): void
    {
        // deep-copy object properties here so copy doesn't share references
    }
}
```

## Magic methods

| Method | Triggered when / purpose |
|--------|--------------------------|
| `__construct(...)` | object created with `new` |
| `__destruct()` | object destroyed / refs gone |
| `__get($name)` | reading an inaccessible/undefined property; returns the value |
| `__set($name, $value)` | writing an inaccessible/undefined property; returns `void` |
| `__isset($name)` | `isset()`/`empty()` on inaccessible property |
| `__unset($name)` | `unset()` on inaccessible property |
| `__call($name, $args)` | calling an inaccessible/undefined instance method |
| `__callStatic($name, $args)` | calling an inaccessible/undefined static method |
| `__toString(): string` | object used in string context |
| `__invoke(...$args)` | object called as a function |
| `__clone()` | after `clone` |

```php
class Magic
{
    private array $data = [];

    public function __get($n)
    {
        return $this->data[$n] ?? null;
    }
    public function __set($n, $v): void
    {
        $this->data[$n] = $v;
    }
    public function __call($n, $args)
    {
        return "called $n";
    }
    public function __toString(): string
    {
        return "Magic object";
    }
    public function __invoke(...$a)
    {
        return "invoked";
    }
}

$m = new Magic();
$m->foo = 1;            // __set
echo $m->foo;           // 1   (__get)
echo $m->bar;           // null (__get, missing key)
echo $m->doStuff();     // "called doStuff" (__call)
echo $m;                // "Magic object"  (__toString)
echo $m();              // "invoked"       (__invoke)
```

## Anonymous classes

Define and instantiate in one expression; can take constructor args, implement interfaces, extend classes.

```php
$o = new class(42) {
    public function __construct(public int $val)
    {
    }
};
echo $o->val; // 42
```

## `#[\Override]`

Marks a method (or property) intended to override a parent — the engine errors if there's nothing to override (catches typos / renamed parents).

```php
class Animal
{
    public function speak(): string
    {
        return "...";
    }
}

class Cat extends Animal
{
    #[\Override]
    public function speak(): string  // OK — parent has speak()
    {
        return "Meow";
    }
    // #[\Override] public function speek() {} // Error: no matching parent method
}
```

<!-- nav -->
---

← [PHP — Arrays](07-arrays.md) · [Index](README.md) · [PHP — Enums](09-enums.md) →
<!-- nav -->
