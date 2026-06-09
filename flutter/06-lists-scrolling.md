# Flutter — Lists & Scrolling

*Source: https://docs.flutter.dev/cookbook/lists/long-lists*

## ListView — static children

A **`ListView`** is a scrolling `Column`: it scrolls automatically when content overflows, vertically (default) or horizontally.

```dart
ListView(
  padding: const EdgeInsets.symmetric(vertical: 8),
  children: [
    _tile('CineArts at the Empire', '85 W Portal Ave', Icons.theaters),
    const Divider(),                                  // mix any widget in
    _tile("K's Kitchen", '757 Monterey Blvd', Icons.restaurant),
  ],
)

// Helper that builds a ListTile — keeps children list readable
ListTile _tile(String title, String subtitle, IconData icon) {
  return ListTile(
    title: Text(title,
        style: const TextStyle(fontWeight: FontWeight.w500, fontSize: 20)),
    subtitle: Text(subtitle),
    leading: Icon(icon, color: Colors.blue[500]),
  );
}
```

## ListView.builder — long / infinite lists

**Gotcha:** the default `ListView(children: [...])` builds **every** child up front — fine for a handful, wasteful for thousands. `ListView.builder` builds items **lazily**, only as they scroll into view.

```dart
ListView.builder(
  itemCount: items.length,                  // omit for an infinite list
  itemBuilder: (context, index) {           // called per visible item
    return ListTile(title: Text(items[index]));
  },
)
```

### Data source

```dart
// Generate a large in-memory list for testing
final items = List<String>.generate(10000, (i) => 'Item $i');
```

## Children extent (performance)

By default each item must be laid out to learn its size. Telling the list the extent up front lets it skip that work and scroll smoother.

```dart
// Option 1 — prototypeItem: list measures this once, applies to all
ListView.builder(
  itemCount: items.length,
  prototypeItem: ListTile(title: Text(items.first)),
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
)

// Option 2 — itemExtent: fixed pixel size, every item identical
ListView.builder(itemExtent: 56, itemCount: items.length, itemBuilder: ...)

// Option 3 — itemExtentBuilder: variable per-index size
ListView.builder(
  itemExtentBuilder: (index, dimensions) => index.isEven ? 56 : 80,
  itemCount: items.length,
  itemBuilder: ...,
)
```

## GridView — 2D scrollable grid

```dart
// .extent — set a MAX tile width; column count derived from available space
GridView.extent(
  maxCrossAxisExtent: 150,
  padding: const EdgeInsets.all(4),
  mainAxisSpacing: 4,                                  // gap between rows
  crossAxisSpacing: 4,                                 // gap between columns
  children: List.generate(30, (i) => Image.asset('images/pic$i.jpg')),
)

// .count — fixed number of columns
GridView.count(
  crossAxisCount: 2,
  children: [...],
)
```

<!-- nav -->
---

← [Flutter — Common Widgets](05-common-widgets.md) · [Index](../README.md) · [Flutter — Input & Forms](07-input-forms.md) →
<!-- nav -->
