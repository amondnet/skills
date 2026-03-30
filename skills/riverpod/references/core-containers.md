---
name: core-containers
description: ProviderContainer and ProviderScope — storing provider state, Flutter vs pure Dart usage, testing containers, why state is not stored in providers directly
---

# ProviderContainer & ProviderScope

ProviderContainer is the central piece that stores all provider state. Providers themselves hold no state — all state lives in the container.

## Flutter Apps — ProviderScope

Wrap your app with `ProviderScope` (creates a ProviderContainer internally):

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

## Pure Dart — ProviderContainer

For CLI apps, server-side, or non-Flutter Dart:

```dart
void main() {
  final container = ProviderContainer();
  try {
    final sub = container.listen(counterProvider, (previous, next) {
      print('Counter changed from $previous to $next');
    });
    print('Counter starts at ${sub.read()}');
  } finally {
    container.dispose();
  }
}
```

## Testing — ProviderContainer.test

Auto-disposes after the test ends:

```dart
test('Counter starts at 0', () {
  final container = ProviderContainer.test();
  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);
});
```

## Why External State Storage?

1. **Separation of concerns** — State updates are centralized in providers.
2. **Better testing** — Each test gets a fresh container with clean state.
3. **Centralized configuration** — Observers, overrides, retry logic all configured on the container.
4. **Scoping support** — Same provider can resolve to different state in different parts of the widget tree.

<!--
Source references:
- https://riverpod.dev/docs/concepts2/containers
-->
