# PHP: Dependency Injection & Reflection

*Source: https://www.php.net/manual/en/class.reflectionclass.php*

## DEPENDENCY INJECTION

**DI** = give an object its dependencies from outside instead of creating them inside. Decouples code, makes testing/mocking trivial.

### Not DI: hard-wired dependency

The class builds its own `PDO` internally: tightly coupled, can't swap the DB or mock it in tests.

```php
class UserService
{
    private PDO $db;

    public function __construct()
    {
        // tightly coupled: caller has no say, tests hit a real DB
        $this->db = new PDO('mysql:host=localhost;dbname=app', 'u', 'p');
    }
}
```

### Constructor injection (preferred)

Dependencies are explicit and required: you can't construct the object without them. Promoted params keep it terse.

```php
class UserService
{
    public function __construct(
        private PDO $db,
        private LoggerInterface $log,
    ) {
    }

    public function find(int $id): array
    {
        $this->log->info("find $id");
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        return $stmt->fetch();
    }
}

// caller wires it (the "composition root"):
$service = new UserService($pdo, $logger);
```

### Setter / method injection

For optional dependencies, set after construction.

```php
class UserService
{
    private ?LoggerInterface $log = null;

    public function setLogger(LoggerInterface $l): void // inject later, optional
    {
        $this->log = $l;
    }
}
```

### Depend on interfaces, not concretions

Type-hint the interface so any implementation drops in: swap transports, mock in tests.

```php
class Mailer
{
    public function __construct(private TransportInterface $transport) {} // not SmtpTransport
}

// production: new Mailer(new SmtpTransport());
// tests:      new Mailer($mockTransport);
```

### DI container (PSR-11)

A **container** builds + wires objects for you, resolving the whole dependency graph on demand.

```php
use Psr\Container\ContainerInterface;
// ContainerInterface:
//   get(string $id): mixed   returns the entry for $id (constructs deps recursively)
//   has(string $id): bool    does an entry exist for $id?

$service = $container->get(UserService::class); // builds PDO + Logger + UserService
$container->has(LoggerInterface::class);         // true / false
```

| Method | Description |
|---|---|
| `get(string $id): mixed` | resolve + return the entry; constructs dependencies recursively |
| `has(string $id): bool` | whether the container can resolve `$id` |

**Autowiring**: many containers use Reflection to read constructor parameter types and resolve them automatically, with no manual wiring. Popular containers are PHP-DI, Symfony DI, and the Laravel container.

### Manual mini-container

Factories build entries lazily; `??=` caches the first build (singleton).

```php
class Container implements ContainerInterface
{
    private array $factories = [];
    private array $instances = [];

    public function set(string $id, callable $factory): void
    {
        $this->factories[$id] = $factory;
    }

    public function has(string $id): bool
    {
        return isset($this->factories[$id]);
    }

    public function get(string $id): mixed
    {
        // build once, then reuse (singleton); pass $this so factories can resolve deps
        return $this->instances[$id] ??= ($this->factories[$id])($this);
    }
}

$c = new Container();
$c->set(PDO::class, fn() => new PDO($dsn, $u, $p));
$c->set(
    UserService::class,
    fn($c) => new UserService($c->get(PDO::class), $c->get(LoggerInterface::class)),
);
$svc = $c->get(UserService::class); // wires PDO + UserService automatically
```

## REFLECTION

**Reflection** = inspect and manipulate classes, methods, properties, functions, and attributes at runtime.

### ReflectionClass

```php
$rc = new ReflectionClass(Dog::class);    // always pass the FULLY-QUALIFIED name

$rc->getName();                           // "Dog"
$rc->isAbstract();                        // bool
$rc->getParentClass()->getName();         // "Animal" (getParentClass() -> ReflectionClass|false)
$rc->getInterfaceNames();                 // string[] of implemented interfaces
$rc->getConstants();                      // ['SOUND' => 'woof']

foreach ($rc->getMethods() as $m) {       // ReflectionMethod[]
    echo $m->getName();
}
foreach ($rc->getProperties() as $p) {    // ReflectionProperty[]
    echo $p->getName();
}

$dog = $rc->newInstanceArgs(['Rex', 3]);  // construct dynamically from an args array

// filter by visibility:
$rc->getMethods(ReflectionMethod::IS_PUBLIC); // public methods only
```

| Method | Description |
|---|---|
| `getName()` | class name, e.g. `"Dog"` |
| `isAbstract()` | bool: is it `abstract`? |
| `getParentClass()` | `ReflectionClass` of parent, or `false` |
| `getInterfaceNames()` | `string[]` of implemented interfaces |
| `getConstants()` | `['SOUND' => 'woof']` |
| `getMethods(?int $filter)` | `ReflectionMethod[]`; filter e.g. `IS_PUBLIC` |
| `getProperties(?int $filter)` | `ReflectionProperty[]` |
| `newInstanceArgs(array $args)` | instantiate from an args array |

### ReflectionMethod

```php
$rm = new ReflectionMethod(Dog::class, 'speak');

$rm->isPublic();          // true
$rm->getReturnType();     // ReflectionNamedType -> "string"
echo $rm->invoke($dog);   // call speak() on the $dog instance
$rm->invokeArgs($dog, []); // call with an args array
```

### ReflectionProperty: read/write, even non-public

Reflection bypasses visibility: it reads and writes `private`/`protected` members.

```php
$rp = new ReflectionProperty(Dog::class, 'age'); // protected property

echo $rp->getValue($dog); // read
$rp->setValue($dog, 5);   // write, bypasses visibility
$rp->getType();           // ReflectionNamedType
```

### ReflectionFunction + ReflectionParameter

```php
$rf = new ReflectionFunction('greet');

$rf->getNumberOfParameters();         // total count
$rf->getNumberOfRequiredParameters(); // required-only count

foreach ($rf->getParameters() as $param) { // ReflectionParameter[]
    $param->getName();
    $param->getType();          // ReflectionNamedType | null
    $param->isOptional();       // bool
    $param->getDefaultValue();  // value (only if it has one)
}

echo $rf->invokeArgs(['Ann', 2]); // call the function with an args array
```

### ReflectionNamedType

```php
$type = $param->getType();

$type->getName();    // "array"
$type->allowsNull(); // bool: true if declared as ?type
$type->isBuiltin();  // true for int/string/array/...; false for class/interface types
```

### Reading attributes via Reflection

Reflection is how attributes are consumed: read metadata off declarations, then instantiate.

```php
$rc = new ReflectionClass(UserController::class);

foreach ($rc->getAttributes(Route::class) as $attr) { // ReflectionAttribute[]
    $attr->getArguments();         // ['/users', 'method' => 'POST'] (raw args)
    $route = $attr->newInstance(); // instantiate the attribute class -> Route
    echo $route->path;
}

// method-level attributes:
$rc->getMethod('index')->getAttributes(Route::class);

// getAttributes(?string $name = null, int $flags = 0);
//   $name  filters by attribute class (null = all)
//   $flags ReflectionAttribute::IS_INSTANCEOF matches subclasses too
```

**Uses:** DI autowiring (read constructor types), ORMs/serializers (map attributes → columns), test frameworks, validators.

**Gotcha:** reflecting an aliased class resolves to the real class: `getName()` returns the original FQN, not the alias.

<!-- nav -->
---

← [PHP: Security](16-security.md) · [Index](README.md) · [PHP: Advanced (Fibers, Streams, Closures, Serialization)](18-advanced.md) →
<!-- nav -->
