# Flutter — Layout

*Source: https://docs.flutter.dev/ui/layout*

## Everything Is a Widget

Layouts are built from widgets — **compose** simple widgets into complex ones.

```dart
// Single-child layout widgets take a `child`:   Center, Container, Padding
// Multi-child  layout widgets take `children`:  Row, Column, ListView, Stack
```

## App Skeletons

```dart
// Material
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    const String appTitle = 'Flutter layout demo';
    return MaterialApp(
      title: appTitle,
      home: Scaffold(
        appBar: AppBar(title: const Text(appTitle)),
        body: const Center(child: Text('Hello World')),
      ),
    );
  }
}
```

```dart
// Cupertino (iOS-style)
const CupertinoApp(
  theme: CupertinoThemeData(
    brightness: Brightness.light,
    primaryColor: CupertinoColors.systemBlue,
  ),
  home: CupertinoPageScaffold(
    navigationBar: CupertinoNavigationBar(middle: Text('Flutter layout demo')),
    child: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [Text('Hello World')],
      ),
    ),
  ),
)
```

## Center

```dart
const Center(child: Text('Hello World'));
```

## Row & Column

Lay children out along an axis. Low-level primitives; nest freely.

```dart
// Row    -> horizontal: main axis = horizontal, cross axis = vertical
// Column -> vertical:   main axis = vertical,   cross axis = horizontal

Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: [
    Image.asset('images/pic1.jpg'),
    Image.asset('images/pic2.jpg'),
    Image.asset('images/pic3.jpg'),
  ],
)
```

### Alignment enums

```dart
// MainAxisAlignment:  start, end, center, spaceBetween, spaceAround, spaceEvenly
// CrossAxisAlignment: start, end, center, stretch, baseline
```

## Expanded & Flexible

`Expanded` fills the **available space** along the main axis (flex factor defaults to `1`).

```dart
Row(
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Expanded(child: Image.asset('images/pic1.jpg')),          // flex 1
    Expanded(flex: 2, child: Image.asset('images/pic2.jpg')), // 2x the share
    Expanded(child: Image.asset('images/pic3.jpg')),          // flex 1
  ],
)
```

```dart
// Expanded -> forces the child to FILL its allotted share (exact size).
// Flexible -> lets the child be SMALLER than its allotted share.
```

## mainAxisSize.min

Pack children tightly instead of filling the main axis (default is `MainAxisSize.max`).

```dart
Row(
  mainAxisSize: MainAxisSize.min, // row shrinks to fit its children
  children: [
    Icon(Icons.star, color: Colors.green[500]),
    const Icon(Icons.star, color: Colors.black),
  ],
)
```

## Overflow Warning

```dart
// Content too large for its box -> yellow/black striped bar in debug.
// Fix by using Expanded/Flexible, a scroll view, or constraining the child.
```

## Container

Single child. Combines painting (`decoration`/`color`), positioning (`margin`/`padding`), and sizing.

```dart
// Box model: margin -> border -> padding -> content
Container(
  decoration: BoxDecoration(
    border: Border.all(width: 10, color: Colors.black38),
    borderRadius: const BorderRadius.all(Radius.circular(8)),
  ),
  margin: const EdgeInsets.all(4),
  child: Image.asset('images/pic.jpg'),
)
```

### EdgeInsets variants

```dart
const EdgeInsets.all(8);                          // all sides
const EdgeInsets.symmetric(horizontal: 8, vertical: 16);
const EdgeInsets.fromLTRB(20, 30, 20, 20);        // left, top, right, bottom
const EdgeInsets.only(left: 8);                   // one side
```

## Stack

Overlap widgets. First child = **base**; subsequent children are **overlaid**. Can't scroll. Place children with `Positioned` or via `alignment`.

```dart
Stack(
  alignment: const Alignment(0.6, 0.6), // -1..1 on each axis; 0,0 = center
  children: [
    const CircleAvatar(
      backgroundImage: AssetImage('images/pic.jpg'),
      radius: 100, // base layer
    ),
    Container(
      decoration: const BoxDecoration(color: Colors.black45),
      child: const Text(
        'Mia B',
        style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.white),
      ),
    ),
  ],
)
```

## SizedBox, Padding, Center

```dart
const SizedBox(width: 16);                          // fixed-size gap / box
const SizedBox(height: 8, child: SomeWidget());
const Padding(padding: EdgeInsets.all(16), child: Text('padded'));
const Center(child: Text('centered'));
```

## Common Layout Widgets

```dart
// Container, GridView, ListView, Stack,
// Scaffold, AppBar, Card, ListTile,
// CupertinoPageScaffold, CupertinoNavigationBar
```

<!-- nav -->
---

← [Flutter — Widgets & State](02-widgets.md) · [Index](README.md) · [Flutter — Constraints & Sizing](04-constraints.md) →
<!-- nav -->
