# Flutter: Responsive & Adaptive Design

*Source: https://docs.flutter.dev/ui/adaptive-responsive/best-practices*

**Responsive** = layout adjusts to available *space*. **Adaptive** = behavior/widgets adjust to the *platform* (Material vs Cupertino, touch vs mouse). Both react to window size, not device type.

## Get the available size

```dart
final Size size = MediaQuery.sizeOf(context); // window size; prefer sizeOf
size.width; size.height;
```

**Gotcha:** `MediaQuery.of(context)` rebuilds on any `MediaQuery` change. `sizeOf(context)` only rebuilds when the size changes, which is cheaper.

## LayoutBuilder: switch layout by breakpoint

Reacts to the parent's constraints (the actual space the widget gets), not the whole screen.

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth >= 840) {
      return ExpandedLayout();   // large / desktop
    } else if (constraints.maxWidth >= 600) {
      return MediumLayout();     // tablet
    } else {
      return CompactLayout();    // phone
    }
  },
)
// Use Material window-size-class breakpoints (compact <600, medium 600-839, expanded 840+).
```

## SafeArea

Insets content past notches, status bars, and rounded corners.

```dart
SafeArea(child: MyContent())
```

## Best practices (from the docs)

- ✅ Use `MediaQuery.sizeOf` or `LayoutBuilder` with breakpoints for layout decisions.
- ✅ Break the UI into small `const` widgets; Flutter reuses them, so rebuilds are faster.
- ✅ Solve touch first, then layer on mouse and keyboard accelerators.
- ✅ Use `PageStorageKey` to preserve scroll position across rotation and rebuilds.
- ❌ Don't switch layouts on orientation (an `OrientationBuilder` near the top): orientation doesn't tell you how much space the window has.
- ❌ Don't check "is this a phone or tablet?" Apps run in resizable windows, split-screen, and picture-in-picture. Check the window size instead.
- ❌ Don't lock orientation. If you truly must, use the Display API, not `MediaQuery`.

```dart
// Preserve scroll position across rebuilds / rotation:
SingleChildScrollView(
  key: const PageStorageKey('myList'),
  child: /* ... */,
)
```

<!-- nav -->
---

← [Flutter: Gestures](14-gestures.md) · [Index](README.md) · [Flutter: Assets & Fonts](16-assets-fonts.md) →
<!-- nav -->
