# PHP: Functions

*Source: https://www.php.net/manual/en/language.functions.php*

## Typed function declaration

Type declarations on params and return are enforced. Modern PHP is heavily typed.

```php
function add(int $a, int $b): int
{
    return $a + $b;
}
echo add(2, 3); // 5

// `void` return = returns nothing; a function with no return yields null:
function log_it(string $msg): void
{
    /* ... */
}
$r = log_it("hi"); // $r === null
```

## Default arguments

Defaults must come after required params. Caller may omit them.

```php
function greet(string $name, string $greeting = "Hello"): string
{
    return "$greeting, $name!";
}
greet("Alice");       // "Hello, Alice!"
greet("Bob", "Hi");   // "Hi, Bob!"
```

## Variadics `...$args`

`...` collects remaining args into an array. Can be typed.

```php
function sum(int ...$numbers): int
{
    return array_sum($numbers); // $numbers is int[]
}
sum(1, 2, 3, 4); // 10
sum();           // 0  (empty array)
```

## Named arguments

Pass by parameter name `name: value`. Order-independent, and lets you skip defaults.

```php
function makeBox(int $width, int $height, string $color = "black"): void
{
    /* ... */
}

makeBox(height: 20, width: 10, color: "red"); // any order
makeBox(width: 10, height: 20);               // skip default $color
```

## Spread / argument unpacking in calls

`...` in a call unpacks an array into arguments. String keys → named args.

```php
function point(int $x, int $y): void
{
    /* ... */
}

$coords = [3, 7];
point(...$coords);              // positional: $x=3, $y=7
point(...['y' => 7, 'x' => 3]); // named keys: matched by name, order-free
```

## By-reference parameters `&$x`

Prefix a param with `&` and the callee modifies the caller's variable directly.

```php
function increment(int &$value): void
{
    $value++;
}
$n = 5;
increment($n);
echo $n; // 6  (original mutated)
```

## Return types: nullable `?` and union `|`

```php
// Nullable: may return the type OR null:
function findUser(int $id): ?string
{
    return $id > 0 ? "User#$id" : null;
}

// Union: one of several types:
function parse(string $s): int|float
{
    return str_contains($s, '.') ? (float)$s : (int)$s;
}
parse("4");    // 4   (int)
parse("4.5");  // 4.5 (float)
```

## Anonymous functions (closures)

Closures don't auto-capture; pull outer vars in with `use`.

```php
// Capture BY VALUE: snapshot at definition time:
$multiplier = 3;
$times = function (int $x) use ($multiplier): int {
    return $x * $multiplier;
};
$times(4); // 12

// Capture BY REFERENCE: shares the variable:
$counter = 0;
$tick = function () use (&$counter): void {
    $counter++;
};
$tick(); $tick();
echo $counter; // 2
```

## Arrow functions `fn`

Single expression, auto-captures the enclosing scope by value, so no `use` is needed.

```php
$factor = 2;
$double = fn($x) => $x * $factor; // $factor captured automatically
$double(10); // 20

// Great for inline callbacks:
array_map(fn($n) => $n * $n, [1, 2, 3]); // [1, 4, 9]
```

## First-class callable syntax `(...)`

`func(...)` makes a `Closure` from any callable, which is cleaner than string names.

```php
$strlen = strlen(...);
$strlen("hello"); // 5

$hi  = $obj->hi(...);      // instance method
$bye = Greeter::bye(...);  // static method

array_map(strlen(...), ['a', 'bb', 'ccc']); // [1, 2, 3]
```

## Variable functions

A string/variable holding a callable name can be invoked directly.

```php
$fn = 'strlen';
$fn('hi'); // 2

$name = 'strtoupper';
$name('abc'); // "ABC"
```

<!-- nav -->
---

← [PHP: Control Flow](04-control-flow.md) · [Index](README.md) · [PHP: Strings](06-strings.md) →
<!-- nav -->
