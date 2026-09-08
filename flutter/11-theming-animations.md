# Flutter: Theming & Animations

*Source: https://docs.flutter.dev/cookbook/design/themes*

## Theming

A `ThemeData` defines app-wide visuals. It most often sets `colorScheme` (colors) and `textTheme` (text styles). Apply it via `MaterialApp(theme:)`.

```dart
MaterialApp(
  title: appName,
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.purple,
      brightness: Brightness.dark, // or Brightness.light; derives a full palette
    ),
    textTheme: TextTheme(
      displayLarge: const TextStyle(fontSize: 72, fontWeight: FontWeight.bold),
      titleLarge: GoogleFonts.oswald(fontSize: 30, fontStyle: FontStyle.italic),
      bodyMedium: GoogleFonts.merriweather(),
      displaySmall: GoogleFonts.pacifico(),
    ),
  ),
  home: const MyHomePage(title: appName),
)
```

### Styling order

Most-specific wins:

A widget-specific style beats the nearest `Theme` override, which beats the app theme. A `TextStyle` passed directly to a `Text`, for example, wins over anything the theme sets.

### Reading the theme

`Theme.of(context)` walks up the tree to the nearest `Theme` (the app theme by default).

```dart
Container(
  color: Theme.of(context).colorScheme.primary,
  child: Text(
    'Text with a background color',
    style: Theme.of(context).textTheme.bodyMedium!.copyWith(
      color: Theme.of(context).colorScheme.onPrimary,
    ),
  ),
)
```

### Overriding for a subtree

Wrap part of the tree in a `Theme` widget, with either a brand-new `ThemeData` or `copyWith` to extend the current one.

```dart
// Unique ThemeData: replaces everything for this subtree:
Theme(
  data: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.pink)),
  child: FloatingActionButton(onPressed: () {}, child: const Icon(Icons.add)),
)

// copyWith: inherit the app theme, override only what you pass:
Theme(
  data: Theme.of(context).copyWith(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.pink),
  ),
  child: const FloatingActionButton(onPressed: null, child: Icon(Icons.add)),
)
```

### Dark mode

```dart
// Build a dark palette with brightness:
ColorScheme.fromSeed(seedColor: Colors.purple, brightness: Brightness.dark);

// MaterialApp supports both themes + a mode switch:
MaterialApp(
  theme: ThemeData(/* light */),
  darkTheme: ThemeData(/* dark */),
  themeMode: ThemeMode.system, // .light / .dark / .system (follows OS)
)
```

## Animations

Two families: **implicit** (set-and-forget, you change a property) and **explicit** (you drive with a controller).

### Implicit animations

`ImplicitlyAnimatedWidget` family: store animatable props as state, change them with `setState`, and the widget auto-animates between old and new. Required: `duration`. Optional: `curve`.

```dart
AnimatedContainer(
  width: _width,
  height: _height,
  decoration: BoxDecoration(color: _color, borderRadius: _borderRadius),
  duration: const Duration(seconds: 1), // required
  curve: Curves.fastOutSlowIn,          // optional easing
)
```

Trigger by mutating the state; no controller needed:

```dart
setState(() {
  _width = random.nextInt(300).toDouble();
  _height = random.nextInt(300).toDouble();
  _color = Color.fromRGBO(random.nextInt(256), random.nextInt(256), random.nextInt(256), 1);
  _borderRadius = BorderRadius.circular(random.nextInt(100).toDouble());
});
```

Family members:

| Widget | Animates |
| --- | --- |
| `AnimatedContainer` | size, color, decoration, padding... |
| `AnimatedOpacity` | opacity (fade in/out) |
| `AnimatedPadding` | padding |
| `AnimatedPositioned` | position inside a `Stack` |
| `AnimatedSwitcher` | swaps between two children with a transition |

### Explicit animations

You control timing with an `AnimationController`. It needs a `vsync` ticker, so mix in `SingleTickerProviderStateMixin`. A `Tween` maps the controller's `0..1` to your value range. **Dispose the controller!**

```dart
class _LogoAppState extends State<LogoApp> with SingleTickerProviderStateMixin {
  late Animation<double> animation;
  late AnimationController controller;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this, // provided by SingleTickerProviderStateMixin
    );
    animation = Tween<double>(begin: 0, end: 300).animate(controller)
      ..addListener(() {
        setState(() {
          // animation.value changed, rebuild to show it
        });
      });
    controller.forward(); // start
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        margin: const EdgeInsets.symmetric(vertical: 10),
        height: animation.value,
        width: animation.value,
        child: const FlutterLogo(),
      ),
    );
  }

  @override
  void dispose() {
    controller.dispose(); // MUST: leaks the ticker otherwise
    super.dispose();
  }
}
```

### AnimatedBuilder

Auto-listens to the animation, so there is no `addListener` / `setState` boilerplate, and only its `builder` rebuilds.

```dart
AnimatedBuilder(
  animation: animation,
  builder: (context, child) {
    return SizedBox(height: animation.value, width: animation.value, child: child);
  },
  child: child, // built once, not rebuilt on each tick (optimization)
)
```

### Status listeners & looping

`addStatusListener` reacts to `AnimationStatus` (`forward` / `completed` / `reverse` / `dismissed`):

```dart
animation = Tween<double>(begin: 0, end: 300).animate(controller)
  ..addStatusListener((status) {
    if (status == AnimationStatus.completed) {
      controller.reverse();        // ping-pong loop
    } else if (status == AnimationStatus.dismissed) {
      controller.forward();
    }
  });
```

### Controls & curves

```dart
controller.forward(); // play forward
controller.reverse(); // play backward
controller.repeat();  // loop continuously

// Apply easing without a status loop:
final curved = CurvedAnimation(parent: controller, curve: Curves.easeIn);
```

### Built-in transitions

Wrap a child and drive it with an `Animation`, with no manual `value` plumbing:

```dart
// FadeTransition, SizeTransition, ScaleTransition, SlideTransition, RotationTransition
FadeTransition(opacity: animation, child: const FlutterLogo());
```

### Hero

Shared-element transition between routes: the same `tag` on both screens animates the widget across the route change:

```dart
Hero(tag: 'logo', child: Image.asset('logo.png'));
```

<!-- nav -->
---

← [Flutter: Networking & Async UI](10-networking-async-ui.md) · [Index](README.md) · [Flutter: Testing](12-testing.md) →
<!-- nav -->
