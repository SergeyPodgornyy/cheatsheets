# PHP — Strings

*Source: https://www.php.net/manual/en/language.types.string.php*

## Quoting

**Single quotes** = literal (no interpolation). **Double quotes** = interpolation + escapes.

```php
$name = 'Bob';

echo "Hi $name";      // Hi Bob   (interpolation)
echo "Hi {$name}s";   // Hi Bobs  (braces for complex/adjacent text)
echo 'Hi $name';      // Hi $name (single quotes: literal, no interpolation)

// Escapes work ONLY in double quotes / heredoc:
echo "line1\nline2\ttab\"quote\""; // \n \t \" interpreted
echo 'line1\nline2';               // line1\nline2  (literal backslash-n)
```

`{...}` braces disambiguate complex expressions inside double quotes:

```php
$arr = ['k' => 'v'];
echo "val: {$arr['k']}"; // val: v
$obj = new stdClass; $obj->p = 1;
echo "prop: {$obj->p}";  // prop: 1
```

### Heredoc `<<<EOT` — interpolates

Behaves like a double-quoted string. Closing marker must start the line.

```php
$name = 'Bob';
$txt = <<<EOT
Multi-line $name
indented and interpolated
EOT;
// "Multi-line Bob\nindented and interpolated"
```

### Nowdoc `<<<'EOT'` — literal

Quoted marker = no interpolation (like single quotes).

```php
$txt = <<<'EOT'
Literal $name, no interpolation
EOT;
// "Literal $name, no interpolation"
```

## Concatenation

Use `.` (dot) to join, `.=` to append. PHP does **not** use `+` for strings.

```php
$s = "foo" . "bar"; // "foobar"
$s .= "baz";        // "foobarbaz"
echo "n=" . 5;      // "n=5"  (int coerced to string)
```

## Common String Functions

| Function | Description / example |
|---|---|
| `strlen($s)` | byte length — `strlen("abc")` → `3` |
| `mb_strlen($s)` | multibyte (UTF-8) char length — `mb_strlen("café")` → `4` |
| `strtoupper($s)` / `strtolower($s)` | case — `strtoupper("ab")` → `"AB"` |
| `ucfirst($s)` | capitalize first char — `ucfirst("hi bob")` → `"Hi bob"` |
| `ucwords($s)` | capitalize each word — `ucwords("hi bob")` → `"Hi Bob"` |
| `trim($s)` / `ltrim` / `rtrim` | strip whitespace (both / left / right) |
| `str_replace($search,$repl,$s)` | replace all — `str_replace("a","X","aba")` → `"XbX"` |
| `substr($s,$start,$len)` | slice; negative start counts from end |
| `str_contains($h,$needle)` | bool — `str_contains("hello","ell")` → `true` |
| `str_starts_with($h,$n)` / `str_ends_with($h,$n)` | bool prefix / suffix test |
| `strpos($h,$needle)` | int index or `false` — **use `=== false`** (see gotcha) |
| `explode($delim,$s)` / `implode($glue,$arr)` | split / join |
| `sprintf($fmt,...)` / `printf($fmt,...)` | format to string / print (see specifiers) |
| `number_format($n,$dec)` | `number_format(1234.5, 2)` → `"1,234.50"` |
| `str_repeat($s,$n)` | `str_repeat("ab", 3)` → `"ababab"` |
| `str_pad($s,$len,$pad,$type)` | `str_pad("7", 3, "0", STR_PAD_LEFT)` → `"007"` |
| `strrev($s)` | reverse — `strrev("abc")` → `"cba"` |
| `nl2br($s)` | insert `<br>` before newlines |
| `wordwrap($s,$w)` | wrap to width |
| `htmlspecialchars($s)` | escape HTML for output (**XSS** safety) |

### substr — negative offsets

```php
substr("hello", 1, 3);  // "ell"
substr("hello", -2);    // "lo"  (from end)
substr("hello", 0, -1); // "hell" (negative len drops from end)
```

### strpos `=== false` gotcha

A match at index `0` is falsy — always compare with `===`.

```php
$pos = strpos("abc", "a"); // 0
if ($pos === false) { /* not found */ }   // CORRECT
if (!$pos) { /* WRONG — 0 is truthy-false, treats found-at-0 as "not found" */ }
```

### sprintf / printf format specifiers

`sprintf` returns a formatted string; `printf` prints it (and returns the length). Each placeholder has the shape:

```text
%[argnum$][flags][width][.precision]specifier
```

**Type specifiers** (the final letter — picks how the argument is rendered):

| Spec | Renders the argument as | Example |
|---|---|---|
| `%s` | **string** | `sprintf("%s", 42)` → `"42"` |
| `%d` | **signed decimal** integer (truncates floats) | `sprintf("%d", 42.9)` → `"42"` |
| `%u` | **unsigned** decimal integer | `sprintf("%u", -1)` → huge positive |
| `%f` | **float**, 6 decimals by default (locale-aware) | `sprintf("%f", 3.14)` → `"3.140000"` |
| `%F` | float, **non**-locale-aware (always `.` decimal) | `"3.140000"` |
| `%e` / `%E` | **scientific** notation (lower/upper) | `sprintf("%e", 36000)` → `"3.6e+4"` |
| `%g` / `%G` | **shortest** of `%e`/`%f` | picks compact form |
| `%x` / `%X` | **hex** (lower/upper) | `sprintf("%x", 255)` → `"ff"` |
| `%o` | **octal** | `sprintf("%o", 8)` → `"10"` |
| `%b` | **binary** | `sprintf("%b", 5)` → `"101"` |
| `%c` | the **ASCII character** for an int code | `sprintf("%c", 65)` → `"A"` |
| `%%` | a **literal `%`** (consumes no argument) | `sprintf("100%%")` → `"100%"` |

**Flags** (after `%`, before width):

| Flag | Effect | Example |
|---|---|---|
| `-` | **left-justify** within the width (default is right) | `sprintf("[%-5s]", "x")` → `"[x    ]"` |
| `+` | show `+` on **positive** numbers too | `sprintf("%+d", 5)` → `"+5"` |
| `0` | left-pad **numbers** with zeros | `sprintf("%04d", 7)` → `"0007"` |
| `' '`(space) | pad with spaces (the default) | `sprintf("[%5d]", 7)` → `"[    7]"` |
| `'X` | pad with a **custom char** `X` (after a `'`) | `sprintf("%'*5d", 7)` → `"****7"` |

**Width** = minimum total characters. **Precision** `.N`:
- on floats (`f`/`e`) → digits **after** the decimal point;
- on strings (`s`) → **truncate** to N chars.

```php
sprintf("%5d", 7);        // "    7"   width 5, right-justified
sprintf("%-5d|", 7);      // "7    |"  left-justified
sprintf("%05.2f", 3.1);   // "03.10"   width 5 (incl. the '.'), 2 decimals
sprintf("%.2f", 3.14159); // "3.14"    2 decimals, no min width
sprintf("%.3s", "monkey");// "mon"     truncate string to 3 chars
sprintf("%08.2f", -3.1);  // "-0003.10" zero-pad goes after the sign
```

**Argument numbering** `argnum$` — pick/reuse arguments by position (1-based), e.g. for i18n where word order changes:

```php
sprintf('%1$s is %2$d', 'age', 30);            // "age is 30"
sprintf('%2$d-%1$s-%2$d', 'x', 7);             // "7-x-7"  (reuse arg 2)
// in "double quotes" escape the $:  "%1\$s"   — or use single quotes
```

## Multibyte (UTF-8)

Plain string funcs count **bytes**; for UTF-8 text prefer `mb_*`.

```php
strlen("café");        // 5  (é is 2 bytes)
mb_strlen("café");     // 4  (chars)
mb_substr("café", 0, 3); // "caf"
mb_strtoupper("café"); // "CAFÉ"
```

<!-- nav -->
---

← [PHP — Functions](05-functions.md) · [Index](README.md) · [PHP — Arrays](07-arrays.md) →
<!-- nav -->
