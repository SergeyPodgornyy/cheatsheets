# PHP: Generators & Iteration

*Source: https://www.php.net/manual/en/language.generators.php*

## Generators

A function containing `yield` is a **generator**: calling it returns a `Generator` object (which implements `Iterator`). Generators are lazy (the body runs only while iterating), forward-only, and memory-efficient, since they hold one state at a time instead of building a whole array.

```php
function xrange($start, $limit, $step = 1)
{
    for ($i = $start; $i <= $limit; $i += $step) {
        yield $i;            // pauses here, resumes on next iteration
    }
}

foreach (xrange(1, 9, 2) as $n) {
    echo "$n ";              // 1 3 5 7 9
}

// range(0, 1000000) ~100 MB in an array; the generator equivalent <1 KB.
```

Assigning a generator does NOT run its body; execution starts on the first iteration:

```php
$g = xrange(1, 9, 2);        // not started yet
foreach ($g as $n) { }       // body runs now (foreach calls ->rewind())
```

### yield forms

```php
yield $value;            // auto-incrementing integer key (0, 1, 2, ...)
yield $key => $value;    // explicit key
yield;                   // yields null: a bare pause point / coroutine
```

### Return value + getReturn()

A generator may `return` a final value; retrieve it with `getReturn()` AFTER iteration completes.

```php
function gen()
{
    yield 1;
    return 99;
}

$g = gen();
foreach ($g as $v) { }       // 1
echo $g->getReturn();        // 99
```

### yield from (delegation)

`yield from` delegates to an array, `Traversable`, or another `Generator`. A delegated generator's `return` becomes the value of the `yield from` expression.

```php
function inner()
{
    yield 1;
    yield 2;
    return 3;
}

function outer()
{
    yield 0;
    $ret = yield from inner();   // yields 1, 2 ; $ret === 3
    yield $ret;
}

foreach (outer() as $v) {
    echo "$v ";                  // 0 1 2 3
}
```

### finally: cleanup

`finally` runs even when the loop `break`s early, so it is the place to release resources.

```php
function getLines($file)
{
    $f = fopen($file, 'r');
    try {
        while ($line = fgets($f)) {
            yield $line;
        }
    } finally {
        fclose($f);              // runs on completion AND on early break
    }
}
```

### send(): two-way coroutine

`$g->send($x)` resumes the generator and makes the *paused* `yield` expression evaluate to `$x`. Values flow back INTO the generator, turning it into a coroutine.

| Method | Description |
| --- | --- |
| `current()` | Current yielded value |
| `key()` | Current key |
| `next()` | Resume; advance to next `yield` |
| `valid()` | `false` once the generator finished |
| `rewind()` | Run to first `yield` (only before iteration starts) |
| `send($value)` | Resume, sending `$value` as the result of the current `yield` |
| `getReturn()` | Final `return` value (after completion) |

```php
$g->send($x);                // inside the generator, `$received = yield;` gets $x
```

## Iteration interfaces

### Traversable

Base interface that makes an object usable in `foreach`. Don't implement it directly; implement `Iterator` or `IteratorAggregate` (both extend `Traversable`).

### Iterator

5 methods; `foreach` drives them in a fixed order.

```php
class MyIterator implements Iterator
{
    private int $position = 0;
    private array $items = ['first', 'second', 'last'];

    public function rewind(): void                  // before the loop
    {
        $this->position = 0;
    }

    public function valid(): bool                   // continue?
    {
        return isset($this->items[$this->position]);
    }

    public function current(): mixed                // value
    {
        return $this->items[$this->position];
    }

    public function key(): mixed                    // key (only if `=> $k` used)
    {
        return $this->position;
    }

    public function next(): void                    // advance
    {
        ++$this->position;
    }
}

foreach (new MyIterator() as $k => $v) {
    echo "$k => $v\n";       // 0 => first   1 => second   2 => last
}
// foreach does: rewind(); while (valid()) { key(); current(); body; next(); }
```

### IteratorAggregate

Delegate iteration to another iterator; only one method to implement.

```php
class Collection implements IteratorAggregate
{
    private array $items = ['a', 'b', 'c'];

    public function getIterator(): Iterator
    {
        return new ArrayIterator($this->items);   // could also yield ... (a generator)
    }
}

foreach (new Collection() as $k => $v) {
    echo "$k => $v\n";       // 0 => a   1 => b   2 => c
}
```

### ArrayAccess

Make an object usable with `[]` subscript syntax.

| Method | Triggered by |
| --- | --- |
| `offsetExists($k)` | `isset($obj[$k])`, `empty($obj[$k])` |
| `offsetGet($k)` | `$obj[$k]` (read) |
| `offsetSet($k, $v)` | `$obj[$k] = $v` (`$k` is `null` for `$obj[] = $v`) |
| `offsetUnset($k)` | `unset($obj[$k])` |

```php
class Config implements ArrayAccess
{
    private array $data = [];

    public function offsetExists(mixed $k): bool
    {
        return isset($this->data[$k]);
    }

    public function offsetGet(mixed $k): mixed
    {
        return $this->data[$k] ?? null;
    }

    public function offsetSet(mixed $k, mixed $v): void
    {
        if ($k === null) {
            $this->data[] = $v;          // $c[] = ... appends
        } else {
            $this->data[$k] = $v;
        }
    }

    public function offsetUnset(mixed $k): void
    {
        unset($this->data[$k]);
    }
}

$c = new Config();
$c['x'] = 1;          // offsetSet
echo $c['x'];         // offsetGet -> 1
isset($c['x']);       // offsetExists -> true
unset($c['x']);       // offsetUnset
```

### Countable

Make `count($obj)` work.

```php
class Bag implements Countable
{
    private array $items = [1, 2, 3];

    public function count(): int
    {
        return count($this->items);
    }
}

count(new Bag());     // 3
```

### Stringable

Guarantees `__toString()`; satisfied automatically just by declaring `__toString`.

```php
class Money implements Stringable
{
    public function __construct(private int $cents) {}

    public function __toString(): string
    {
        return number_format($this->cents / 100, 2);
    }
}

echo new Money(1050); // 10.50
```

## Built-in SPL iterators

`ArrayIterator`, `IteratorIterator`, `LimitIterator`, `FilterIterator`, `CallbackFilterIterator`, `RecursiveIteratorIterator`, `RecursiveDirectoryIterator`, and more.

```php
iterator_to_array($iter);    // drain an iterator/generator into a plain array
```

<!-- nav -->
---

← [PHP: Database (PDO) & Tooling](13-database-tooling.md) · [Index](README.md) · [PHP: SPL (Standard PHP Library)](15-spl.md) →
<!-- nav -->
