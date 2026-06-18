# PHP — Basics

*Source: https://www.php.net/manual/en/langref.php*

**PHP** is a **server-side** scripting language: **dynamically typed** (type inferred at runtime), **loosely typed** (auto-coercion between types), and **interpreted**. Code lives between `<?php ... ?>` tags; statements end with `;`.

## PHP Tags & Output

A pure-PHP file opens with `<?php` and **omits the closing `?>`** (avoids accidental whitespace in output).

```php
<?php
echo "Hello, World!\n";   // echo — no return value, accepts multiple args
echo "a", "b", "c";       // abc — comma-separated

print "Hi";               // print — returns 1, single arg only (usable in expressions)
$ok = print "x";          // prints x, $ok === 1

printf("%d items", 5);    // 5 items — formatted (returns length printed)
$s = sprintf("%05.2f", 3.1); // "03.10" — returns string, no output
```

**Short echo** in templates — interleave with HTML:

```php
<p>Hello <?= $name ?>, you have <?= $count ?> messages</p>
// <?= $x ?> is shorthand for <?php echo $x; ?>
```

## Comments

```php
// single-line (C++ style)
# single-line (shell style)
/* block
   comment */
echo 1; // inline
```

## Variables

All variables start with `$` and are **case-sensitive** (`$name` != `$Name`). **Loosely typed** — type inferred from value; reassignment can change type.

```php
$name  = 'Bob';     // string
$age   = 42;        // int
$age   = 'forty';   // OK — type can change at runtime
$ready = true;      // bool

var_dump($name); // string(3) "Bob"
```

**References** — `&` makes two names point at the same value:

```php
$a = 1;
$b = &$a;   // $b references $a
$b = 99;
echo $a;    // 99 — changed through the reference
```

**Variable variables** — use a variable's value as another variable's name:

```php
$key = 'color';
$$key = 'red';   // creates $color
echo $color;     // red
echo ${$key};    // red — explicit form
```

## Constants

No `$` prefix. Conventionally UPPERCASE. Cannot be reassigned.

```php
const MAX = 100;            // compile-time, namespaced, must be top-level
define('MIN', 0);           // runtime, can be conditional/computed

echo MAX;                   // 100
echo defined('MIN') ? MIN : '?'; // 0

const COLORS = ['r', 'g', 'b']; // arrays allowed
```

`const` is resolved at compile time and respects namespaces; `define()` runs at execution and can sit inside `if`/loops.

## Magic Constants

Compile-time constants that change depending on where they are used.

| Constant | Value |
|---|---|
| `__LINE__` | current line number |
| `__FILE__` | full path of the file |
| `__DIR__` | directory of the file (no trailing slash) |
| `__FUNCTION__` | current function name |
| `__CLASS__` | current class name |
| `__METHOD__` | `Class::method` |
| `__NAMESPACE__` | current namespace name |

```php
echo __LINE__;       // e.g. 12
require __DIR__ . '/config.php';  // path relative to this file
echo __NAMESPACE__;  // "" in global namespace
```

**Predefined** constants (not magic, but always available):

```php
echo PHP_EOL;        // platform line ending ("\n" on Unix)
echo PHP_INT_MAX;    // 9223372036854775807 (64-bit)
echo PHP_INT_SIZE;   // 8
echo PHP_FLOAT_EPSILON; // smallest float diff
```

## Inspecting Values

```php
var_dump($x);        // type + value, recursive: int(42), string(3) "abc"
var_dump([1, 'a']);  // array(2) { [0]=> int(1) [1]=> string(1) "a" }

print_r($arr);       // human-readable, no types
$str = print_r($arr, true); // return as string instead of printing

echo gettype($x);    // "integer", "string", "double" (= float), "boolean", "array", "NULL"

var_export($arr);    // valid PHP code representation
```

<!-- nav -->
---

← [Home](../README.md) · [Index](README.md) · [PHP — Types](02-types.md) →
<!-- nav -->
