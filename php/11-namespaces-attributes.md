# PHP — Namespaces & Attributes

*Source: https://www.php.net/manual/en/language.namespaces.php*

## Declaring a namespace

Must be the **first statement** in a file (only `declare` may precede it). Scopes classes, functions, and constants.

```php
<?php
namespace App\Models;

class User
{
}
function helper()
{
}
const VERSION = 1;
```

## Sub-namespaces

Directory-like hierarchy, separated by `\`.

```php
namespace App\Http\Controllers;
```

## `use` — importing

```php
use App\Models\User;             // import a class
use App\Models\Post as Article;  // alias
use function App\Models\helper;  // import a function
use const App\Models\VERSION;    // import a constant

$u = new User();
$a = new Article();
helper();
echo VERSION;
```

## Global prefix `\`

Inside a namespace, unqualified names resolve to the current namespace first. Prefix with `\` to reach the **global** namespace explicitly (built-ins, root-level classes).

```php
namespace App;

$len = \strlen('hi');     // explicit global function
$c = new \DateTime();     // global class — without \ PHP looks for App\DateTime
```

## Fallback rule

Unqualified **function** and **constant** names fall back to the global namespace if not found locally. **Classes do NOT** fall back — they must be imported or `\`-prefixed.

```php
namespace App;

echo strlen('hi');   // OK — function falls back to global \strlen
echo PHP_EOL;        // OK — constant falls back to global

$d = new DateTime(); // Error: Class "App\DateTime" not found (no class fallback)
$d = new \DateTime();// OK
```

## `__NAMESPACE__`

Magic constant holding the current namespace string. `namespace\X` refers to something in the current namespace.

```php
namespace App;
echo __NAMESPACE__;        // "App"
$class = __NAMESPACE__ . '\\User';
echo namespace\VERSION;    // current-namespace constant
```

## PSR-4 autoloading (Composer)

Map a namespace prefix to a base directory; the **fully-qualified class name** maps to a file path. Loading `vendor/autoload.php` registers the autoloader.

```php
// composer.json
// {
//   "autoload": { "psr-4": { "App\\": "src/" } }
// }
//
// App\Models\User   ->   src/Models/User.php

require 'vendor/autoload.php';
$user = new App\Models\User();   // file loaded on demand
```

Manual autoloader (no Composer):

```php
spl_autoload_register(
    fn($c) => require __DIR__ . '/src/' . str_replace('\\', '/', $c) . '.php'
);
```

## PSR standards

**PSRs** (PHP Standard Recommendations, by the **PHP-FIG** group) are shared interfaces and conventions that let independent libraries interoperate. You depend on the **interface** package (e.g. `psr/log`) and any compliant implementation drops in.

| PSR | Title | Purpose |
|---|---|---|
| **PSR-1** | Basic Coding Standard | fundamental code conventions (class/method naming, `<?php` tags) |
| **PSR-12** | Extended Coding Style | full formatting rules (indentation, spacing, line length) — supersedes PSR-2 |
| **PSR-4** | Autoloading | map namespace prefix → directory (the Composer `autoload` standard) |
| **PSR-3** | Logger Interface | `LoggerInterface` — `$log->info()/error()/...` (pkg `psr/log`) |
| **PSR-11** | Container Interface | DI container — `$c->get($id)` / `$c->has($id)` (`psr/container`) |
| **PSR-6** / **PSR-16** | Caching / Simple Cache | pool+item caching / simpler `get`/`set`/`delete` (`psr/cache`, `psr/simple-cache`) |
| **PSR-7** | HTTP Message | `RequestInterface` / `ResponseInterface` value objects (`psr/http-message`) |
| **PSR-17** | HTTP Factories | factories that create PSR-7 objects (`psr/http-factory`) |
| **PSR-15** | HTTP Handlers | server request handlers + middleware (`psr/http-server-handler`) |
| **PSR-18** | HTTP Client | `$client->sendRequest($req)` (`psr/http-client`) |
| **PSR-14** | Event Dispatcher | dispatch events to listeners (`psr/event-dispatcher`) |
| **PSR-20** | Clock | `$clock->now()` → mockable time (`psr/clock`) |

```php
// PSR-3 logging — code depends on the interface, not a concrete logger:
use Psr\Log\LoggerInterface;
function process(LoggerInterface $log): void
{
    $log->info('started', ['user' => 42]); // works with Monolog, etc.
}
```

**Deprecated:** PSR-0 (old autoloading → use PSR-4), PSR-2 (old style → use PSR-12).

## Attributes

Structured, machine-readable metadata attached to declarations (classes, methods, properties, params...). Read back via Reflection.

### Declaring an attribute

An attribute is a class marked `#[Attribute]`. Promoted constructor params become its data.

```php
use Attribute;

#[Attribute]
class Route
{
    public function __construct(
        public string $path,
        public string $method = 'GET',
    ) {
    }
}
```

### Using an attribute

Place `#[...]` above the target; constructor args (incl. named args) are passed inline.

```php
class UserController
{
    #[Route('/users', method: 'GET')]
    public function index()
    {
    }
}
```

### Reading via Reflection

```php
$r = new \ReflectionMethod(UserController::class, 'index');

foreach ($r->getAttributes(Route::class) as $attr) {
    $route = $attr->newInstance();   // instantiate -> Route object
    echo $route->path;               // "/users"
    echo $route->method;             // "GET"
}
```

### Built-in attributes

| Attribute | Purpose |
|-----------|---------|
| `#[Attribute]` | marks a class as usable as an attribute |
| `#[Override]` | method intended to override a parent; errors if it doesn't |
| `#[Deprecated]` | marks a function/method/constant deprecated |
| `#[NoDiscard]` | warns if the return value is ignored |
| `#[SensitiveParameter]` | redacts the parameter value from stack traces |
| `#[ReturnTypeWillChange]` | suppresses a return-type compatibility warning |

<!-- nav -->
---

← [PHP — Exceptions & Errors](10-exceptions.md) · [Index](README.md) · [PHP — Web Runtime](12-web-runtime.md) →
<!-- nav -->
