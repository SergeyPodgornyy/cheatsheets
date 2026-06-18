# PHP — Exceptions & Errors

*Source: https://www.php.net/manual/en/language.exceptions.php*

## try / catch / finally

A `try` block needs at least one `catch` **or** a `finally`. `finally` runs no matter what — normal exit, caught exception, or rethrow.

```php
function inverse($x)
{
    if (!$x) { throw new Exception('Division by zero.'); }
    return 1 / $x;
}

try {
    echo inverse(0);
} catch (Exception $e) {
    echo 'Caught: ', $e->getMessage();   // "Caught: Division by zero."
} finally {
    echo "Always runs";                  // runs regardless...
}
```

**Gotchas:**

```php
// exit()/die() in a catch SKIPS finally:
try { throw new Exception('x'); }
catch (Exception $e) { exit('bye'); }    // finally NOT executed
finally { echo "never"; }

// finally-return WINS — overrides any return in try/catch:
function f(): int
{
    try { return 1; }
    finally { return 2; }   // f() returns 2
}
```

## `throw` as an expression

`throw` is an expression — usable in `??`, `?:`, arrow functions, `or`, etc.

```php
do_something_risky() or throw new Exception('failed');

$name = $input ?? throw new InvalidArgumentException('required');

$fn = fn($v) => $v >= 0 ? $v : throw new ValueError('negative');
```

## Multiple catch types

Catch several unrelated types in one block with `|`.

```php
try {
    // ...
} catch (TypeError | ValueError $e) {
    echo "bad input: " . $e->getMessage();
}
```

## Catch without a variable

Omit the variable when you don't need the exception object.

```php
try {
    // ...
} catch (SpecificException) {
    echo "don't care about details";
}
```

## Custom exceptions

Extend `Exception` (or a more specific base).

```php
class MyException extends Exception
{
}

throw new MyException('something broke');
```

## Throwable hierarchy

```
Throwable (interface)
 ├── Error      (engine errors)
 │     ├── TypeError
 │     ├── ValueError
 │     ├── ArithmeticError └── DivisionByZeroError
 │     └── ...
 └── Exception  (user-land)
       ├── RuntimeException
       │     └── (e.g. PDOException)
       ├── LogicException
       │     └── InvalidArgumentException
       └── ...
```

`catch (Throwable $e)` catches **both** `Error` and `Exception`.

```php
try {
    intdiv(1, 0);          // throws DivisionByZeroError (an Error, not an Exception)
} catch (Throwable $e) {
    echo get_class($e);    // "DivisionByZeroError"
}

// Common types:
// Error branch:     TypeError, ValueError, DivisionByZeroError
// Exception branch: RuntimeException, LogicException, InvalidArgumentException
```

## Methods

| Method | Returns |
|--------|---------|
| `getMessage()` | the message string |
| `getCode()` | the (int) code |
| `getPrevious()` | the chained previous Throwable, or `null` |
| `getFile()` | file where thrown |
| `getLine()` | line where thrown |
| `getTrace()` | stack trace as array |
| `getTraceAsString()` | stack trace as string |

```php
try { throw new Exception('boom', 42); }
catch (Exception $e) {
    echo $e->getMessage();  // "boom"
    echo $e->getCode();     // 42
    echo $e->getLine();     // line number
}
```

## Exception chaining

Pass the original exception as the **3rd constructor arg** (`$previous`) to preserve the cause.

```php
try {
    try {
        throw new Exception('original failure');
    } catch (Exception $e) {
        throw new Exception("insert name again", 0, $e); // 3rd arg = previous
    }
} catch (Exception $e) {
    echo $e->getMessage();                  // "insert name again"
    echo $e->getPrevious()->getMessage();   // "original failure"
}
```

## Global handler

`set_exception_handler()` registers a fallback for **uncaught** exceptions (after which the script terminates).

```php
set_exception_handler(function (Throwable $e) {
    error_log($e->getMessage());
    echo "Fatal: " . $e->getMessage();
});
```

<!-- nav -->
---

← [PHP — Enums](09-enums.md) · [Index](README.md) · [PHP — Namespaces & Attributes](11-namespaces-attributes.md) →
<!-- nav -->
