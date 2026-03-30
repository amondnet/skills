---
name: howto-select
description: Reduce widget/provider rebuilds using select to watch only specific properties of a provider's state, and selectAsync for async providers
---

# Reducing Rebuilds with Select

By default, `ref.watch` rebuilds when any property of the watched object changes. Use `select` to rebuild only when specific properties change.

## Basic Select

```dart
Consumer(
  builder: (context, ref, _) {
    // Only rebuilds when `name` changes, not when `age` changes
    final name = ref.watch(
      userProvider.select((user) => user.name),
    );
    return Text(name);
  },
);
```

## Multiple Selects

Call `select` multiple times per property:

```dart
final name = ref.watch(userProvider.select((u) => u.name));
final email = ref.watch(userProvider.select((u) => u.email));
```

## Async Select

For async providers, use `selectAsync` to get a `Future`:

```dart
@riverpod
Future<String> userName(Ref ref) async {
  final name = await ref.watch(
    userProvider.selectAsync((user) => user.name),
  );
  return 'Hello, $name';
}
```

## Caveats

- Selected properties must be **immutable**. Returning a mutated `List` won't trigger rebuilds.
- `select` slightly slows individual reads. Don't over-optimize — only use when the "other properties" change frequently.
- Benchmark first before adding `select` everywhere.

<!--
Source references:
- https://riverpod.dev/docs/how_to/select
-->
