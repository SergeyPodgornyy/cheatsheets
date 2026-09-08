# PHP: Security

*Source: https://www.php.net/manual/en/security.php*

## Password hashing

Password hashes must be slow and salted per password. Never use `md5()`/`sha1()`: they are fast and unsalted, so they fall to GPU brute-force and rainbow tables.

```php
// Hash on signup. Algo + cost + salt are embedded in the output string.
// Store that ONE string → DB column >= 255 chars.
$hash = password_hash($password, PASSWORD_DEFAULT);  // currently bcrypt; may change
$hash = password_hash($password, PASSWORD_ARGON2ID); // modern recommended
$hash = password_hash($password, PASSWORD_BCRYPT, ['cost' => 13]); // tune cost

// output: $2y$12$<22-char salt><31-char hash>   ($2y$ = bcrypt, 12 = cost)
```

Tune `cost` to ~250-350 ms per hash on your server: slow for attackers, tolerable for users.

```php
// Verify on login. Extracts salt/cost from $hash; already timing-safe.
if (password_verify($password, $hash)) {
    // Transparently upgrade old hashes when params change:
    if (password_needs_rehash($hash, PASSWORD_DEFAULT)) {
        $hash = password_hash($password, PASSWORD_DEFAULT);
        update_db($userId, $hash); // re-store stronger hash
    }
}
```

**Gotchas:**
```php
// Do NOT supply your own salt: deprecated and ignored. Auto salt uses the OS CSPRNG.
// bcrypt truncates input at 72 bytes and STOPS at a NUL byte.
$hash = password_hash(bin2hex($binary), PASSWORD_BCRYPT); // never feed raw binary; hex first
```

## Timing-safe comparison

For manual secret/token/HMAC comparison (not needed with `password_verify`, which is already constant-time).

```php
if (hash_equals($expected, $provided)) { // constant-time
    // ...
}
// '===' on secrets leaks length/content via timing → use hash_equals
```

## Secure randomness (CSPRNG)

For tokens, codes, salts. Never use `rand()`/`mt_rand()`/`uniqid()` for security; they are predictable.

```php
$token = bin2hex(random_bytes(32));  // 256-bit hex token
$code  = random_int(100000, 999999); // secure 6-digit PIN
```

## SQL injection

Bind ALL data via prepared statements; allowlist any non-data SQL part.

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?');
$stmt->execute([$email]); // value is bound, never concatenated

// Dynamic ORDER BY can't be bound → validate against a known list:
$dir = $_GET['dir'] === 'DESC' ? 'DESC' : 'ASC'; // allowlist
// Connect with a minimal-privilege DB user.
```

## XSS

Escape output for its context.

```php
echo htmlspecialchars($userInput, ENT_QUOTES | ENT_HTML5, 'UTF-8'); // HTML + attributes
echo rawurlencode($v); // URL context
echo json_encode($v);  // JS context
// + send a Content-Security-Policy header for defense in depth.
```

## CSRF

Per-session random token + timing-safe check.

```php
session_start();
if (empty($_SESSION['csrf'])) {
    $_SESSION['csrf'] = bin2hex(random_bytes(32)); // one token per session
}

// On POST, compare submitted token constant-time:
if (!hash_equals($_SESSION['csrf'], $_POST['csrf'] ?? '')) {
    http_response_code(403);
    exit('Invalid CSRF token');
}
```

Embed the token as a hidden field in every state-changing form:

```html
<input type="hidden" name="csrf" value="<?= $_SESSION['csrf'] ?>">
```

## Input validation

Never trust ANY input, including selects, hidden fields, and cookies.

```php
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL); // false if invalid
$id    = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
$url   = filter_var($v, FILTER_VALIDATE_URL);

if (!ctype_digit($_GET['page'] ?? '')) { // digits-only check
    exit('bad page');
}
// validators: FILTER_VALIDATE_INT | EMAIL | URL | BOOLEAN | FLOAT | IP | DOMAIN
```

## RCE & file inclusion

Never put user input into `eval`/`include`/`require`/shell.

```php
// VULNERABLE: include $_GET['page'] . '.php';  // LFI/RFI
$pages = ['home' => 'home.php', 'about' => 'about.php'];
include $pages[$_GET['page'] ?? 'home'] ?? $pages['home']; // allowlist

$arg = escapeshellarg($userInput); // if you MUST shell out
```

```ini
; php.ini: disable dangerous functions
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,eval
```

## Don't leak errors in production

Error messages can reveal schema, paths, internals.

```ini
; php.ini
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log
```

```php
ini_set('display_errors', '0');
error_reporting(E_ALL); // log all, show none
```

Other hardening: set cookies `httponly` + `secure` + `samesite`; validate uploads with `is_uploaded_file` + verify type server-side; keep dependencies patched (`composer audit`).

| Threat | Defense |
| --- | --- |
| SQL injection | prepared statements + allowlist |
| XSS | `htmlspecialchars(ENT_QUOTES)` / context escaping + CSP |
| CSRF | per-session `random_bytes` token + `hash_equals` |
| Bad input | `filter_input` / `filter_var` / `ctype_*` |
| RCE / LFI | allowlist; no user input in `eval`/`include`/shell |
| Weak passwords | `password_hash` + `password_verify` (never md5/sha1) |
| Info leak | `display_errors` Off, `log_errors` On |

<!-- nav -->
---

← [PHP: SPL (Standard PHP Library)](15-spl.md) · [Index](README.md) · [PHP: Dependency Injection & Reflection](17-di-reflection.md) →
<!-- nav -->
