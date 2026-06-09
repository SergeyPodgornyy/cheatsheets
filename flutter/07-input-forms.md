# Flutter — Input & Forms

*Source: https://docs.flutter.dev/cookbook/forms/validation*

## TextField vs TextFormField

**`TextField`** is the common input box. **`TextFormField`** wraps it to integrate with a `Form` (adds `validator`, save/reset). Both show a default underline; customize with **`InputDecoration`**.

```dart
TextField(
  decoration: InputDecoration(
    border: OutlineInputBorder(),                // boxed border
    hintText: 'Enter a search term',             // placeholder, hidden once typing
  ),
)

TextFormField(
  decoration: const InputDecoration(
    border: UnderlineInputBorder(),              // default underline style
    labelText: 'Enter your username',            // floats above the field
  ),
)
```

## TextEditingController

Read or observe the field value. **Always `dispose()`** it to avoid memory leaks — so the field must live in a `StatefulWidget`.

```dart
class _MyCustomFormState extends State<MyCustomForm> {
  final myController = TextEditingController();   // 1. create

  @override
  void dispose() {
    myController.dispose();                       // 3. clean up — ALWAYS
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(controller: myController);   // 2. assign
  }
}

// Read the current value anywhere:
myController.text                                  // current String
```

### onChanged

Fires on every keystroke — lightweight alternative to a controller.

```dart
TextField(
  onChanged: (text) {
    print('value: $text');
  },
)
```

## Form + GlobalKey<FormState>

A `Form` groups fields and is driven through a **`GlobalKey<FormState>`**. Use a `StatefulWidget` so the key is created **once** (not rebuilt each `build`).

```dart
class MyCustomFormState extends State<MyCustomForm> {
  // GlobalKey<FormState> — NOT GlobalKey<MyCustomFormState>
  final _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          TextFormField(
            // validator: return an error String, or null if valid
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter some text';
              }
              return null;
            },
          ),
          Padding(
            padding: const EdgeInsets.symmetric(vertical: 16),
            child: ElevatedButton(
              onPressed: () {
                // validate() runs every field's validator; true if all pass
                if (_formKey.currentState!.validate()) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Processing Data')),
                  );
                }
              },
              child: const Text('Submit'),
            ),
          ),
        ],
      ),
    );
  }
}
```

## GestureDetector — taps

Wrap any widget to detect taps and other gestures (see basics for the full gesture set).

```dart
GestureDetector(
  onTap: () {
    print('tapped');
  },
  child: Container(color: Colors.blue, child: const Text('Tap me')),
)
```

<!-- nav -->
---

← [Flutter — Lists & Scrolling](06-lists-scrolling.md) · [Index](../README.md) · [Flutter — Navigation & Routing](08-navigation.md) →
<!-- nav -->
