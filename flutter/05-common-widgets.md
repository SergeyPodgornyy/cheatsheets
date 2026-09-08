# Flutter: Common Widgets

*Source: https://docs.flutter.dev/ui/widgets/material*

Standard widgets (`Text`, `Row`, `Column`, `Container`, etc.) come from the widgets library and are available in any app. The specialized design widgets below come from the Material library (`import 'package:flutter/material.dart';`) and require a Material app. Cupertino widgets give iOS-style design.

## Material widgets

| Widget | Purpose |
| --- | --- |
| `Scaffold` | Basic Material layout structure; slots: `appBar`, `body`, `floatingActionButton`, `drawer`, `bottomNavigationBar` |
| `AppBar` | Top bar: `title`, content, `actions` |
| `ElevatedButton` | Raised button with a shadow, for the primary action |
| `TextButton` | Flat low-emphasis button |
| `OutlinedButton` | Bordered medium-emphasis button |
| `IconButton` | Clickable icon |
| `FloatingActionButton` | Circular button for a key action (FAB) |
| `Card` | Rounded, shadowed container for related content |
| `ListTile` | Fixed-height row: text + optional `leading`/`trailing` icons |
| `TextField` | Text input box |
| `Checkbox` | Select one/more from a set |
| `Switch` | Toggle a single on/off |
| `Radio` | Select only one option |
| `Slider` | Pick a value along a track |
| `AlertDialog` | Modal prompting for data/decision |
| `SnackBar` | Brief message at bottom |
| `BottomNavigationBar` | Bottom bar to switch primary destinations |
| `TabBar` | Horizontal tabs |
| `Drawer` | Side panel for navigation |
| `CircularProgressIndicator` | Spinning progress indicator |

## Cupertino widgets

iOS-style design. `import 'package:flutter/cupertino.dart';`

| Widget | Purpose |
| --- | --- |
| `CupertinoApp` | iOS-design app root |
| `CupertinoPageScaffold` | Basic iOS page layout (nav bar + content) |
| `CupertinoNavigationBar` | iOS top nav bar |
| `CupertinoButton` | iOS button |
| `CupertinoTabScaffold` | Tabbed iOS structure |
| `CupertinoTabBar` | iOS bottom tab bar |
| `CupertinoAlertDialog` | iOS alert dialog |
| `CupertinoActivityIndicator` | iOS spinner |
| `CupertinoSwitch` / `CupertinoSlider` | iOS toggle / slider |
| `CupertinoPicker` | iOS wheel picker |
| `CupertinoTextField` | iOS text field |

## Common widget snippets

```dart
// Text: styled via TextStyle
Text('Hello', style: TextStyle(
  fontSize: 20,
  fontWeight: FontWeight.bold,
  color: Colors.black,
))

// Icon: from the Icons set; Colors.green[500] picks a shade
Icon(Icons.star, color: Colors.green[500])

// Images
Image.asset('images/pic.jpg')      // bundled asset (declare in pubspec.yaml)
Image.network('https://...')        // remote image

// Buttons: onPressed: null disables the button
ElevatedButton(onPressed: () {}, child: const Text('Increment'))
IconButton(icon: Icon(Icons.menu), tooltip: 'Menu', onPressed: () {})

// CircleAvatar: round badge, often initials or a photo
CircleAvatar(backgroundColor: Colors.blue, child: Text('A'))

// Divider: thin horizontal rule between content
Divider()

// SizedBox: fixed-size box; also pure spacing between widgets
SizedBox(width: 16)                 // 16px gap
SizedBox(height: 210, child: ...)   // constrains child height
```

## Card

`Card` rounds + shadows its child. Give it a bounded size (e.g. via `SizedBox`).

```dart
SizedBox(
  height: 210,
  child: Card(
    child: Column(
      children: [
        ListTile(
          title: const Text('1625 Main Street',
              style: TextStyle(fontWeight: FontWeight.w500)),
          subtitle: const Text('My City, CA 99984'),
          leading: Icon(Icons.restaurant_menu, color: Colors.blue[500]),
        ),
        const Divider(),                            // separates the two tiles
        ListTile(
          title: const Text('costa@example.com'),
          leading: Icon(Icons.contact_mail, color: Colors.blue[500]),
        ),
      ],
    ),
  ),
)
```

## SnackBar via ScaffoldMessenger

Don't call it on the widget directly; use `ScaffoldMessenger.of(context)`.

```dart
ScaffoldMessenger.of(context).showSnackBar(
  const SnackBar(content: Text('Processing Data')),
);

// Replace whatever is currently showing, then show a new one (cascade ..)
ScaffoldMessenger.of(context)
  ..removeCurrentSnackBar()
  ..showSnackBar(SnackBar(content: Text('$result')));
```

## AlertDialog via showDialog

```dart
showDialog(
  context: context,
  builder: (context) => AlertDialog(
    content: Text(myController.text),
  ),
);
// Dismiss from inside the dialog with Navigator.pop(context).
```

<!-- nav -->
---

← [Flutter: Constraints & Sizing](04-constraints.md) · [Index](README.md) · [Flutter: Lists & Scrolling](06-lists-scrolling.md) →
<!-- nav -->
