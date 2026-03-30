---
name: core-refs
description: Ref API for interacting with providers — watch, listen, read, invalidate, refresh, lifecycle hooks (onDispose, onCancel, onResume), and mounted check
---

# Refs

Ref is the primary way to interact with providers. Similar to `BuildContext` for widgets, but for providers.

## Obtaining a Ref

- **In providers**: First parameter of the function, or `this.ref` in Notifiers.
- **In widgets**: Use Consumer/ConsumerWidget/ConsumerStatefulWidget to get a `WidgetRef`.
- **Elsewhere**: Pass ref from widget/provider to your function.

```dart
@riverpod
int myProvider(Ref ref) {
  // ref available here
}

@riverpod
class MyNotifier extends _$MyNotifier {
  @override
  int build() {
    ref.watch(someProvider); // this.ref available in notifiers
    return 0;
  }
}
```

## Listening to State

### ref.watch — Declarative (preferred)

Subscribes to a provider. Rebuilds when the provider changes. Use in `build` methods.

```dart
// In a provider
final tick = ref.watch(tickProvider);

// In a widget
Consumer(
  builder: (context, ref, _) {
    final tick = ref.watch(tickProvider);
    return Text('Tick: $tick');
  },
);
```

Providers can watch other providers to create derived/combined state:

```dart
@riverpod
bool isDivisibleBy4(Ref ref) {
  return ref.watch(tickProvider) % 4 == 0;
}
```

### ref.listen — Imperative (side-effects)

Fires a callback on each change. Use for navigation, dialogs, logging.

```dart
ref.listen(tickProvider, (previous, next) {
  print('Tick changed from $previous to $next');
});
```

In widgets, use `ref.listen` inside `build`. For outside `build`, use `ref.listenManual`.

### ref.read — One-shot (event handlers)

Read current value without subscribing. Use only in callbacks like `onPressed`.

```dart
onPressed: () {
  final tick = ref.read(tickProvider);
  print('Current tick: $tick');
}
```

**Never use `ref.read` to "optimize"** — use `ref.watch` or `select` instead.

## Modifying State

### ref.invalidate — Reset provider

Discards current state. Re-evaluates next time it's read.

```dart
ref.invalidate(tickProvider);
```

### ref.refresh — Reset + read

Shorthand for `invalidate` + `read`:

```dart
final newTick = ref.refresh(tickProvider);
```

### ref.mounted (Riverpod 3.0)

Check if provider is still alive after async operations:

```dart
Future<void> addTodo(String title) async {
  final newTodo = await api.addTodo(title);
  if (!ref.mounted) return;
  state = [...state, newTodo];
}
```

## Lifecycle Hooks

Register callbacks for provider lifecycle events. No need to unregister — cleaned up automatically.

```dart
@riverpod
int counter(Ref ref) {
  ref.onDispose(() => print('disposed'));
  ref.onCancel(() => print('no listeners'));
  ref.onResume(() => print('listener added after cancel'));
  return 0;
}
```

- `onDispose`: Called when state is destroyed (auto-dispose or recomputation).
- `onCancel`: Called when last listener is removed.
- `onResume`: Called when a new listener is added after `onCancel`.

Listeners return an unregister function if manual removal is needed:

```dart
final unregister = ref.onDispose(() => print('disposed'));
unregister(); // manually remove
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/refs
-->
