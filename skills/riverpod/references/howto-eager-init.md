---
name: howto-eager-init
description: Eagerly initialize lazy providers at app startup using a Consumer widget under ProviderScope, handling loading states, and using requireValue
---

# Eager Initialization

Providers are lazy by default. To initialize at startup, watch them in a Consumer under your root `ProviderScope`.

## Pattern

```dart
void main() {
  runApp(
    ProviderScope(
      child: const _EagerInitialization(child: MyApp()),
    ),
  );
}

class _EagerInitialization extends ConsumerWidget {
  const _EagerInitialization({required this.child});
  final Widget child;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Eagerly initialize by watching
    ref.watch(someImportantProvider);
    ref.watch(anotherProvider);
    return child;
  }
}
```

This does **not** rebuild the entire app when providers change — the `child` widget reference stays the same.

## With Loading State

Show a loading screen until async providers complete:

```dart
class _EagerInitialization extends ConsumerWidget {
  const _EagerInitialization({required this.child});
  final Widget child;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final config = ref.watch(configProvider);
    if (config is AsyncLoading) {
      return const MaterialApp(home: Scaffold(body: Center(child: CircularProgressIndicator())));
    }
    return child;
  }
}
```

## Using requireValue

After confirming the provider is initialized, skip AsyncValue pattern matching in child widgets:

```dart
class MyPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Safe because _EagerInitialization guarantees it's loaded
    final config = ref.watch(configProvider).requireValue;
    return Text(config.apiUrl);
  }
}
```

`requireValue` throws with a clear message if data isn't available — useful for catching bugs.

<!--
Source references:
- https://riverpod.dev/docs/how_to/eager_initialization
-->
