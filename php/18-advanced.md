# PHP: Advanced (Fibers, Streams, Closures, Serialization)

*Source: https://www.php.net/manual/en/langref.php*

## Fibers

A **Fiber** is a full-stack, interruptible function for cooperative concurrency: one runs at a time, in the same thread, and suspension is explicit. It's the low-level primitive async frameworks build on.

```php
$fiber = new Fiber(function (): void {
    echo "started\n";
    $x = Fiber::suspend('step 1'); // pause; sends 'step 1' OUT to caller; returns what resume() passes IN
    echo "resumed with: $x\n";
});

$out = $fiber->start();    // runs until first suspend(); $out === 'step 1'
$fiber->resume('step 2');  // continues; suspend() returns 'step 2', prints "resumed with: step 2"
$fiber->isTerminated();    // true
```

Hand-off: `start()` runs the body until the first `suspend()`, which yields a value back to the caller. `resume($v)` re-enters the fiber, and `$v` becomes the return value of `suspend()` inside it.

| Member | Description |
| --- | --- |
| `new Fiber($callable)` | Create a fiber from a callable |
| `->start(...$args)` | Begin execution; args passed to the callable; returns value from first `suspend()` |
| `Fiber::suspend($v)` | Static, called from INSIDE; pauses, sends `$v` out |
| `->resume($v)` | Resume; `$v` becomes return of `suspend()` |
| `->throw($e)` | Resume by throwing `$e` at the suspension point |
| `->getReturn()` | The callable's return value (after termination) |
| `Fiber::getCurrent()` | Current fiber, or `null` outside any fiber |
| `->isStarted()` | Has `start()` been called |
| `->isSuspended()` | Currently paused at a `suspend()` |
| `->isRunning()` | Currently executing |
| `->isTerminated()` | Finished (returned or threw) |

```php
$f = new Fiber(function (): int {
    Fiber::suspend();
    return 42;
});

$f->start();
$f->resume();
echo $f->getReturn(); // 42
```

## WeakReference & WeakMap

A **weak reference** does NOT prevent its target from being garbage-collected, which avoids memory leaks in caches and registries.

```php
$obj = new stdClass();
$ref = WeakReference::create($obj);
$ref->get();   // the object, or null once $obj is gc'd

unset($obj);
$ref->get();   // null
```

**WeakMap**: keys are objects held weakly; an entry is auto-removed when its key object is gc'd. Perfect for attaching metadata to objects without leaking.

```php
$map  = new WeakMap();
$user = new User();

$map[$user] = ['last_seen' => time()]; // object key -> data
count($map);   // 1

unset($user);  // key object gone...
count($map);   // 0 (entry auto-removed)
```

## Closures (advanced)

A **Closure** is the object type of every anonymous function, arrow fn, and first-class callable.

Rebind `$this` and scope to another object:

```php
$getX = function () {
    return $this->x;
};

$bound = Closure::bind($getX, $obj, MyClass::class); // static: returns a NEW bound closure
echo $bound();

$bound2 = $getX->bindTo($obj, MyClass::class);       // instance method, same effect
```

Call a closure as if it were a method of an object (binds + invokes in one step):

```php
$result = $getX->call($obj); // $this === $obj inside
```

Make a Closure from any callable:

```php
$fn = Closure::fromCallable('strlen'); // same as strlen(...)
$fn = strlen(...);                      // first-class callable syntax (preferred)
```

Closures capture by value via `use ($v)` (auto for `fn()`), by reference via `use (&$v)`.

## I/O stream wrappers

`php://` stream wrappers:

| Wrapper | Description |
| --- | --- |
| `php://stdin` `php://stdout` `php://stderr` | CLI standard streams (prefer the `STDIN`/`STDOUT`/`STDERR` constants) |
| `php://input` | Raw HTTP request body (read-only); use for JSON/XML APIs |
| `php://output` | Write to the output buffer (like `echo`) |
| `php://memory` | Read-write, always in memory |
| `php://temp` | Read-write, in memory until ~2MB then spills to a temp file; `php://temp/maxmemory:N` |

Other wrappers: `file://` (default fs), `data://` (inline), `http://`, `ftp://` (need `allow_url_fopen`).

Read raw POST body (JSON API):

```php
$data = json_decode(file_get_contents('php://input'), true, flags: JSON_THROW_ON_ERROR);
```

Read STDIN in CLI:

```php
$line = fgets(STDIN);
$all  = file_get_contents('php://stdin'); // echo "hi" | php script.php
```

String as a file resource (no disk):

```php
$fp = fopen('php://temp', 'r+');
fwrite($fp, "data");
rewind($fp);
echo stream_get_contents($fp); // data
```

Stream context: supply the HTTP method, headers, and POST body to `file_get_contents`:

```php
$ctx = stream_context_create([
    'http' => [
        'method'  => 'POST',
        'header'  => 'Content-Type: application/json',
        'content' => $json,
    ],
]);
$resp = file_get_contents('https://api.example.com', false, $ctx);
```

## CLI

```console
$ php script.php arg1 arg2   # run
$ php -a                     # interactive shell
$ php -l script.php          # lint (syntax check)
$ php -S localhost:8000      # built-in web server
```

```php
$argv; // array of args: $argv[0] = script name, $argv[1..] = args
$argc; // count

// 'a:' = required value, 'b::' = optional value, 'c' = flag; long opts too
$opts = getopt('a:b::c', ['name:', 'verbose']);

fwrite(STDERR, "error\n"); // write to stderr
exit(1);                   // exit code (0 = success)

readline('Prompt: ');      // read a line (ext-readline)
```

```php
#!/usr/bin/env php
<?php
// shebang makes the script directly executable
```

## Serialization

PHP-native `serialize`/`unserialize` (PHP-specific format):

```php
$s    = serialize(['a' => 1, 'obj' => $obj]); // string
$data = unserialize($s);

// SECURITY: never unserialize untrusted input (object injection). Restrict classes:
$data = unserialize($s, ['allowed_classes' => [Point::class]]); // or false to allow none
```

Control object serialization with magic methods:

```php
class Session
{
    public function __serialize(): array
    {
        return ['id' => $this->id]; // what to store
    }

    public function __unserialize(array $data): void
    {
        $this->id = $data['id']; // how to restore
    }
}
// (older: __sleep() / __wakeup())
```

JSON serialization: customize how an object is encoded by `json_encode`:

```php
class Money implements JsonSerializable
{
    public function __construct(private int $cents) {}

    public function jsonSerialize(): mixed
    {
        return ['amount' => $this->cents / 100];
    }
}

echo json_encode(new Money(1050)); // {"amount":10.5}
```

## Output buffering

Capture output instead of sending it immediately: templating, post-processing, capturing includes.

```php
ob_start();
echo "Hello";
include 'template.php';
$html = ob_get_clean(); // get buffer contents AND discard the buffer
```

| Function | Description |
| --- | --- |
| `ob_start()` | Begin buffering output |
| `ob_get_contents()` | Peek at current buffer without clearing |
| `ob_get_clean()` | Return buffer contents and discard the buffer |
| `ob_end_clean()` | Discard buffer, stop buffering |
| `ob_end_flush()` | Send buffer, stop buffering |
| `ob_get_level()` | Nesting level of active buffers |

<!-- nav -->
---

← [PHP: Dependency Injection & Reflection](17-di-reflection.md) · [Index](README.md) · [Home](../README.md) →
<!-- nav -->
