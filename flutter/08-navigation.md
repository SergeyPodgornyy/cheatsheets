# Flutter — Navigation & Routing

*Source: https://docs.flutter.dev/ui/navigation*

## Navigator — the stack model

**`Navigator`** manages a **stack** of `Route` objects. **Push** a route to put a new screen on top; **pop** to remove it and reveal the one beneath. Transitions match the platform.

## Push & pop

```dart
// MaterialPageRoute = Material-style transition
Navigator.of(context).push(
  MaterialPageRoute<void>(builder: (context) => const SecondScreen()),
);

// Short form
Navigator.push(
  context,
  MaterialPageRoute<void>(builder: (context) => const SecondScreen()),
);

// Go back
Navigator.pop(context);
```

## Passing data via constructor (recommended)

Plain Dart — pass the data into the next screen's constructor.

```dart
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key, required this.todo});
  final Todo todo;                                    // received via constructor

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(todo.title)),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Text(todo.description),
      ),
    );
  }
}

// Navigate and hand over the data:
onTap: () {
  Navigator.push(
    context,
    MaterialPageRoute<void>(builder: (context) => DetailScreen(todo: todos[index])),
  );
},
```

### Alternative — RouteSettings arguments

```dart
Navigator.push(
  context,
  MaterialPageRoute<void>(
    builder: (context) => const DetailScreen(),
    settings: RouteSettings(arguments: todos[index]),   // attach payload
  ),
);

// Read it inside DetailScreen:
final todo = ModalRoute.of(context)!.settings.arguments as Todo;
```

## Returning data

`push` returns a `Future` that completes when the pushed route pops. `await` it; type the route (`MaterialPageRoute<String>`) so the result is typed.

```dart
Future<void> _navigateAndDisplaySelection(BuildContext context) async {
  final result = await Navigator.push(
    context,
    MaterialPageRoute<String>(builder: (context) => const SelectionScreen()),
  );

  // Gotcha: after an async gap the widget may be gone — check before using context
  if (!context.mounted) return;

  ScaffoldMessenger.of(context)
    ..removeCurrentSnackBar()
    ..showSnackBar(SnackBar(content: Text('$result')));
}

// On the second screen — pass the result as pop's second arg:
Navigator.pop(context, 'Yep!');
```

## Named routes (NOT recommended)

Declare routes in `MaterialApp.routes`, navigate by name. **Avoid for most apps** — can't customize deep-link behavior, and the browser forward button won't work. Prefer `Navigator` + `MaterialPageRoute`, or `go_router`.

```dart
Navigator.pushNamed(context, '/second');

MaterialApp(
  routes: {
    '/': (context) => const Home(),
    '/second': (context) => const Second(),
  },
)
```

## go_router — declarative routing (recommended for advanced / web)

Apps with advanced navigation or web deep links should use the **`Router`** (via `MaterialApp.router`) with the **`go_router`** package.

```dart
context.go('/second');                              // declarative navigation

MaterialApp.router(
  routerConfig: router,                             // GoRouter config
);
```

<!-- nav -->
---

← [Flutter — Input & Forms](07-input-forms.md) · [Index](README.md) · [Flutter — State Management](09-state-management.md) →
<!-- nav -->
