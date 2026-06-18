# PHP — Operators

*Source: https://www.php.net/manual/en/language.operators.php*

## Arithmetic

| Operator | Meaning | Example |
|---|---|---|
| `+` | addition | `2 + 3 // 5` |
| `-` | subtraction | `5 - 2 // 3` |
| `*` | multiplication | `4 * 3 // 12` |
| `/` | division (float if not even) | `7 / 2 // 3.5` |
| `%` | modulo (int) | `7 % 3 // 1` |
| `**` | exponentiation | `2 ** 3 // 8` |
| `intdiv()` | integer division | `intdiv(7, 2) // 3` |

```php
echo 7 / 2;          // 3.5  (float — / never floor-divides)
echo intdiv(7, 2);   // 3    (integer division)
echo 7 % 3;          // 1
echo -7 % 3;         // -1   (% follows sign of dividend)
echo 2 ** 10;        // 1024
echo fmod(7.5, 2);   // 1.5  (float modulo)
intdiv(1, 0);        // DivisionByZeroError
```

## Increment / Decrement

```php
$a = 5;
echo $a++;   // 5  — post: returns THEN increments ($a now 6)
echo ++$a;   // 7  — pre: increments THEN returns
echo $a--;   // 7  — post-decrement ($a now 6)
echo --$a;   // 5  — pre-decrement

$s = 'Az';
$s++;        // "Ba"  — string increment (Perl-style, letters carry)
```

## Assignment

| Operator | Equivalent |
|---|---|
| `=` | assign |
| `+=` `-=` `*=` `/=` `%=` `**=` | arithmetic compound |
| `.=` | concat-assign |
| `??=` | null-coalescing assign |
| `&=` `\|=` `^=` `<<=` `>>=` | bitwise compound |

```php
$x = 5; $x += 3;     // 8
$s = "Hi"; $s .= "!";// "Hi!"
$config['x'] ??= 'default'; // assign 'default' only if $config['x'] is null/unset
```

`=` returns the assigned value, so assignments chain: `$a = $b = 0;`.

## Comparison

| Operator | Meaning |
|---|---|
| `==` | equal after type juggling (loose) |
| `===` | identical: same value **and** type |
| `!=` / `<>` | not equal (loose) |
| `!==` | not identical |
| `<` `<=` `>` `>=` | ordering |
| `<=>` | spaceship: -1 / 0 / 1 |

**Loose `==` gotchas** — type juggling produces surprising results. **Prefer `===`.**

```php
var_dump(0 == "a");      // false — non-numeric string NOT cast to 0
var_dump("1" == "01");   // true  — both numeric → compared as numbers
var_dump(100 == "1e2");  // true  — "1e2" is numeric (100.0)
var_dump("1" === 1);     // false — different types (string vs int)
var_dump(null == false); // true
var_dump([] == false);   // true  — empty array is falsy
var_dump("abc" == 0);    // false
```

**Spaceship `<=>`** — returns `int` for ordering; ideal for sort comparators.

```php
echo 1 <=> 2;  // -1  (left < right)
echo 2 <=> 2;  //  0  (equal)
echo 3 <=> 2;  //  1  (left > right)

usort($arr, fn($a, $b) => $a->age <=> $b->age); // ascending sort
```

## Logical

| Operator | Meaning | Precedence |
|---|---|---|
| `!` | not | high |
| `&&` | and | high |
| `\|\|` | or | high |
| `and` | and | **below `=`** |
| `or` | or | **below `=`** |
| `xor` | exclusive or | below `=` |

`&&`/`||` **short-circuit** (stop evaluating once result known).

```php
$ok = isset($x) && $x > 0;   // right side skipped if $x not set
$v  = $cache ?: compute();   // compute() only if $cache falsy
```

**Precedence trap** — `and`/`or` bind *looser* than `=`:

```php
$a = $b && $c;   // $a = ($b && $c)   ✓ as expected
$a = $b and $c;  // ($a = $b) and $c  — $a gets $b! the `and` runs after assign
```

Use `&&`/`||` in expressions; reserve `and`/`or` for flow control like `$fh = fopen(...) or die();`.

## String Concatenation

```php
$s = "Hello" . " " . "World";  // "Hello World"  (. is the concat operator)
$s .= "!";                     // "Hello World!"
$n = 5; echo "x" . $n;         // "x5"  — int coerced to string
```

## Ternary & Short Ternary

```php
$label = $age >= 18 ? "adult" : "minor";   // standard ternary

$name = $username ?: "guest";   // short ternary (Elvis): $username ?: ...
// returns $username if truthy, else "guest"  (== $username ? $username : "guest")
// NOTE: triggers a warning if $username is undefined — use ?? for that
```

## Null Coalescing — `??` / `??=`

Returns left operand if it **exists and is not null**, else the right. No warning on undefined.

```php
$user = $_GET['user'] ?? 'nobody';     // no warning if 'user' unset
$x = $a ?? $b ?? 'default';            // chainable — first non-null wins
$config['x'] ??= 'default';            // assign only if null/unset

// ?? vs ?: — ?? only checks null/unset (not falsy):
$n = 0;
echo $n ?? 'd';   // 0   (0 is not null)
echo $n ?: 'd';   // 'd' (0 is falsy)
```

## Nullsafe — `?->`

Short-circuits the whole chain to `null` if any link is `null` — no error.

```php
$country = $session?->user?->getAddress()?->country;
// null if $session, ->user, or ->getAddress() is null — no "method on null" error

// equivalent verbose form:
$country = $session !== null ? $session->user?->... : null;
```

## Bitwise

| Operator | Meaning | Example |
|---|---|---|
| `&` | and | `6 & 3 // 2` |
| `\|` | or | `6 \| 3 // 7` |
| `^` | xor | `6 ^ 3 // 5` |
| `~` | not | `~5 // -6` |
| `<<` | left shift | `1 << 3 // 8` |
| `>>` | right shift | `16 >> 2 // 4` |

```php
echo 6 & 3;   // 2   (110 & 011 = 010)
echo 6 | 3;   // 7   (110 | 011 = 111)
echo 6 ^ 3;   // 5   (110 ^ 011 = 101)
echo 1 << 3;  // 8   (multiply by 2^3)
echo 16 >> 2; // 4   (divide by 2^2)

// flag sets:
const READ = 1, WRITE = 2;
$perms = READ | WRITE;            // 3
$canWrite = (bool)($perms & WRITE); // true
```

## `instanceof`

Tests whether an object is an instance of a class / subclass / interface.

```php
if ($obj instanceof MyClass) { ... }
var_dump($e instanceof Throwable); // true for any exception/error
var_dump("x" instanceof Foo);      // false — non-objects never match
```

## Pipe — `|>`

Passes the left value as the **single argument** to the right callable, evaluated **left-to-right**.

```php
$result = "  Hello World  " |> trim(...) |> strtoupper(...);
// "HELLO WORLD"  — same as strtoupper(trim("  Hello World  "))

$len = "Hello World" |> strlen(...);  // 11

// reads as a forward pipeline instead of nested calls:
$out = $data |> array_filter(...) |> array_values(...) |> count(...);
```

Use parentheses to force evaluation order when mixing operators.

<!-- nav -->
---

← [PHP — Types](02-types.md) · [Index](README.md) · [PHP — Control Flow](04-control-flow.md) →
<!-- nav -->
