---
name: features-auto-dispose
description: Automatic disposal of providers when no longer used — enabling/disabling, when disposal triggers, keepAlive for fine-tuned control, cacheFor extension pattern
---

# Automatic Disposal

Riverpod can automatically destroy provider state when it's no longer used (no listeners).

## Enabling/Disabling

**With code generation** — enabled by default:

```dart
// Disable auto-dispose
@Riverpod(keepAlive: true)
String helloWorld(Ref ref) => 'Hello world!';
```

**Without code generation** — opt-in:

```dart
final helloWorldProvider = Provider<String>(
  isAutoDispose: true,
  (ref) => 'Hello world!',
);
```

## When Disposal Triggers

1. Riverpod tracks `ref.watch`/`ref.listen` calls.
2. When listener count reaches zero, `ref.onCancel` fires.
3. After one frame (`await null`), if still unused, provider is destroyed and `ref.onDispose` fires.

Always enable auto-dispose for family providers to avoid memory leaks (one state per parameter combination).

## Reacting to Disposal

Use `ref.onDispose` to clean up resources:

```dart
@riverpod
Stream<int> myStream(Ref ref) {
  final controller = StreamController<int>();
  ref.onDispose(controller.close);
  return controller.stream;
}
```

Other lifecycle hooks: `ref.onCancel` (last listener removed), `ref.onResume` (new listener after cancel).

## Fine-Tuning with ref.keepAlive

Keep state alive conditionally (e.g., cache successful requests but not failures):

```dart
@riverpod
Future<Data> fetchData(Ref ref) async {
  final link = ref.keepAlive();
  try {
    final data = await api.fetch();
    return data; // keepAlive stays active — state persists
  } catch (e) {
    link.close(); // revert to auto-dispose on failure
    rethrow;
  }
}
```

`ref.keepAlive` returns a `KeepAliveLink`. Call `link.close()` to re-enable auto-dispose. Recomputation resets keepAlive.

## Cache for Duration Pattern

Keep state alive for a specific time using a reusable extension:

```dart
extension CacheForExtension on Ref {
  void cacheFor(Duration duration) {
    final link = keepAlive();
    final timer = Timer(duration, link.close);
    onDispose(timer.cancel);
  }
}

// Usage
@riverpod
Future<Data> fetchData(Ref ref) async {
  ref.cacheFor(const Duration(minutes: 5));
  return api.fetch();
}
```

## ref.invalidate

Force-destroy a provider, even if it has listeners:

```dart
ref.invalidate(myProvider);
// If listened to: new state is created immediately
// If not listened to: fully destroyed
```

For family providers, invalidate specific or all combinations:

```dart
ref.invalidate(userProvider('123'));   // specific
ref.invalidate(userProvider);          // all combinations
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/auto_dispose
-->
