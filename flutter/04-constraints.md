# Flutter — Constraints & Sizing

*Source: https://docs.flutter.dev/ui/layout/constraints*

## THE RULE

> **Constraints go down. Sizes go up. Parent sets position.**

### The 4-step process

```dart
// 1. A widget gets CONSTRAINTS from its parent:
//      4 doubles -> min/max width, min/max height.
// 2. It passes constraints DOWN to its children, asking each its size.
// 3. It POSITIONS its children (x, y) relative to itself.
// 4. It reports its OWN size UP to the parent (within the original constraints).
```

### Limitations

```dart
// - A widget chooses its size ONLY within the constraints its parent gave it.
// - A widget CAN'T decide its own position on screen — its parent decides.
// - A child's size may be IGNORED if the parent has no alignment info.
```

## Tight vs Loose

```dart
// Tight constraint -> child MUST be a specific size (no choice).
// Loose constraint -> child CAN be anything up to a max (may be smaller).
```

```dart
// Loose: Container is free to be only as big as its content
Scaffold(
  body: Container(
    color: Colors.blue,
    child: const Column(children: [Text('Hello!'), Text('Goodbye!')]),
  ),
)

// Tight: SizedBox.expand forces the Container to FILL the body
Scaffold(
  body: SizedBox.expand(
    child: Container(color: Colors.blue, child: const Column(/* ... */)),
  ),
)
```

```dart
// Center / Scaffold  -> loosen the constraints they pass down.
// SizedBox.expand     -> tightens to fill.
```

## Three Kinds of Boxes

```dart
// 1. As big as possible      -> Center, ListView
// 2. As big as their child   -> Transform, Opacity
// 3. A particular size       -> Image, Text
```

## Why `width: 100` Gets Ignored

`Container` defaults to **as-big-as-possible**, but honors `width`/`height` **if the incoming constraints allow it**.

```dart
// Screen passes TIGHT constraints -> width/height ignored, fills the screen:
Container(width: 100, height: 100, color: Colors.red)

// Center LOOSENS the constraints -> Container becomes exactly 100x100, centered:
Center(child: Container(width: 100, height: 100, color: Colors.red))

// Align loosens AND positions -> 100x100, pinned to the bottom-right:
Align(
  alignment: Alignment.bottomRight,
  child: Container(width: 100, height: 100, color: Colors.red),
)
```

## ConstrainedBox

Adds **additional** constraints on top of what it receives — never overrides a tight parent.

```dart
// Under a tight parent these extra constraints are IGNORED.
ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 70, minHeight: 70,
    maxWidth: 150, maxHeight: 150,
  ),
  child: Container(color: Colors.red, width: 10, height: 10),
  // child wants 10x10 but minWidth/minHeight force it up to >= 70x70
)
```

## Summary Table

| Widget | Effect on constraints / sizing |
|---|---|
| **Screen** (root) | passes **tight** constraints — fills the device |
| **Center** | **loosens**, then centers the child |
| **Align** | **loosens**, then positions the child (e.g. `bottomRight`) |
| **SizedBox.expand** | **tightens** — child forced to fill |
| **ConstrainedBox** | adds **extra constraints**, always applied (within parent's) |
| **LimitedBox** | limits **only when** the incoming constraint is infinite |
| **UnconstrainedBox** | imposes **no** constraints (child may overflow → warns) |
| **FittedBox** | **loosens**, then **scales** the (bounded) child to fit |

<!-- nav -->
---

← [Flutter — Layout](03-layout.md) · [Index](README.md) · [Flutter — Common Widgets](05-common-widgets.md) →
<!-- nav -->
