# PHP: Arrays

*Source: https://www.php.net/manual/en/book.array.php*

## Arrays are ordered maps

One type does both list (indexed) and dict (associative). `[]` literal; `[]=` appends.

```php
$list = [10, 20, 30];           // indexed: keys 0,1,2
$dict = ['a' => 1, 'b' => 2];   // associative (string keys)
$mix  = [0 => 'x', 'k' => 'y']; // mixed; insertion order preserved

$list[] = 40;        // append → key 3
$dict['c'] = 3;      // add/overwrite key
echo $list[1];       // 20
$list[10];           // Warning: Undefined array key 10
```

### Nested

```php
$users = [
    ['id' => 1, 'name' => 'Al'],
    ['id' => 2, 'name' => 'Bo'],
];
echo $users[1]['name']; // "Bo"
```

## Counting & searching

| Function | Description / example |
|---|---|
| `count($a)` | element count: `count([1,2,3])` → `3` |
| `in_array($v,$a)` | bool; `in_array($v,$a,true)` for strict type check |
| `array_search($v,$a)` | key of first match or `false`: `array_search('b',['a','b'])` → `1` |
| `array_key_exists($k,$a)` | bool: `array_key_exists('x',['x'=>1])` → `true` |

```php
in_array(2, [1, 2, 3]);        // true
in_array("2", [1, 2], true);   // false (strict: string vs int)
array_search('b', ['a','b','c']); // 1 (key, or false)
```

## Transforming

| Function | Description / example |
|---|---|
| `array_map($fn,$a)` | apply to each: `array_map(fn($n)=>$n*2,[1,2,3])` → `[2,4,6]` |
| `array_filter($a,$fn)` | keep where truthy; keys are preserved! |
| `array_reduce($a,$fn,$init)` | fold: `array_reduce([1,2,3],fn($c,$n)=>$c+$n,0)` → `6` |
| `array_flip($a)` | swap keys/values: `['a'=>1]` → `[1=>'a']` |
| `array_unique($a)` | drop duplicate values (keeps first key) |

### array_filter: keys-preserved gotcha

```php
$r = array_filter([1, 2, 3, 4], fn($n) => $n % 2 === 0);
// [1 => 2, 3 => 4], original keys kept, gaps remain!
$r = array_values($r); // [2, 4], reindex to a clean list

array_unique([1, 1, 2, 3, 3]); // [0=>1, 2=>2, 3=>3]
```

## Keys & values

| Function | Description / example |
|---|---|
| `array_keys($a)` | `array_keys(['a'=>1,'b'=>2])` → `['a','b']` |
| `array_values($a)` | values, reindexed: `['a'=>1,'b'=>2]` → `[1,2]` |
| `array_column($a,$col)` | pluck column: `array_column([['id'=>1],['id'=>2]],'id')` → `[1,2]` |
| `array_combine($k,$v)` | zip into map: `array_combine(['a','b'],[1,2])` → `['a'=>1,'b'=>2]` |

## Building

| Function | Description / example |
|---|---|
| `array_merge($a,$b)` | `array_merge([1,2],[3,4])` → `[1,2,3,4]` |
| `[...$a, ...$b]` | spread merge in a literal |
| `array_fill($start,$n,$v)` | `array_fill(0,3,'x')` → `['x','x','x']` |
| `range($start,$end)` | `range(1,5)` → `[1,2,3,4,5]` |

```php
// array_merge: string keys OVERWRITE, integer keys REINDEX:
array_merge(['a'=>1], ['a'=>2, 0=>'x']); // ['a'=>2, 0=>'x']
array_merge([5=>'a'], [9=>'b']);         // [0=>'a', 1=>'b'] (renumbered!)

$all = [...$a, ...$b]; // spread merge (string keys overwrite too)
```

## Stack & queue

| Function | Description |
|---|---|
| `array_push($a, $v)` | append to end (mutates `$a`) |
| `array_pop($a)` | remove + return last |
| `array_shift($a)` | remove + return first (reindexes) |
| `array_unshift($a, $v)` | prepend to front (reindexes) |

```php
$a = [1, 2];
array_push($a, 3);  // $a = [1,2,3]
array_pop($a);      // 3 → $a = [1,2]
array_shift($a);    // 1 → $a = [2]
array_unshift($a, 0); // $a = [0,2]
```

## Slicing

| Function | Description / example |
|---|---|
| `array_slice($a,$off,$len)` | non-destructive: `array_slice([1,2,3,4],1,2)` → `[2,3]` |
| `array_splice($a,$off,$len,$repl)` | modifies original; remove/replace span |

```php
$a = [1, 2, 3, 4];
array_splice($a, 1, 2, ['x']); // returns removed [2,3]; $a = [1,'x',4]
```

## Sorting

Sorts are in-place (mutate the array) and return `bool`.

| Function | Description |
|---|---|
| `sort($a)` | ascending by value, reindexes keys |
| `rsort($a)` | descending by value, reindexes |
| `asort($a)` | by value ascending, keeps keys |
| `arsort($a)` | by value descending, keeps keys |
| `ksort($a)` | by key ascending |
| `krsort($a)` | by key descending |
| `usort($a,$cmp)` | custom comparator, reindexes |
| `uasort($a,$cmp)` | custom comparator, keeps keys |
| `uksort($a,$cmp)` | custom comparator on keys |

```php
$a = [3, 1, 2];
usort($a, fn($x, $y) => $x <=> $y); // [1,2,3] (spaceship comparator)
```

### Sort-name decoder

```text
a  prefix → keeps keys (associative-preserving)   asort/arsort
k         → sorts by KEY                          ksort/krsort
r         → reverse / descending                  rsort/arsort/krsort
u  prefix → user callback comparator              usort/uasort/uksort
(none)    → by value, reindex                     sort
```

## String ↔ array

```php
implode(',', [1, 2, 3]);   // "1,2,3"  (join)
explode(',', "1,2,3");     // ['1','2','3']  (split → strings)
```

## Callback searches

| Function | Description / example |
|---|---|
| `array_find($a,$fn)` | first matching value or `null`: `array_find([1,2,3],fn($n)=>$n>1)` → `2` |
| `array_any($a,$fn)` | true if any match: `array_any([1,2,3],fn($n)=>$n>2)` → `true` |
| `array_all($a,$fn)` | true if all match: `array_all([2,4,6],fn($n)=>$n%2===0)` → `true` |
| `array_find_key($a,$fn)` | key of first match: `array_find_key([1,2,3],fn($n)=>$n>1)` → `1` |

Also handy: `array_sum`, `array_product`, `min`, `max`.

## Destructuring

```php
[$a, $b] = [1, 2];           // $a=1, $b=2
['x' => $x] = ['x' => 5];    // $x=5 (by key)
[, $second] = [10, 20];      // skip first → $second=20

foreach ($users as ['name' => $n]) { /* destructure each row */ }
```

## Spread in a literal

```php
$a = [1, 2];
$b = [3, 4];
$all = [...$a, ...$b]; // [1,2,3,4]
```

## Copy-on-write value-type gotcha

Arrays are **value types**: assignment and passing copy them (lazily, copy-on-write). Use `&` for by-reference.

```php
$a = [1, 2, 3];
$b = $a;          // COPY (not a reference)
$b[] = 4;
echo count($a);   // 3, $a unchanged

function addOne(array $arr): void   // gets a copy
{
    $arr[] = 1;
}
function addRef(array &$arr): void  // by reference
{
    $arr[] = 1;
}
addOne($a); echo count($a); // 3  (untouched)
addRef($a); echo count($a); // 4  (mutated)

$c = &$a;         // explicit reference alias
$c[] = 99;        // $a also gets 99
```

<!-- nav -->
---

← [PHP: Strings](06-strings.md) · [Index](README.md) · [PHP: OOP](08-oop.md) →
<!-- nav -->
