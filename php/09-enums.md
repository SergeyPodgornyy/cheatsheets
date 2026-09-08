# PHP: Enums

*Source: https://www.php.net/manual/en/language.enumerations.php*

## Pure enum

A fixed set of named **cases** with no scalar value attached.

```php
enum Suit
{
    case Hearts;
    case Diamonds;
    case Clubs;
    case Spades;
}

$s = Suit::Hearts;
echo $s->name; // "Hearts"  (read-only)
```

## Backed enum

Each case is **backed** by a `string` or `int` scalar. Declare the type after `:` and give every case a literal value.

```php
enum Suit: string
{
    case Hearts   = 'H';
    case Diamonds = 'D';
    case Clubs    = 'C';
    case Spades   = 'S';
}

enum Priority: int
{
    case Low  = 1;
    case High = 10;
}
```

## `name` & `value` props

Read-only properties on a case.

```php
Suit::Hearts->name;   // 'Hearts'  (all enums)
Suit::Hearts->value;  // 'H'       (backed enums only)
```

## `cases()`

Returns all cases, in declaration order. Available on any enum.

```php
Suit::cases(); // [Suit::Hearts, Suit::Diamonds, Suit::Clubs, Suit::Spades]
```

## `from()` / `tryFrom()`

**Backed enums only**: map a scalar back to a case. `from()` throws on a bad value; `tryFrom()` returns `null`.

```php
Suit::from('H');     // Suit::Hearts
Suit::from('X');     // throws \ValueError
Suit::tryFrom('X');  // null  (no exception)
Suit::tryFrom('D');  // Suit::Diamonds
```

## Methods, constants & `match($this)`

Enums may declare methods, constants, and static methods. A case constant can reference another case. `match($this)` is the idiomatic dispatch.

```php
interface HasColor
{
    public function color(): string;
}

enum Suit: string implements HasColor
{
    case Hearts = 'H';
    case Spades = 'S';

    const WILD = self::Spades;          // constant referencing a case

    public function color(): string
    {
        return match($this) {           // dispatch on the current case
            Suit::Hearts => 'Red',
            Suit::Spades => 'Black',
        };
    }

    public static function fromChar(string $c): self
    {
        return self::from($c);
    }
}

echo Suit::Hearts->color();        // "Red"
echo Suit::fromChar('S')->color(); // "Black"
echo Suit::WILD->name;             // "Spades"
```

## Implementing interfaces

An enum can `implements` interfaces (and use traits without properties).

```php
enum Suit: string implements HasColor
{
    case Hearts = 'H';
    public function color(): string
    {
        return 'Red';
    }
}
```

## Facts

- You can't `new` an enum; `new Suit()` is an error.
- Can't extend an enum, and an enum can't extend a class.
- Each case is a singleton object → compare with `===`.
- May use traits (traits must not declare properties).

```php
Suit::Hearts === Suit::Hearts;   // true  (same singleton instance)
Suit::Hearts === Suit::Spades;   // false
// new Suit();                    // Error: cannot instantiate enum
```

<!-- nav -->
---

← [PHP: OOP](08-oop.md) · [Index](README.md) · [PHP: Exceptions & Errors](10-exceptions.md) →
<!-- nav -->
