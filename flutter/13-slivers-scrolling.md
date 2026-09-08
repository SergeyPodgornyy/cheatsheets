# Flutter: Slivers & Advanced Scrolling

*Source: https://docs.flutter.dev/cookbook/lists/floating-app-bar*

A **sliver** is a portion of a scrollable area with custom scroll behavior. `ListView` and `GridView` are built on `SliverList` / `SliverGrid`. For custom scroll effects (collapsing headers, mixed list+grid), compose slivers inside a `CustomScrollView`.

## CustomScrollView

Hosts a list of slivers in one shared scroll view.

```dart
CustomScrollView(
  slivers: <Widget>[
    // sliver widgets go here, they scroll together
  ],
)
```

## SliverAppBar: collapsing / floating header

```dart
CustomScrollView(
  slivers: [
    const SliverAppBar(
      title: Text('Floating App Bar'),
      pinned: true,            // stays visible (pinned) when scrolled up
      flexibleSpace: Placeholder(), // fills the expanded area (e.g. an image)
      expandedHeight: 200,     // initial height before it shrinks
    ),
    SliverList.builder(
      itemBuilder: (context, index) => ListTile(title: Text('Item #$index')),
      itemCount: 50,
    ),
  ],
)
```

Header behavior flags:

- `pinned: true` remains on screen, shrunk, when scrolled past.
- `floating: true` reappears immediately on any upward scroll.
- `snap: true` animates fully in or out (with `floating`).

## Sliver list & grid

```dart
SliverList.builder(                    // lazy list as a sliver
  itemBuilder: (context, index) => ListTile(title: Text('Item #$index')),
  itemCount: 50,
)

SliverGrid.count(                      // grid as a sliver
  crossAxisCount: 2,
  children: [ /* ... */ ],
)
```

## Other common slivers

```dart
// Embed a single non-sliver (box) widget in a CustomScrollView:
SliverToBoxAdapter(child: MyHeaderCard())

// Fill the remaining viewport space (e.g. an empty-state):
SliverFillRemaining(child: Center(child: Text('No more items')))

// A sticky custom header you control:
SliverPersistentHeader(pinned: true, delegate: MyHeaderDelegate())
```

## Cupertino equivalent

```dart
CupertinoApp(
  home: CupertinoPageScaffold(
    child: CustomScrollView(
      slivers: [
        const CupertinoSliverNavigationBar(largeTitle: Text('Items')),
        SliverList.builder(
          itemBuilder: (context, index) =>
              CupertinoListTile(title: Text('Item #$index')),
          itemCount: 50,
        ),
      ],
    ),
  ),
)
```

<!-- nav -->
---

← [Flutter: Testing](12-testing.md) · [Index](README.md) · [Flutter: Gestures](14-gestures.md) →
<!-- nav -->
