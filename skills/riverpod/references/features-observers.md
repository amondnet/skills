---
name: features-observers
description: ProviderObserver for logging, analytics, and debugging — observing provider lifecycle events, state changes, and error handling
---

# ProviderObservers

ProviderObserver listens to provider lifecycle events globally. Used for logging, analytics, and debugging.

## Usage

Extend `ProviderObserver` and override lifecycle methods:

```dart
class MyObserver extends ProviderObserver {
  @override
  void didAddProvider(ProviderObserverContext context, Object? value) {
    print('Provider ${context.provider} added with value: $value');
  }

  @override
  void didUpdateProvider(ProviderObserverContext context, Object? previousValue, Object? newValue) {
    print('${context.provider}: $previousValue -> $newValue');
  }

  @override
  void providerDidFail(ProviderObserverContext context, Object error, StackTrace stackTrace) {
    if (error is ProviderException) return; // Already logged by root cause
    print('Provider failed: $error');
  }
}
```

Register on `ProviderScope` or `ProviderContainer`:

```dart
runApp(
  ProviderScope(
    observers: [MyObserver()],
    child: MyApp(),
  ),
);
```

## Provider Names

For better logging, give providers a name:

```dart
final myProvider = Provider<int>((ref) => 0, name: 'MyProvider');
```

With code generation, names are assigned automatically.

## Riverpod 3.0 Changes

The observer interface changed: instead of separate `ProviderBase` and `ProviderContainer` parameters, a single `ProviderObserverContext` object is passed containing both plus extra info (like associated mutation).

## Gotcha: Mutable State

If state is mutated in place (e.g., modifying a List with `ref.notifyListeners`), `previousValue` and `newValue` may be the same reference. Clone objects before mutating to get distinct values.

<!--
Source references:
- https://riverpod.dev/docs/concepts2/observers
-->
