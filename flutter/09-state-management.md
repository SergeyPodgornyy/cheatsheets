# Flutter — State Management

*Source: https://docs.flutter.dev/data-and-backend/state-mgmt/intro*

## Ephemeral vs App State

Two conceptual kinds of state. **Ephemeral** (UI/local) lives in one widget, others rarely need it, no serialization. **App state** is shared across many parts and kept between sessions.

```dart
// EPHEMERAL — current page in PageView, current tab, animation progress.
//   Tool: StatefulWidget + setState. Stays inside the widget.
// APP STATE — user prefs, login info, shopping cart, notifications.
//   Tool: a state-management solution (e.g. Provider).
```

No universal rule — **"do whatever is less awkward."** The line moves as the app grows; refactor freely.

## The Declarative Model

Flutter is **declarative**: the UI is a function of state — `UI = f(state)`. You don't mutate widgets imperatively; you **rebuild** them with new data.

```dart
// Widgets are IMMUTABLE: "They don't change — they get replaced."
// To change the screen, call setState() and return a fresh widget tree.
```

Ephemeral state with `setState` — the current tab of a `BottomNavigationBar`:

```dart
class _MyHomepageState extends State<MyHomepage> {
  int _index = 0; // local, ephemeral — no one else needs it

  @override
  Widget build(BuildContext context) {
    return BottomNavigationBar(
      currentIndex: _index,
      onTap: (newIndex) {
        setState(() {
          _index = newIndex; // mutate state, then rebuild
        });
      },
    );
  }
}
```

## Lifting State Up

When several widgets need the same data, **lift the state up** to a common ancestor and pass it down. For app-wide state this gets awkward (prop drilling) — that's the cue to reach for a state-management solution.

Flutter's low-level primitives: **InheritedWidget**, **InheritedNotifier**, **InheritedModel** — propagate data down the tree and let descendants rebuild on change. Higher-level packages (like Provider) wrap these so you rarely touch them directly.

## Provider

Recommended starting solution. Wraps `InheritedWidget` with less boilerplate.

```console
$ flutter pub add provider
```

Three concepts: **ChangeNotifier**, **ChangeNotifierProvider**, **Consumer**.

### ChangeNotifier

The model. Holds state, calls `notifyListeners()` whenever it changes.

```dart
class CartModel extends ChangeNotifier {
  final List<Item> _items = [];

  // Expose an UNMODIFIABLE view so outside code can't mutate without notify.
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  int get totalPrice => _items.length * 42;

  void add(Item item) {
    _items.add(item);
    notifyListeners(); // tells listening widgets to rebuild
  }

  void removeAll() {
    _items.clear();
    notifyListeners();
  }
}
```

### ChangeNotifierProvider

Provides a `ChangeNotifier` to descendants. **Auto-disposes** it when removed.

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: const MyApp(),
    ),
  );
}
```

`MultiProvider` for several providers (avoids nesting):

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (context) => CartModel()),
    Provider(create: (context) => SomeOtherClass()), // plain Provider for non-notifiers
  ],
  child: const MyApp(),
)
```

### Consumer

Rebuilds its `builder` when `notifyListeners()` fires. **Must** specify the type. Builder signature is `(context, model, child)`.

```dart
Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text('Total price: ${cart.totalPrice}');
  },
)
```

```dart
// GOTCHA: put Consumer as DEEP as possible — only its subtree rebuilds.
//   Wrapping a whole screen in Consumer rebuilds everything on every change.
// `child` arg = a subtree that does NOT depend on the model; built once,
//   passed back into builder, never rebuilt (optimization):
Consumer<CartModel>(
  builder: (context, cart, child) => Row(children: [child!, Text('${cart.totalPrice}')]),
  child: const Icon(Icons.shopping_cart), // built once, reused
)
```

### Reading Without Rebuilding

To call a method without subscribing to rebuilds, use `listen: false`:

```dart
Provider.of<CartModel>(context, listen: false).removeAll(); // read once, no rebuild
```

Extension-method equivalents from the provider package:

| Call | Behavior |
| --- | --- |
| `context.watch<T>()` | listen + rebuild on change (like `Consumer`) |
| `context.read<T>()` | read once, no rebuild (like `Provider.of(listen: false)`) |

```dart
context.watch<CartModel>().totalPrice; // rebuilds when cart changes
context.read<CartModel>().add(item);   // fire-and-forget, no rebuild
```

## InheritedWidget (the low-level primitive)

Provider and `Theme.of`/`MediaQuery.of` are built on **`InheritedWidget`** — it propagates data down the tree, and dependents **auto-rebuild** when it changes. Convention: a static `of(context)` using `dependOnInheritedWidgetOfExactType`, plus `updateShouldNotify`.

```dart
class FrogColor extends InheritedWidget {
  const FrogColor({super.key, required this.color, required super.child});
  final Color color;

  // Look up the nearest FrogColor; registers this context as a dependent.
  static FrogColor? maybeOf(BuildContext context) =>
      context.dependOnInheritedWidgetOfExactType<FrogColor>();

  static FrogColor of(BuildContext context) {
    final result = maybeOf(context);
    assert(result != null, 'No FrogColor found in context');
    return result!;
  }

  // Rebuild dependents only when the value actually changed.
  @override
  bool updateShouldNotify(FrogColor oldWidget) => color != oldWidget.color;
}

// Usage — context MUST be a descendant of FrogColor:
Widget build(BuildContext context) =>
    Text('Frog', style: TextStyle(color: FrogColor.of(context).color));
```

**Gotcha:** `.of(context)` asserts if `context` is a *parent* of the InheritedWidget — wrap the consumer in a `Builder` to get a descendant context. Related: `InheritedNotifier` (value is a `Listenable`), `InheritedModel` (subscribe to sub-parts).

<!-- nav -->
---

← [Flutter — Navigation & Routing](08-navigation.md) · [Index](../README.md) · [Flutter — Networking & Async UI](10-networking-async-ui.md) →
<!-- nav -->
