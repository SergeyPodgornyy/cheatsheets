# Dart — Async (Future, Stream, Isolate)

*Source: https://dart.dev/libraries/async*

## Event loop / single-thread model

Dart code runs in an **isolate** on a **single thread** with an event loop processing a FIFO queue. `async` APIs give **interleaved concurrency** on that one thread — not parallelism.

```dart
while (eventQueue.waitForEvent()) {   // conceptual event loop
  eventQueue.processNextEvent();
}
```

## Future

A **Future\<T>** is the result of an async op. States: *uncompleted*, or *completed* with a value or an error. No value → `Future<void>`.

```dart
Future.delayed(const Duration(seconds: 2), () => 'Large Latte'); // completes later
Future.value(42);                  // already completed with a value
Future<void> f = doWork();         // completes with no value

// Callback style (.then / .catchError / .whenComplete):
http.get('https://example.com').then((response) {
  if (response.statusCode == 200) print('Success!');
}).catchError((e) {
  print('Failed: $e');
}).whenComplete(() => print('done'));  // runs on value OR error (like finally)

// Future-returning helper:
Future<String> _readFileAsync(String filename) {
  final file = File(filename);
  return file.readAsString().then((contents) => contents.trim());
}

// Wait for several at once:
await Future.wait([fetchA(), fetchB(), fetchC()]); // completes when all do
```

Complete with an error:

```dart
return Future.delayed(const Duration(seconds: 2),
    () => throw Exception('Logout failed: user ID is invalid'));
```

## async / await

`async` before the body makes an **async function** (returns `Future<T>`/`Future<void>`). `await` is allowed **only inside** an async function. An async fn runs **synchronously until the first `await`**.

```dart
void main() async { ··· }            // both forms are valid
Future<void> main() async { ··· }
```

### WRONG vs RIGHT

```dart
// WRONG — no await: you get the Future object, not its value.
String createOrderMessage() {
  var order = fetchUserOrder();
  return 'Your order is: $order';
}
Future<String> fetchUserOrder() =>
    Future.delayed(const Duration(seconds: 2), () => 'Large Latte');
// Prints: Your order is: Instance of 'Future<String>'
```

```dart
// RIGHT — await the Future inside an async fn.
Future<String> createOrderMessage() async {
  var order = await fetchUserOrder();
  return 'Your order is: $order';
}
Future<String> fetchUserOrder() =>
    Future.delayed(const Duration(seconds: 2), () => 'Large Latte');

Future<void> main() async {
  print('Fetching user order...');
  print(await createOrderMessage());
}
// Fetching user order...
// Your order is: Large Latte
```

Fire-and-forget (no `await`) runs the rest of the function first:

```dart
void main() {
  fetchUserOrder();                  // started, not awaited
  print('Fetching user order...');   // prints before the order resolves
}
```

### try-catch around await

`await` rethrows the Future's error synchronously, so wrap it in a normal `try-catch`.

```dart
Future<void> printOrderMessage() async {
  try {
    print('Awaiting user order...');
    var order = await fetchUserOrder();
    print(order);
  } catch (err) {
    print('Caught error: $err');
  }
}

Future<String> changeUsername() async {
  try {
    return await fetchNewUsername();
  } catch (err) {
    return err.toString();
  }
}
```

## Stream

A **Stream\<T>** is an async sequence of events — an async `Iterable`. Pairs with `Future` (one value) for many values over time.

### await for & async* / yield

`async*` makes a generator that `yield`s values; `await for` consumes them (the enclosing fn must be `async`, and the loop ends when the stream is done).

```dart
Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    yield i;                       // emit each value
  }
}

Future<int> sumStream(Stream<int> stream) async {
  var sum = 0;
  await for (final value in stream) {   // consume until done
    sum += value;
  }
  return sum;
}

void main() async {
  var stream = countStream(10);
  var sum = await sumStream(stream);
  print(sum);                      // 55
}
```

### Error handling

Wrap `await for` in `try-catch`. Most streams **stop after the first error**.

```dart
Future<int> sumStream(Stream<int> stream) async {
  var sum = 0;
  try {
    await for (final value in stream) {
      sum += value;
    }
  } catch (e) {
    return -1;
  }
  return sum;
}
```

### Single vs broadcast

```dart
// single-subscription: exactly one listener; e.g. file I/O, a web request.
// broadcast:           many listeners; e.g. UI / mouse events.
```

### Transforms (return new streams)

```dart
stream.map((x) => x * 2);          // transform each event
stream.where((x) => x.isEven);     // filter
stream.asyncMap((x) => fetch(x));  // async transform, preserves order
// also: expand, take, skip, distinct, handleError, timeout, transform
stream.lastWhere((x) => x >= 0);   // + first / last / length / toList /
                                   //   forEach / reduce / fold / join
```

### listen() — low-level

Returns a `StreamSubscription` (`pause`/`resume`/`cancel`).

```dart
StreamSubscription<T> listen(
  void Function(T event)? onData, {
  Function? onError,
  void Function()? onDone,
  bool? cancelOnError,
});
```

### StreamController

Programmatically create a stream and push events into it via `controller.sink.add(...)`; expose `controller.stream` to consumers.

## Isolates

Each isolate has its **own memory** and a single-thread event loop. They communicate by **message passing** — no shared mutable state, so **no locks/mutexes and no data races** (Actor model). `async`/`await` is *not* parallel; isolates *are* (across CPU cores).

```dart
// Global mutable state is a SEPARATE COPY per isolate.
// Web targets use web workers instead of isolates.
```

### Isolate.run — one-off background work (recommended)

```dart
int slowFib(int n) => n <= 1 ? 1 : slowFib(n - 1) + slowFib(n - 2);

void fib40() async {
  var result = await Isolate.run(() => slowFib(40)); // runs off the main thread
  print('Fib(40) = $result');
}
```

### Isolate.spawn — long-lived worker

Use a long-lived isolate handling many messages over time, with `SendPort`/`ReceivePort` to pass data back and forth.

```dart
final receivePort = ReceivePort();
await Isolate.spawn(worker, receivePort.sendPort); // worker handles messages
receivePort.listen((message) { /* results arrive here */ });
```

<!-- nav -->
---

← [Dart — Error Handling](10-error-handling.md) · [Index](../README.md) · [Dart — Libraries & Packages](12-libraries-packages.md) →
<!-- nav -->
