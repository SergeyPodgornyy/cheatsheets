# PHP — SPL (Standard PHP Library)

*Source: https://www.php.net/manual/en/book.spl.php*

The **SPL** ships built-in data structures, iterators, and interfaces — concrete classes for stacks, queues, heaps, fixed arrays, and object maps.

## Data structures

### SplDoublyLinkedList

O(1) insert/remove at both ends; base for stack & queue.

```php
$dll = new SplDoublyLinkedList();
$dll->push('a');
$dll->push('b');     // add to end
$dll->unshift('z');  // add to front
echo $dll->top();    // 'b' (last element)
echo $dll->bottom(); // 'z' (first element)
$dll->pop();         // remove from end
$dll->shift();       // remove from front
```

### SplStack (LIFO)

Last-in, first-out.

```php
$s = new SplStack();
$s->push(1);
$s->push(2);
$s->push(3);
echo $s->top();  // 3 (peek, no remove)
echo $s->pop();  // 3 (remove top)
echo count($s);  // 2
```

### SplQueue (FIFO)

First-in, first-out.

```php
$q = new SplQueue();
$q->enqueue('first');
$q->enqueue('second');
echo $q->dequeue(); // 'first'
```

### SplHeap (abstract — implement compare())

Abstract; subclass and define `compare()` to set ordering. Top is the "largest" per `compare()`.

```php
class MyHeap extends SplHeap
{
    // return >0 if $a outranks $b; <=> gives max-heap order
    protected function compare($a, $b): int
    {
        return $a <=> $b;
    }
}

$h = new MyHeap();
$h->insert(3);
$h->insert(1);
$h->insert(5);
echo $h->top();     // 5
echo $h->extract(); // 5 (remove top)
```

### SplMaxHeap / SplMinHeap

Concrete heaps — no `compare()` needed.

```php
$max = new SplMaxHeap();
$max->insert(2);
$max->insert(9);
echo $max->extract(); // 9 (largest first)

$min = new SplMinHeap();
$min->insert(2);
$min->insert(9);
echo $min->extract(); // 2 (smallest first)
```

### SplPriorityQueue

Extract highest priority first; priority passed alongside the value.

```php
$pq = new SplPriorityQueue();
$pq->insert('low', 1);
$pq->insert('high', 10);
$pq->insert('mid', 5);
echo $pq->extract(); // 'high' (priority 10)
echo $pq->extract(); // 'mid'  (priority 5)
```

### SplFixedArray

Fixed-size, integer-indexed; faster and leaner than a native array.

```php
$a = new SplFixedArray(3);
$a[0] = 'a';
$a[1] = 'b';
echo $a[1];          // 'b'
$a->setSize(5);      // resize
echo $a->getSize();  // 5
$native = $a->toArray(); // back to native array
```

### ArrayObject

Wrap a native array as a traversable object.

```php
$ao = new ArrayObject(['x' => 1, 'y' => 2]);
$ao['z'] = 3;   // array access
$ao->append(4); // push
foreach ($ao as $k => $v) {
    // iterate like an array
}
echo count($ao); // 4
```

### SplObjectStorage (object→data map / set)

Maps objects to data, or acts as a set of objects.

```php
$st = new SplObjectStorage();
$o1 = new stdClass();
$o2 = new stdClass();
$st->attach($o1, 'data for o1'); // attach with associated data
$st->attach($o2);                // attach as a plain set entry
var_dump($st->contains($o1));    // true
echo $st[$o1];                   // 'data for o1'
$st->detach($o2);
echo count($st);                 // 1
```

### Quick reference

| Structure | Purpose | Key methods |
| --- | --- | --- |
| `SplDoublyLinkedList` | bidirectional list | `push`/`pop`, `shift`/`unshift`, `top`/`bottom` |
| `SplStack` | LIFO | `push`, `pop`, `top` |
| `SplQueue` | FIFO | `enqueue`, `dequeue` |
| `SplHeap` (abstract) | custom-ordered heap | `insert`, `extract`, `top`, `compare()` |
| `SplMaxHeap` / `SplMinHeap` | max / min on top | `insert`, `extract` |
| `SplPriorityQueue` | priority-ordered | `insert($v, $prio)`, `extract` |
| `SplFixedArray` | fixed int-indexed array | `[]`, `setSize`, `toArray` |
| `ArrayObject` | array-as-object | `append`, `[]`, iterate |
| `SplObjectStorage` | object→data / object set | `attach`, `detach`, `contains` |

## SPL iterators

Wrap and transform other iterables.

```php
// ArrayIterator — iterate an array as an object
$it = new ArrayIterator(['a', 'b', 'c']);
foreach ($it as $v) {
    // 'a', 'b', 'c'
}

// IteratorIterator — wrap any Traversable as a concrete Iterator
$it = new IteratorIterator($someTraversable);

// LimitIterator — slice: offset 0, take 10
$page = new LimitIterator($it, 0, 10);

// CallbackFilterIterator — filter via callback (FilterIterator is abstract)
$even = new CallbackFilterIterator($it, fn($v) => $v % 2 === 0);

// RecursiveArrayIterator + RecursiveIteratorIterator — flatten nested arrays
$flat = new RecursiveIteratorIterator(new RecursiveArrayIterator($nestedArray));
foreach ($flat as $leaf) {
    // each leaf value, depth-first
}

// RecursiveDirectoryIterator — walk a directory tree
$files = new RecursiveIteratorIterator(new RecursiveDirectoryIterator('/path'));
foreach ($files as $file) {
    // each SplFileInfo
}

// GlobIterator — directory listing by glob pattern
$logs = new GlobIterator('/var/log/*.log');
```

Consume iterators as arrays/counts:

```php
$arr   = iterator_to_array($it); // pull all values into a native array
$n     = iterator_count($it);    // count elements
iterator_apply($it, fn() => true); // call fn for each element
```

## SPL exceptions

Semantic exception types — extend these in your own code. Two trees: **LogicException** (bug at dev time) vs **RuntimeException** (error at run time).

```text
LogicException                  // programmer mistake — fix the code
  ├── BadFunctionCallException
  ├── BadMethodCallException
  ├── DomainException
  ├── InvalidArgumentException
  ├── LengthException
  └── OutOfRangeException

RuntimeException                // condition arising only while running
  ├── OutOfBoundsException
  ├── OverflowException
  ├── RangeException
  ├── UnderflowException
  └── UnexpectedValueException
```

Convention: `LogicException` = programmer mistake (fix code); `RuntimeException` = runtime condition.

```php
throw new InvalidArgumentException('age must be >= 0');
```

<!-- nav -->
---

← [PHP — Generators & Iteration](14-generators-iteration.md) · [Index](README.md) · [PHP — Security](16-security.md) →
<!-- nav -->
