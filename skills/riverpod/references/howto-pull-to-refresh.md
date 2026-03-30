---
name: howto-pull-to-refresh
description: Pull-to-refresh pattern with RefreshIndicator, ref.refresh, AsyncValue pattern matching for loading/error/data states
---

# Pull-to-Refresh

Riverpod natively supports pull-to-refresh through its declarative nature.

## Pattern

```dart
class ActivityPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final activity = ref.watch(activityProvider);

    return Scaffold(
      body: RefreshIndicator(
        onRefresh: () => ref.refresh(activityProvider.future),
        child: ListView(
          children: [
            switch (activity) {
              AsyncData(:final value) => Text(value.description),
              AsyncError(:final error) => Text('Error: $error'),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ],
        ),
      ),
    );
  }
}
```

## Key Points

1. **`ref.refresh(provider.future)`** returns a `Future` that completes when the refresh finishes — exactly what `RefreshIndicator.onRefresh` expects.

2. **AsyncValue pattern matching** handles three states:
   - `AsyncData` — show the data
   - `AsyncError` — show error with previous data if available
   - `AsyncLoading` — show spinner (only on initial load, not refresh)

3. During refresh, previous data remains visible (AsyncValue preserves `value` during re-fetching).

4. Use `value` (renamed from `valueOrNull` in 3.0) to access data that may be null:

```dart
switch (activity) {
  AsyncData(:final value) => Text(value.description),
  AsyncError(:final error, :final value?) =>
    Text('Error but had: ${value.description}'),
  _ => CircularProgressIndicator(),
}
```

<!--
Source references:
- https://riverpod.dev/docs/how_to/pull_to_refresh
-->
