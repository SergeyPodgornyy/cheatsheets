# PHP: Web Runtime

*Source: https://www.php.net/manual/en/language.variables.superglobals.php*

## Superglobals

Built-in arrays available in every scope, so no `global` keyword is needed inside functions.

| Superglobal | Holds |
|---|---|
| `$GLOBALS` | All variables in global scope, by name: `$GLOBALS['x']` |
| `$_SERVER` | Server / execution info: `REQUEST_METHOD`, `HTTP_HOST`, `REQUEST_URI`, `REMOTE_ADDR`, ... |
| `$_GET` | URL query-string params (`?page=2`) |
| `$_POST` | HTTP POST body (form data) |
| `$_REQUEST` | Merged `$_GET` + `$_POST` + `$_COOKIE` (avoid, the source is ambiguous) |
| `$_FILES` | Uploaded files |
| `$_COOKIE` | HTTP cookies sent by client |
| `$_SESSION` | Per-user session vars (after `session_start()`) |
| `$_ENV` | Environment variables |

```php
$_SERVER['REQUEST_METHOD'];  // 'GET' | 'POST' | ...
$_SERVER['HTTP_HOST'];       // 'example.com'
$_SERVER['REQUEST_URI'];     // '/path?x=1'
$_SERVER['REMOTE_ADDR'];     // client IP (string)
```

## Reading form data

**All input is untrusted.** Validate on input, escape on output. Use `??` to avoid *Undefined array key* warnings.

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'] ?? '';                                  // string, default ''
    $email    = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);  // false if invalid
    echo htmlspecialchars($username);                                      // escape on OUTPUT: prevents XSS
}

$page = (int)($_GET['page'] ?? 1);   // coerce + default; never trust raw type
```

`filter_input` / `filter_var` for validation:

```php
filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);      // int or false
filter_var($email, FILTER_VALIDATE_EMAIL);               // validate an existing value
// FILTER_VALIDATE_INT | _EMAIL | _URL | _BOOLEAN | _FLOAT ...
```

`htmlspecialchars` on every value echoed into HTML: it converts `<`, `>`, `&`, `"` so injected markup and scripts render as text.

## File uploads

Form must use `method="post"` and `enctype="multipart/form-data"`:

```html
<form method="post" enctype="multipart/form-data" action="upload.php">
  <input type="file" name="userfile">
  <button>Upload</button>
</form>
```

`$_FILES['userfile']` structure:

```php
// ['name' => client filename (UNTRUSTED), 'type' => MIME (UNTRUSTED, client-set),
//  'tmp_name' => server temp path, 'error' => int code, 'size' => bytes]

if (isset($_FILES['userfile']) && $_FILES['userfile']['error'] === UPLOAD_ERR_OK) {
    $tmp  = $_FILES['userfile']['tmp_name'];
    $name = basename($_FILES['userfile']['name']);   // sanitize: strip any path component
    $dest = __DIR__ . '/uploads/' . $name;

    // is_uploaded_file: confirm it really came via HTTP upload (not a forged path)
    if (is_uploaded_file($tmp) && move_uploaded_file($tmp, $dest)) {
        echo "OK";
    }
}
// UPLOAD_ERR_OK === 0; nonzero error codes mean the upload failed (size limit, partial, no file...)
```

Never trust `name`/`type` from the client. Derive the type server-side, sanitize the filename with `basename()`.

## Cookies

`setcookie()` must run before any output (it sends an HTTP header). Cookie is visible starting the *next* request.

```php
setcookie('user', 'Alice', [
    'expires'  => time() + 3600,   // 1h from now; 0 = session cookie
    'path'     => '/',
    'secure'   => true,            // HTTPS only
    'httponly' => true,            // not readable by JS: mitigates XSS theft
    'samesite' => 'Lax',           // CSRF mitigation: 'Lax' | 'Strict' | 'None'
]);

// Read (this is from a PREVIOUS request's Set-Cookie):
if (isset($_COOKIE['user'])) {
    echo htmlspecialchars($_COOKIE['user']);   // still untrusted, so escape
}

// Delete: re-set with a past expiry
setcookie('user', '', time() - 3600, '/');
```

## Sessions

`session_start()` before any output, on every page that uses the session. Data lives server-side, keyed by a session-id cookie.

```php
session_start();

$_SESSION['user_id'] = 42;                  // write
if (isset($_SESSION['user_id'])) { /*...*/ } // read

// Destroy / logout:
$_SESSION = [];        // clear in-memory data
session_destroy();     // remove server-side store
```

## Gotcha: superglobals as parameters

```php
function f($_POST)   // Fatal error: Cannot re-assign auto-global variable $_POST
{
}
```

Superglobals can't be used as function parameter names. Pass the needed value instead:

```php
function f(array $data)
{
    /* use $data */
}
f($_POST);   // OK
```

<!-- nav -->
---

← [PHP: Namespaces & Attributes](11-namespaces-attributes.md) · [Index](README.md) · [PHP: Database (PDO) & Tooling](13-database-tooling.md) →
<!-- nav -->
