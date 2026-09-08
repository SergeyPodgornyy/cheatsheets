# Flutter: Constraints & Sizing

*Source: https://docs.flutter.dev/ui/layout/constraints*

## THE RULE

> Constraints go down. Sizes go up. Parent sets position.

### The 4-step process

1. A widget gets constraints from its parent: four doubles, min/max width and min/max height.
2. It passes constraints down to its children, asking each its size.
3. It positions its children (x, y) relative to itself.
4. It reports its own size up to the parent, within the original constraints.

### Limitations

- A widget chooses its size only within the constraints its parent gave it.
- A widget can't decide its own position on screen; its parent decides.
- A child's size may be ignored if the parent has no alignment info.

## Tight vs loose

- A tight constraint means the child must be a specific size, with no choice.
- A loose constraint means the child can be anything up to a max, and may be smaller.

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

`Center` and `Scaffold` loosen the constraints they pass down; `SizedBox.expand` tightens them to fill.

## Three kinds of boxes

1. As big as possible: `Center`, `ListView`
2. As big as their child: `Transform`, `Opacity`
3. A particular size: `Image`, `Text`

## Why `width: 100` gets ignored

`Container` defaults to as-big-as-possible, but honors `width`/`height` if the incoming constraints allow it.

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

Adds extra constraints on top of what it receives, and never overrides a tight parent.

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

## Summary table

| Widget | Effect on constraints / sizing |
|---|---|
| Screen (root) | passes tight constraints: the child fills the device |
| Center | loosens, then centers the child |
| Align | loosens, then positions the child (e.g. `bottomRight`) |
| SizedBox.expand | tightens: the child is forced to fill |
| ConstrainedBox | adds extra constraints, always applied (within the parent's) |
| LimitedBox | limits only when the incoming constraint is infinite |
| UnconstrainedBox | imposes no constraints (child may overflow → warns) |
| FittedBox | loosens, then scales the (bounded) child to fit |

<!-- nav -->
---

← [Flutter: Layout](03-layout.md) · [Index](README.md) · [Flutter: Common Widgets](05-common-widgets.md) →
<!-- nav -->
