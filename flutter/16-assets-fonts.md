# Flutter: Assets & Fonts

*Source: https://docs.flutter.dev/ui/assets/assets-and-images*

Assets (images, JSON, fonts) are declared in `pubspec.yaml` and bundled into the app at build time.

## Declaring assets

Indentation matters: `assets:` is two spaces under `flutter:`.

```yaml
flutter:
  assets:
    - assets/my_icon.png      # a single file
    - assets/background.png
    - directory/              # all files DIRECTLY in a dir (trailing slash)
    - directory/subdir/       # subdirs need their own entry
```

## Loading images

```dart
Image.asset('assets/background.png')              // most common
const Image(image: AssetImage('assets/bg.png'))   // AssetImage provider
Image.network('https://example.com/pic.jpg')      // remote
```

## Resolution-aware images

Provide `2.0x` / `3.0x` variants; Flutter picks one by device pixel ratio. Declare only the main asset; variants are bundled automatically.

```
.../my_icon.png         # 1.0x baseline
.../1.5x/my_icon.png
.../2.0x/my_icon.png
.../3.0x/my_icon.png
```

```yaml
flutter:
  assets:
    - assets/my_icon.png    # only the main asset needs listing
```

## Loading text / binary assets

```dart
import 'package:flutter/services.dart' show rootBundle;

Future<String> loadConfig() async {
  return await rootBundle.loadString('assets/config.json'); // text
}
// Prefer DefaultAssetBundle.of(context).loadString(...) inside a widget,
// so a parent can swap the bundle (testing / localization).
```

## Package assets

```dart
const AssetImage('icons/heart.png', package: 'my_icons'); // from a dependency
```

## Custom fonts

Declare the font family and its weight/style variants, then reference by `fontFamily`.

```yaml
flutter:
  fonts:
    - family: MyFont
      fonts:
        - asset: fonts/MyFont-Regular.ttf
        - asset: fonts/MyFont-Bold.ttf
          weight: 700
        - asset: fonts/MyFont-Italic.ttf
          style: italic
```

```dart
Text('Hello', style: TextStyle(fontFamily: 'MyFont'));

// Set app-wide via the theme:
MaterialApp(theme: ThemeData(fontFamily: 'MyFont'));
```

The `google_fonts` package fetches fonts at runtime instead of bundling them:

```dart
import 'package:google_fonts/google_fonts.dart';
Text('Hi', style: GoogleFonts.oswald(fontSize: 30));
```

<!-- nav -->
---

← [Flutter: Responsive & Adaptive Design](15-responsive-adaptive.md) · [Index](README.md) · [Home](../README.md) →
<!-- nav -->
