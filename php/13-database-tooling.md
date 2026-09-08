# PHP: Database (PDO) & Tooling

*Source: https://www.php.net/manual/en/book.pdo.php*

## Connecting with PDO

`new PDO($dsn, $user, $pass, $options)`. Always set `ERRMODE_EXCEPTION` so failures throw instead of returning silent `false`.

```php
$options = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,  // throw PDOException on error
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,        // default fetch shape
    PDO::ATTR_EMULATE_PREPARES   => false,                  // use real native prepares
];

try {
    $dbh = new PDO('mysql:host=localhost;dbname=test', $user, $pass, $options);
} catch (PDOException $e) {
    // handle here: an UNCAUGHT PDOException can leak connection details (user/pass) in the trace
    echo 'Connection failed';
}
```

DSN examples:

```php
'mysql:host=localhost;dbname=test'   // MySQL
'sqlite:/path/to/db.sqlite'          // SQLite (file); 'sqlite::memory:' for in-memory
'pgsql:host=localhost;dbname=test'   // PostgreSQL
```

## Prepared statements

Bind data separately from SQL. This is the way to prevent SQL injection: values are never parsed as SQL.

```php
// Named placeholders:
$stmt = $dbh->prepare('SELECT * FROM users WHERE status = :status AND age > :age');
$stmt->execute([':status' => 'active', ':age' => 18]);   // leading ':' optional in array

// Positional placeholders:
$stmt = $dbh->prepare('SELECT * FROM users WHERE status = ? AND age > ?');
$stmt->execute(['active', 18]);                          // order matches the ?'s
```

Never interpolate input into SQL strings: `"... WHERE name = '$name'"` is injectable.

### bindValue vs bindParam

```php
$stmt->bindValue(':name', $name, PDO::PARAM_STR);  // captures VALUE at bind time
$stmt->bindParam(':age', $age, PDO::PARAM_INT);    // binds VARIABLE by reference, read at execute()

$age = 21;
$stmt->execute();   // bindParam sees 21 (latest value); bindValue would have used the old value
// Types: PDO::PARAM_STR | PARAM_INT | PARAM_BOOL | PARAM_NULL | PARAM_LOB
```

## INSERT + lastInsertId

```php
$stmt = $dbh->prepare('INSERT INTO users (name, age) VALUES (:name, :age)');
$stmt->execute([':name' => 'Alice', ':age' => 30]);
echo $dbh->lastInsertId();   // auto-increment id of the row just inserted
```

## Fetching

```php
// Row-by-row (memory-friendly for big results):
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    echo $row['name'];
}

$all = $stmt->fetchAll(PDO::FETCH_ASSOC);   // array of all rows at once

// Single scalar (first column of next row):
$count = $dbh->query('SELECT COUNT(*) FROM users')->fetchColumn();
```

### Fetch modes

| Mode | Each row becomes |
|---|---|
| `PDO::FETCH_ASSOC` | Array keyed by column name |
| `PDO::FETCH_NUM` | Array keyed by 0-based column index |
| `PDO::FETCH_BOTH` | Both assoc + numeric keys (default) |
| `PDO::FETCH_OBJ` | `stdClass`, e.g. `$row->name` |
| `PDO::FETCH_CLASS` | Instance of a named class (cols mapped to properties) |

## query() vs exec()

```php
$stmt = $dbh->query('SELECT * FROM users');          // runs + returns PDOStatement (to fetch)
$n    = $dbh->exec("DELETE FROM logs WHERE old = 1"); // runs + returns affected-row count (int)
```

Use prepared statements (not `query`/`exec`) whenever untrusted values are involved.

## Transactions

All-or-nothing: commit on success, roll back on any failure.

```php
try {
    $dbh->beginTransaction();
    $dbh->exec("UPDATE accounts SET balance = balance - 50 WHERE id = 1");
    $dbh->exec("UPDATE accounts SET balance = balance + 50 WHERE id = 2");
    $dbh->commit();
} catch (PDOException $e) {
    $dbh->rollBack();   // undo everything since beginTransaction()
    throw $e;
}
```

---

## JSON

```php
$json = json_encode(['name' => 'Bob', 'age' => 30]);  // '{"name":"Bob","age":30}'
$json = json_encode($data, JSON_PRETTY_PRINT);         // indented, human-readable

$data = json_decode($json, true);   // true => assoc ARRAY
$obj  = json_decode($json);         // omit => stdClass, so $obj->name

// json_decode returns null on error by default, so check json_last_error(), or force throwing:
$data = json_decode($json, true, flags: JSON_THROW_ON_ERROR);  // throws JsonException on bad JSON
```

## Dates

```php
$now = new DateTime();                       // mutable, "now"
$d   = new DateTimeImmutable('2024-01-15');  // immutable, PREFER this

echo $d->format('Y-m-d H:i:s');              // '2024-01-15 00:00:00'
$d2  = $d->modify('+1 day');                 // immutable: returns a NEW instance ($d unchanged)

$diff = $d->diff(new DateTimeImmutable('2024-02-01'));  // DateInterval
echo $diff->days;                            // 17

// Procedural:
echo date('Y-m-d', time());   // format a timestamp; time() = current Unix timestamp
$ts = strtotime('2024-01-15'); // parse text -> timestamp (int)
```

### `format()` characters

Pass these to `format()` / `date()`. Each letter is a code; unknown chars print as-is; escape a letter with `\` to print it literally.

**Day / week**

| Char | Meaning | Example |
|---|---|---|
| `d` | day of month, 2-digit | `01`-`31` |
| `j` | day of month, no leading zero | `1`-`31` |
| `D` | weekday, short text | `Mon`-`Sun` |
| `l` | weekday, full text | `Sunday`-`Saturday` |
| `N` | ISO weekday number | `1` (Mon) to `7` (Sun) |
| `w` | weekday number | `0` (Sun) to `6` (Sat) |
| `S` | ordinal suffix (pairs with `j`) | `st` `nd` `rd` `th` |
| `z` | day of year | `0`-`365` |
| `W` | ISO week number of year | `42` |

**Month / year**

| Char | Meaning | Example |
|---|---|---|
| `m` | month, 2-digit | `01`-`12` |
| `n` | month, no leading zero | `1`-`12` |
| `M` | month, short text | `Jan`-`Dec` |
| `F` | month, full text | `January`-`December` |
| `t` | days in the month | `28`-`31` |
| `Y` | year, 4-digit | `2024` |
| `y` | year, 2-digit | `24` |
| `L` | is leap year? | `1` / `0` |

**Time**

| Char | Meaning | Example |
|---|---|---|
| `H` | hour 24h, 2-digit | `00`-`23` |
| `G` | hour 24h, no leading zero | `0`-`23` |
| `h` | hour 12h, 2-digit | `01`-`12` |
| `g` | hour 12h, no leading zero | `1`-`12` |
| `i` | minutes, 2-digit | `00`-`59` |
| `s` | seconds, 2-digit | `00`-`59` |
| `a` / `A` | am/pm, lower / upper | `am` / `PM` |
| `u` / `v` | microseconds / milliseconds | `654321` / `654` |

**Timezone / full**

| Char | Meaning | Example |
|---|---|---|
| `e` | timezone identifier | `UTC`, `Europe/Berlin` |
| `T` | timezone abbreviation | `EST`, `CET` |
| `P` | GMT offset, with colon | `+02:00` |
| `O` | GMT offset, no colon | `+0200` |
| `c` | ISO 8601 datetime | `2024-02-12T15:19:21+00:00` |
| `r` | RFC 2822 datetime | `Thu, 21 Dec 2000 16:01:07 +0200` |
| `U` | seconds since Unix epoch | `1705276800` |

```php
$d->format('Y-m-d H:i:s');   // 2024-01-15 13:45:30
$d->format('D, d M Y');      // Mon, 15 Jan 2024
$d->format('l \t\h\e jS');   // Monday the 15th; \t \h \e escape letters to print literally
$d->format('g:i A');         // 1:45 PM
```

### Parsing with `createFromFormat()`

Same letters, but they describe the input. Without a control char, fields you don't parse default to the current date and time, which is usually a bug.

| Char | Effect when parsing |
|---|---|
| `!` | reset all fields to the Unix epoch first; then apply parsed fields (→ clean zeroed time) |
| `\|` | reset fields not yet parsed to the epoch (put at the end) |
| `\` | escape: treat the next char as a literal, not a code |
| `*` | skip input up to the next separator |
| `+` | tolerate trailing data (warning instead of failure) |

```php
// No control char: time defaults to NOW (gotcha!)
DateTimeImmutable::createFromFormat('Y-m-d', '2024-01-01')
    ->format('H:i:s');                                 // current time, e.g. 14:30:25

// '!' at the start zeros the unspecified time fields:
DateTimeImmutable::createFromFormat('!Y-m-d', '2024-01-01')
    ->format('Y-m-d H:i:s');                           // 2024-01-01 00:00:00

// '|' keeps parsed fields, resets the rest to epoch:
DateTimeImmutable::createFromFormat('H:i|', '13:45')
    ->format('Y-m-d H:i:s');                           // 1970-01-01 13:45:00

// '\' escapes the literal 'T' between date and time:
DateTimeImmutable::createFromFormat('!Y-m-d\TH:i:s', '2024-01-01T13:45:30');
```

## Regex (PCRE)

Pattern is wrapped in **delimiters** (`/.../`, or `#...#`, `~...~` when the pattern contains `/`), with optional trailing flags.

```php
preg_match('/(\d+)/', 'abc123', $m);   // returns 1 (matched); $m[0]==='123' (whole), $m[1]==='123' (group 1)
preg_match('/\d+/', 'abc');            // 0 (no match); returns false on error
preg_match_all('/\d+/', 'a1b22', $m);  // 2; $m[0] === ['1','22'] (all matches)
preg_replace('/\s+/', ' ', $text);     // collapse runs of whitespace to a single space
preg_replace_callback('/\d+/', fn($m) => $m[0] * 2, 'a3b'); // "a6b"
preg_split('/,\s*/', 'a, b,c');        // ['a', 'b', 'c']
preg_quote('a.b*c');                   // 'a\.b\*c', escape user input before embedding
```

### Metacharacters

| Token | Matches |
|---|---|
| `.` | any single char (except newline, unless `s` flag) |
| `^` | start of string (or line, with `m` flag) |
| `$` | end of string (or line, with `m`) |
| `\|` | alternation: `cat\|dog` matches either |
| `( )` | capturing group → fills `$m[1]`, `$m[2]`, … |
| `(?: )` | non-capturing group (group, but don't capture) |
| `(?<name> )` | named group → `$m['name']` |
| `[ ]` | character class: any one char inside |
| `[^ ]` | negated class: any char not inside |
| `\` | escape the next metachar (`\.` = literal dot) |

### Quantifiers (how many of the preceding token)

| Token | Meaning |
|---|---|
| `*` | 0 or more |
| `+` | 1 or more |
| `?` | 0 or 1 (optional) |
| `{n}` | exactly `n` |
| `{n,}` | `n` or more |
| `{n,m}` | between `n` and `m` |
| `*?` `+?` `??` `{n,m}?` | lazy (append `?`): match as few as possible; the default is greedy, as many as possible |

```php
preg_match('/a.*c/',  'aXcYc', $m);  // $m[0] === 'aXcYc'  (greedy: to the LAST c)
preg_match('/a.*?c/', 'aXcYc', $m);  // $m[0] === 'aXc'    (lazy: to the FIRST c)
```

### Character-class shorthands

| Token | Matches | Negation |
|---|---|---|
| `\d` | digit `[0-9]` | `\D` non-digit |
| `\w` | word char `[A-Za-z0-9_]` | `\W` non-word |
| `\s` | whitespace (space/tab/newline) | `\S` non-whitespace |
| `\b` | word boundary (zero-width) | `\B` non-boundary |
| `[a-z]` | range inside a class | `[^a-z]` not in range |

### Anchors & look-arounds (zero-width: they assert, consume nothing)

| Token | Asserts |
|---|---|
| `\A` / `\z` | start / end of whole string (ignore `m`) |
| `(?= )` | lookahead: followed by … |
| `(?! )` | negative lookahead: not followed by … |
| `(?<= )` | lookbehind: preceded by … |
| `(?<! )` | negative lookbehind: not preceded by … |

```php
preg_match('/\d+(?= USD)/', '50 USD', $m); // $m[0]==='50' (number only IF followed by " USD")
preg_match('/(?<=\$)\d+/', '$50', $m);      // $m[0]==='50' (digits preceded by $)
```

### Pattern modifiers (flags after the closing delimiter)

| Flag | Effect |
|---|---|
| `i` | case-insensitive |
| `m` | multiline: `^`/`$` match at each line break |
| `s` | dotall: `.` also matches newline |
| `x` | extended: ignore whitespace in the pattern, allow `#` comments |
| `u` | treat pattern + subject as UTF-8 (always use for Unicode) |
| `U` | ungreedy: invert greedy/lazy (`*` becomes lazy) |

```php
preg_match('/^cafe$/i', 'CAFE');       // 1, case-insensitive
preg_match('/^b$/m', "a\nb\nc");        // 1, ^/$ per line
preg_match('/café/u', 'café');          // use u for multibyte text
```

## Files

```php
file_get_contents('f.txt');                 // entire file -> string
file_put_contents('f.txt', $data);          // write (truncates existing)
file_put_contents('f.txt', $data, FILE_APPEND);   // append instead of overwrite

$lines = file('f.txt', FILE_IGNORE_NEW_LINES);    // array of lines (no trailing \n)

$fh = fopen('f.txt', 'r');                  // 'r' read, 'w' write, 'a' append, 'r+' ...
while (($line = fgets($fh)) !== false) { echo $line; }
fclose($fh);

file_exists($p);   // bool
is_dir($p);        // bool
unlink($p);        // delete file
mkdir($p);         // create directory
scandir($dir);     // array of entries ('.', '..', files...)
```

## Tooling / runtime

```console
$ php script.php              # run a script
$ php -S localhost:8000       # built-in dev web server (docroot = cwd)
$ php -l file.php             # lint: syntax check only, no execution
$ php -a                      # interactive REPL
```

Composer (dependency manager):

```console
$ composer init               # create composer.json
$ composer require vendor/pkg # add a dependency
$ composer install            # install from composer.lock
$ composer update             # update to latest allowed versions
```

```php
require 'vendor/autoload.php';   // load Composer's PSR-4 autoloader
```

```json
{
    "require": {
        "vendor/pkg": "^1.0"
    }
}
```

Strict typing goes at the very top of each file, as the first statement:

```php
declare(strict_types=1);   // disable scalar type coercion in this file: TypeError instead of silent cast
```

Debug / inspection helpers:

```php
var_dump($x);    // types + values (verbose)
print_r($arr);   // readable structure
gettype($x);     // 'integer' | 'string' | 'array' ...
isset($x); empty($x); unset($x);
is_null($x); is_int($x); is_string($x); is_array($x); is_callable($x);

error_reporting(E_ALL); ini_set('display_errors', '1');  // dev only, never on in production
```

<!-- nav -->
---

← [PHP: Web Runtime](12-web-runtime.md) · [Index](README.md) · [PHP: Generators & Iteration](14-generators-iteration.md) →
<!-- nav -->
