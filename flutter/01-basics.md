# Flutter — Basics

*Source: https://docs.flutter.dev/ui/widgets-intro*

## Everything Is a Widget

Flutter UI is built from **widgets** (framework inspired by **React**). A widget describes its view given its current **config + state**. On state change, the widget rebuilds; the framework **diffs** the new description against the previous one and applies the **minimal changes** to the underlying render tree.

```dart
// You author widgets as subclasses of StatelessWidget or StatefulWidget.
// A widget's main job is build() — describing itself via lower-level widgets,
// down to RenderObject (handles geometry/painting).
```

The **widget tree** is the composition: widgets nest other widgets via `child` / `children`.

## runApp — Minimal Hello World

`runApp` makes the given widget the **root** of the tree. The root is forced to **fill the screen**.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    const Center(
      child: Text(
        'Hello, world!',
        textDirection: TextDirection.ltr, // required without MaterialApp —
                                          // no ambient Directionality otherwise
        style: TextStyle(color: Colors.blue),
      ),
    ),
  );
}
```

## MaterialApp + Scaffold

`MaterialApp` adds a **Navigator** and **theme**. `Scaffold` lays out the major Material components (app bar, body, FAB).

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MaterialApp(title: 'Flutter Tutorial', home: TutorialHome()));
}

class TutorialHome extends StatelessWidget {
  const TutorialHome({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: const IconButton(
          icon: Icon(Icons.menu),
          tooltip: 'Navigation menu',
          onPressed: null, // null disables the button (greyed out)
        ),
        title: const Text('Example title'),
        actions: const [
          IconButton(icon: Icon(Icons.search), tooltip: 'Search', onPressed: null),
        ],
      ),
      body: const Center(child: Text('Hello, world!')),
      floatingActionButton: const FloatingActionButton(
        tooltip: 'Add',
        onPressed: null,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

## pubspec — uses-material-design

Enable the bundled Material icon font so `Icons.*` glyphs render.

```yaml
name: my_app
flutter:
  uses-material-design: true
```

## Material vs Cupertino

Two bundled design languages.

```dart
// Material (Android-style):  MaterialApp, Scaffold, AppBar
// Cupertino (iOS-style):     CupertinoApp, CupertinoNavigationBar
```

## Project Structure & Running

```console
$ flutter create my_app   # scaffolds the project
$ flutter run             # builds + launches on a connected device/emulator
```

```dart
// lib/main.dart is the entry point:
void main() => runApp(const MyApp());
```

## Hot Reload vs Hot Restart

**Debug mode only.** Both speed up iteration; they differ in what they preserve.

```console
$ # in the `flutter run` terminal:
$ # r  → hot reload   (injects updated source into the running Dart VM,
$ #                     rebuilds widget tree, PRESERVES app state)
$ # R  → hot restart  (reloads changes and RESETS state; reruns from scratch)
```

```dart
// Hot reload does NOT rerun main() or initState().
// Hot restart re-runs them.
```

### Hot reload limitations (need a full restart)

```dart
// - Changes to main() or initState() are NOT re-executed.
// - enum <-> class conversions.
// - Changed generic type parameters.
// - Global / static field initializers are not re-run.
//     Prefer `const` or a getter instead of a `final` static initializer.
// - Native (platform) code changes need a full restart.
```

<!-- nav -->
---

← [Home](../README.md) · [Index](README.md) · [Flutter — Widgets & State](02-widgets.md) →
<!-- nav -->
