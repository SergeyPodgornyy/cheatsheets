# PHP: Types

*Source: https://www.php.net/manual/en/language.types.php*

## The type system

Scalar types: `bool`, `int`, `float`, `string`. Special types: `null`, `array`, `object`, `callable`, `iterable`, `resource`. Abstract types, usable only as declarations: `mixed`, `void`, `never`.

```php
$flag  = true;     // bool
$count = 42;        // int
$price = 3.14;      // float
$name  = "PHP";     // string
$none  = null;      // null

var_dump($count);            // int(42)
var_dump(is_null($none));    // bool(true)
var_dump(is_int($count));    // bool(true)
```

### Arrays

Ordered map; keys are `int` or `string`. Acts as list, dict, stack, queue.

```php
$list = [1, 2, 3];            // indexed (keys 0,1,2)
$map  = ['a' => 1, 'b' => 2]; // associative
$list[] = 4;                  // append → [1,2,3,4]
$map['c'] = 3;                // add key

var_dump($list);  // array(4) { [0]=>int(1) ... }
$arr[10];         // Warning: Undefined array key 10 (returns null)
```

### Objects

```php
$obj = new stdClass();
$obj->prop = 'x';
var_dump($obj instanceof stdClass); // bool(true)
```

## Type juggling / coercion

PHP auto-converts types based on context (mainly arithmetic vs string).

```php
$r = "10" + 5;      // int(15): numeric string coerced to number
$c = 10 . "5";      // "105": int coerced to string by `.`
$f = "10.5" + 1;    // float(11.5)
$x = true + 1;      // int(2): true→1, false→0
$n = null + 5;      // int(5): null→0

var_dump("0" == false);  // true: "0", 0, "", null, [] are falsy
var_dump(0 == "abc");    // false: non-numeric string NOT cast to 0
var_dump("1" == "01");   // true: both numeric, compared as numbers
```

**Falsy** values: `false`, `0`, `0.0`, `""`, `"0"`, `[]`, `null`. Everything else is truthy (note: `"0.0"` and `"false"` are truthy).

## Type casts

Explicit conversion with `(type)`.

```php
$n = (int) "123abc";   // 123: parses leading digits
$n = (int) "abc";      // 0
$n = (int) 3.99;       // 3: truncates, no rounding
$s = (string) 3.14;    // "3.14"
$b = (bool) "";        // false
$b = (bool) "0";       // false, but (bool)"0.0" is true
$a = (array) "x";      // ["x"]: scalar wrapped
$o = (object) ['k'=>1];// stdClass with ->k = 1
```

## Type declarations

Enforce types on parameters, return values, and properties.

```php
function add(int $a, int $b): int
{
    return $a + $b;
}

class User
{
    public string $name;           // typed property (must be set before read)
    public ?int $age = null;       // nullable, with default
    public array $tags = [];
}
```

### Nullable: `?type`

Shorthand for `type|null`.

```php
function find(?string $id): ?User
{
    return $id === null ? null : User::load($id);
}
// ?string  ==  string|null
```

### Union: `A|B`

Value may be any one of several types.

```php
function format(int|float $n): string
{
    return number_format($n, 2);
}
function id(): int|string
{
    ...
}
```

### Intersection: `A&B`

Value must satisfy all listed types (object/interface types only).

```php
function process(Countable&Traversable $c): void
{
    echo count($c);
    foreach ($c as $item) { ... }
}
```

### `void` / `never` / `mixed`

```php
function logMsg(string $m): void     // returns nothing; `return;` or no return
{
    echo $m;
}
function fail(string $m): never      // never returns (always throws or exits)
{
    throw new RuntimeException($m);
}
function dump(mixed $v): void         // mixed = any type at all
{
    var_dump($v);
}
```

## Strict vs coercive typing

By default PHP is **coercive**: scalar args are silently coerced to the declared type. Add `declare(strict_types=1)` as the first statement of a file to enforce exact types (only `int`→`float` widening allowed).

```php
declare(strict_types=1);   // must be the very first statement

function square(int $n): int
{
    return $n * $n;
}

square(4);     // 16, OK
square("4");   // TypeError in strict mode
               // → coerced to int(4) → 16 in default (coercive) mode
square(4.0);   // TypeError in strict: float not int (no auto-narrowing)
```

## Numeric strings

Strings that look like numbers behave as numbers in arithmetic / numeric comparison.

```php
var_dump(is_numeric("123"));    // true
var_dump(is_numeric("1.5e3"));  // true (scientific notation)
var_dump(is_numeric("0x1A"));   // false (hex strings not numeric)
var_dump(is_numeric("12abc"));  // false

echo "1.5e3" + 0;   // 1500: treated as float
echo "  42" + 0;    // 42: leading whitespace allowed
```

<!-- nav -->
---

← [PHP: Basics](01-basics.md) · [Index](README.md) · [PHP: Operators](03-operators.md) →
<!-- nav -->
