# Flutter — Widgets & State

*Source: https://docs.flutter.dev/ui/widgets-intro*

## StatelessWidget

Receives args from its parent, stores them in **`final` fields**, and uses them in `build()` to derive output. Fields in any `Widget` subclass are **always `final`**.

```dart
class Greeting extends StatelessWidget {
  const Greeting({super.key, required this.name});
  final String name; // final — immutable config from parent

  @override
  Widget build(BuildContext context) => Text('Hello, $name!');
}
```

## StatefulWidget + State

Mutable state lives in a separate **`State`** object. `createState()` wires the two together.

```dart
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _counter = 0;

  void _increment() {
    setState(() {
      // setState tells the framework state changed -> reruns build().
      // Without setState(), build() isn't called and NOTHING updates.
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        ElevatedButton(onPressed: _increment, child: const Text('Increment')),
        const SizedBox(width: 16),
        Text('Count: $_counter'),
      ],
    );
  }
}
```

## The Two-Object Model

```dart
// Widget object  -> TEMPORARY. Recreated cheaply on every rebuild. Holds config.
// State object   -> PERSISTS across build() calls. Holds mutable data.
```

- `createState()` is called the **first time** a widget appears at a location in the tree.
- If the parent rebuilds with the **same runtimeType (+ same key)**, the framework **reuses** the existing `State` object.
- Access the (possibly new) widget's props from `State` via the **`widget`** property.

```dart
class _ProductState extends State<ProductView> {
  @override
  Widget build(BuildContext context) => Text(widget.product.name); // widget.<prop>
}
```

## Lifting State Up (Callbacks)

A child doesn't mutate its own state — it **calls a parent callback**. **Change flows up** via callbacks; **state flows down** to stateless presentation widgets. The common parent's `State` redirects.

```dart
typedef CartChangedCallback = void Function(Product product, bool inCart);

class ShoppingListItem extends StatelessWidget {
  ShoppingListItem({
    required this.product,
    required this.inCart,
    required this.onCartChanged,
  }) : super(key: ObjectKey(product)); // key derived from the product identity

  final Product product;
  final bool inCart;
  final CartChangedCallback onCartChanged;

  // build() wires ListTile.onTap -> onCartChanged(product, inCart)
}

class _ShoppingListState extends State<ShoppingList> {
  final _shoppingCart = <Product>{};

  void _handleCartChanged(Product product, bool inCart) {
    setState(() {
      if (!inCart) {
        _shoppingCart.add(product);
      } else {
        _shoppingCart.remove(product);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Shopping List')),
      body: ListView(
        padding: const EdgeInsets.symmetric(vertical: 8),
        children: widget.products.map((product) {
          return ShoppingListItem(
            product: product,
            inCart: _shoppingCart.contains(product), // state flows DOWN
            onCartChanged: _handleCartChanged,        // change flows UP
          );
        }).toList(),
      ),
    );
  }
}
```

## BuildContext

A handle to **where** in the tree a build is happening — lets lookups find ancestor data (e.g. the right theme).

```dart
Theme.of(context).primaryColor; // walks up from this context to nearest Theme
```

## Lifecycle

```dart
class _MyState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();          // MUST be called FIRST
    // one-time setup: animations, subscriptions
  }

  @override
  void didUpdateWidget(MyWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // called when the parent rebuilds this widget with new config
    // compare against oldWidget.<prop> to react to changes
  }

  @override
  void dispose() {
    // cleanup: cancel timers, unsubscribe, dispose controllers
    super.dispose();            // typically called LAST
  }
}
```

## Keys

Keys control **which widgets the framework matches** on rebuild.

```dart
// Default match  = runtimeType + position/order among siblings.
// With a key     = match requires same key AND same runtimeType.
// Use when reordering a list of same-type widgets to PRESERVE their state.
```

| Key | Scope / use |
|---|---|
| `ValueKey(v)` | local — keyed by a value (`ValueKey('a')`) |
| `ObjectKey(o)` | local — keyed by object identity (`ObjectKey(product)`) |
| `UniqueKey()` | local — guaranteed unique among siblings |
| `GlobalKey()` | globally unique; can retrieve a widget's `State` from anywhere |

Local keys must be unique **among siblings**; a `GlobalKey` is unique across the whole app.

<!-- nav -->
---

← [Flutter — Basics](01-basics.md) · [Index](README.md) · [Flutter — Layout](03-layout.md) →
<!-- nav -->
