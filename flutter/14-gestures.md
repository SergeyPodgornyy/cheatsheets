# Flutter: Gestures

*Source: https://docs.flutter.dev/ui/interactivity/gestures*

Two layers: pointers (raw touch/mouse/stylus events, i.e. location and movement, listened via `Listener`) and gestures (semantic actions like tap/drag/scale, recognized from pointer events, listened via `GestureDetector`).

## GestureDetector

Wrap any widget; set only the callbacks you need (which callbacks are non-null decides which gestures are attempted).

```dart
GestureDetector(
  onTap: () => print('tap'),
  onDoubleTap: () => print('double tap'),
  onLongPress: () => print('long press'),
  child: Container(color: Colors.blue, child: const Text('Tap me')),
)
```

### Tap callbacks

- `onTapDown`: a pointer that might tap contacted the screen.
- `onTapUp`: the pointer that triggers a tap lifted.
- `onTap`: a down followed by an up, so a tap happened.
- `onTapCancel`: the down won't become a tap.

### Drag callbacks

```dart
GestureDetector(
  onVerticalDragUpdate: (details) => print(details.delta.dy),   // vertical drag
  onHorizontalDragUpdate: (details) => print(details.delta.dx), // horizontal drag
  // Pan = both axes. DON'T mix pan with vertical/horizontal, it crashes.
  onPanUpdate: (details) => print(details.delta), // Offset moved since last event
  child: const FlutterLogo(size: 200),
)
```

Each drag has a start, an update, and an end:

- `onPanStart` gives `details.globalPosition`.
- `onPanUpdate` gives `details.delta`, the movement since the last callback.
- `onPanEnd` gives `details.velocity`.

## InkWell: Material ripple

For the Material "ink splash" on tap (use instead of `GestureDetector` when you want the ripple). Needs a `Material` ancestor (Scaffold provides one).

```dart
InkWell(
  onTap: () => print('tapped'),
  child: const Padding(padding: EdgeInsets.all(12), child: Text('Tap')),
)
```

Many Material widgets already handle gestures: `IconButton`/`TextButton` respond to taps, `ListView` to swipes.

## Dismissible: swipe to dismiss

Swipe a list item away. Each must have a unique `key`; `onDismissed` fires after the swipe.

```dart
Dismissible(
  key: Key(item),                          // unique key required
  onDismissed: (direction) {
    setState(() => items.removeAt(index)); // remove from data source
    ScaffoldMessenger.of(context)
        .showSnackBar(SnackBar(content: Text('$item dismissed')));
  },
  background: Container(color: Colors.red), // "leave behind" shown while swiping
  child: ListTile(title: Text(item)),
)
```

## Gesture arena (disambiguation)

When multiple recognizers compete for the same pointer, the framework runs a **gesture arena**: a recognizer can eliminate itself (leaving) or declare itself the winner (forcing others to lose). E.g. with horizontal vs vertical drag, whichever direction passes the movement threshold first wins. A lone recognizer wins immediately on the first pixel.

<!-- nav -->
---

← [Flutter: Slivers & Advanced Scrolling](13-slivers-scrolling.md) · [Index](README.md) · [Flutter: Responsive & Adaptive Design](15-responsive-adaptive.md) →
<!-- nav -->
