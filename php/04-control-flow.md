# PHP — Control Flow

*Source: https://www.php.net/manual/en/control-structures.intro.php*

## if / elseif / else

```php
if ($score >= 90) {
    echo "A";
} elseif ($score >= 70) {   // `elseif` one word (or `else if` two words)
    echo "B";
} else {
    echo "C";
}

if ($ok) echo "yes";        // braces optional for single statement
```

## switch

Compares with **loose `==`**. Cases **fall through** to the next unless `break`.

```php
switch ($day) {
    case 1:
    case 7:                  // shared body — fall-through is intentional here
        echo "Weekend";
        break;
    case 3:
        echo "Midweek";
        break;
    default:                 // optional, matches when nothing else does
        echo "Other";
}
// GOTCHA: forgetting break runs the NEXT case too.
// GOTCHA: loose == — switch("0") matches case 0.
```

## match

**Strict `===`** comparison, **returns a value**, **no fall-through**, throws `UnhandledMatchError` if nothing matches (no `default`).

```php
$msg = match ($status) {
    200, 201 => "Success",       // multiple conditions per arm (comma)
    404      => "Not Found",
    default  => "Unknown",
};                                // note the trailing semicolon — it's an expression

// strict comparison — no type juggling:
$x = match ("1") {
    1   => "int",
    "1" => "string",   // ← this matches ("1" === "1")
};

match (5) { 1 => 'a' };  // UnhandledMatchError — no arm matched, no default
```

**`match(true)`** for ranges / arbitrary conditions:

```php
$group = match (true) {
    $age < 13 => "Child",
    $age < 20 => "Teen",
    default   => "Adult",
};   // first arm whose expression === true wins (top-to-bottom)
```

## while

```php
$i = 1;
while ($i <= 3) {
    echo $i;   // 123
    $i++;
}
```

## do-while

Body runs **at least once** — condition checked after.

```php
$i = 10;
do {
    echo $i;   // 10 — printed even though condition is false
} while ($i < 5);
```

## for

```php
for ($i = 0; $i < 3; $i++) {
    echo $i;   // 012
}

for ($i = 0, $j = 10; $i < $j; $i++, $j--) { ... } // multiple expressions
```

## foreach

The primary way to iterate arrays and `Traversable` objects.

```php
foreach ($arr as $value) {            // values only
    echo $value;
}

foreach ($prices as $fruit => $price) {   // key => value
    echo "$fruit: $price";
}
```

**By reference** with `&` to modify the array in place:

```php
foreach ($nums as &$value) {
    $value *= 2;        // mutates the original array
}
unset($value);          // GOTCHA: $value still references the last element!
                        // forgetting unset() corrupts the array on the next reuse.
```

## break / continue with Levels

```php
foreach ($rows as $row) {
    foreach ($row as $cell) {
        if ($cell === null) continue 2; // skip to next ITERATION of outer loop
        if ($cell === 'STOP') break 2;  // break out of BOTH loops
        echo $cell;
    }
}
// continue;  → next iteration of current loop (level 1, default)
// break;     → exit current loop
```

## Alternative Syntax (Templating)

Replace `{` with `:` and `}` with `endif;` / `endforeach;` / `endwhile;` / `endfor;` / `endswitch;`. Cleaner when interleaving with HTML.

```php
<?php if (count($items) > 0): ?>
    <ul>
    <?php foreach ($items as $item): ?>
        <li><?= $item ?></li>
    <?php endforeach; ?>
    </ul>
<?php else: ?>
    <p>No items.</p>
<?php endif; ?>
```

## list() / Array Destructuring

Unpack arrays into variables.

```php
[$a, $b] = [1, 2];          // $a=1, $b=2  (short syntax)
list($a, $b) = [1, 2];      // older equivalent

[, , $third] = [1, 2, 3];   // skip elements → $third=3

['x' => $x, 'y' => $y] = ['x' => 10, 'y' => 20]; // by key

[[$a, $b], [$c, $d]] = [[1, 2], [3, 4]];          // nested

[$a, $b] = [$b, $a];        // swap without a temp

foreach ([[1, 'a'], [2, 'b']] as [$id, $label]) { // destructure in loop
    echo "$id=$label";
}
```

<!-- nav -->
---

← [PHP — Operators](03-operators.md) · [Index](README.md) · [PHP — Functions](05-functions.md) →
<!-- nav -->
