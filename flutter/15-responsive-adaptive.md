# Flutter — Responsive & Adaptive Design

*Source: https://docs.flutter.dev/ui/adaptive-responsive/best-practices*

**Responsive** = layout adjusts to available *space*. **Adaptive** = behavior/widgets adjust to the *platform* (Material vs Cupertino, touch vs mouse). Both react to **window size**, not device type.

## Get the available size

```dart
final Size size = MediaQuery.sizeOf(context); // window size — prefer sizeOf
size.width; size.height;
```

```dart
// GOTCHA: MediaQuery.of(context) rebuilds on ANY MediaQuery change.
//   sizeOf(context) only rebuilds when the SIZE changes — cheaper.
```

## LayoutBuilder — switch layout by breakpoint

Reacts to the **parent's** constraints (the actual space the widget gets), not the whole screen.

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

```dart
// ✅ Use MediaQuery.sizeOf / LayoutBuilder + breakpoints for layout decisions.
// ✅ Break UI into small `const` widgets — Flutter reuses them, faster rebuilds.
// ✅ Solve touch first, then layer on mouse/keyboard accelerators.
// ✅ Use PageStorageKey to preserve scroll position across rotation/rebuilds.
//
// ❌ Don't switch layouts on ORIENTATION (OrientationBuilder near the top) —
//      orientation doesn't tell you how much space the window has.
// ❌ Don't check "is this a phone or tablet?" — apps run in resizable windows,
//      split-screen, picture-in-picture. Check the WINDOW SIZE instead.
// ❌ Don't lock orientation (if you truly must, use the Display API, not MediaQuery).
```

```dart
// Preserve scroll position across rebuilds / rotation:
SingleChildScrollView(
  key: const PageStorageKey('myList'),
  child: /* ... */,
)
```

<!-- nav -->
---

← [Flutter — Gestures](14-gestures.md) · [Index](README.md) · [Flutter — Assets & Fonts](16-assets-fonts.md) →
<!-- nav -->
