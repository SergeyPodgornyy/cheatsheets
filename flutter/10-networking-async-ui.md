# Flutter: Networking & Async UI

*Source: https://docs.flutter.dev/cookbook/networking/fetch-data*

## The `http` package

Simplest way to fetch data. Avoid using `dart:io` / `dart:html` directly.

```console
$ flutter pub add http
```

```dart
import 'package:http/http.dart' as http; // import with a prefix
```

Android requires the INTERNET permission in `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Making a request

`http.get` returns a `Future<Response>`. `Response` has `.statusCode` and `.body`.

```dart
Future<Album> fetchAlbum() async {
  final response = await http.get(
    Uri.parse('https://jsonplaceholder.typicode.com/albums/1'),
    headers: {'Accept': 'application/json'},
  );

  if (response.statusCode == 200) {
    // jsonDecode returns dynamic; cast to the expected shape.
    return Album.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
  } else {
    throw Exception('Failed to load album'); // on non-200, THROW; don't return null
  }
}
```

## JSON model with `fromJson`

Define a model and a factory constructor to parse a decoded map. The body uses a `switch` with a map pattern to validate shape and bind fields.

```dart
class Album {
  final int userId;
  final int id;
  final String title;

  const Album({required this.userId, required this.id, required this.title});

  factory Album.fromJson(Map<String, dynamic> json) {
    return switch (json) {
      // map pattern: matches keys AND checks each value's type
      {'userId': int userId, 'id': int id, 'title': String title} =>
        Album(userId: userId, id: id, title: title),
      _ => throw const FormatException('Failed to load album.'), // shape mismatch
    };
  }
}
```

### `dart:convert`

```dart
import 'dart:convert';

jsonDecode(response.body); // JSON string -> Dart Map/List (dynamic)
jsonEncode(obj);           // Dart object -> JSON string (to send in a request body)
```

## Fetching in `initState`

```dart
class _MyAppState extends State<MyApp> {
  late Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum(); // GOTCHA: fetch here, NOT in build()
  }
}
```

Why not `build()`? It runs on every rebuild, so fetching there would spam the network. `initState` runs once, when the `State` is created.

## FutureBuilder

Rebuilds itself based on the latest snapshot of a `Future`.

```dart
FutureBuilder<Album>(
  future: futureAlbum, // the Future created in initState
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text(snapshot.data!.title); // resolved successfully
    } else if (snapshot.hasError) {
      return Text('${snapshot.error}');  // future threw
    }
    return const CircularProgressIndicator(); // still loading
  },
)
```

## StreamBuilder

Same pattern, for a continuous source of data (`Stream`). Rebuilds on every emitted event.

```dart
StreamBuilder<int>(
  stream: counterStream, // emits values over time
  builder: (context, snapshot) {
    if (snapshot.hasError) return Text('${snapshot.error}');
    if (snapshot.hasData) return Text('${snapshot.data}');
    return const CircularProgressIndicator();
  },
)
```

<!-- nav -->
---

← [Flutter: State Management](09-state-management.md) · [Index](README.md) · [Flutter: Theming & Animations](11-theming-animations.md) →
<!-- nav -->
