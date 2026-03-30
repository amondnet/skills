---
name: features-scoping
description: Scoped providers — changing provider behavior for a subtree, dependencies declaration, performance optimization, and widget-specific customization
---

# Scoping Providers

Scoping changes a provider's behavior for a specific part of the widget tree. Useful for:

- Page/widget-specific customization
- Performance optimization (e.g., rebuilding only changed items in a ListView)
- Avoiding parameter passing (alternative to family)

> This feature is complex and may be reworked in the future. Use carefully.

## Defining a Scoped Provider

Opt-in by specifying `dependencies`:

```dart
// Code generation
@Riverpod(dependencies: [])
int currentItemId(Ref ref) => throw UnimplementedError();

// Manual
final currentItemIdProvider = Provider<int>(
  (ref) => throw UnimplementedError(),
  dependencies: [],
);
```

## Listening to Scoped Providers

Use normally:

```dart
final itemId = ref.watch(currentItemIdProvider);
```

If a provider depends on a scoped provider, include it in `dependencies`:

```dart
@Riverpod(dependencies: [currentItemId])
Widget itemWidget(Ref ref) {
  final id = ref.watch(currentItemIdProvider);
  // ...
}
```

Only scoped providers need to be listed in `dependencies`.

## Setting the Value

Override in a nested `ProviderScope`:

```dart
ListView.builder(
  itemBuilder: (context, index) {
    return ProviderScope(
      overrides: [
        currentItemIdProvider.overrideWithValue(items[index].id),
      ],
      child: const ItemWidget(),
    );
  },
);
```

## Static Safety (Code Generation)

With `riverpod_lint`, lint rules detect when a scoped provider's override is missing. Use `@Dependencies` on widgets to propagate requirements:

```dart
@Dependencies([currentItemId])
class ItemWidget extends ConsumerWidget { ... }
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/scoping
-->
