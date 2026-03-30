---
name: advanced-migration-v3
description: Migration guide from Riverpod 2.x to 3.0 — automatic retry, paused out-of-view providers, legacy imports, unified Ref, removed AutoDispose/Family interfaces, ProviderException wrapping
---

# Migrating from Riverpod 2.x to 3.0

## Automatic Retry (new default)

Providers now retry on failure. Disable globally or per-provider:

```dart
// Globally
ProviderScope(retry: (retryCount, error) => null, child: MyApp());

// Per-provider
@Riverpod(retry: (_, __) => null)
class TodoList extends _$TodoList { ... }
```

## Out-of-View Providers Paused

Widgets not visible (e.g., behind a route) have their listeners paused. Control with `TickerMode`:

```dart
TickerMode(
  enabled: true, // Force listeners to stay active
  child: Consumer(builder: (context, ref, _) {
    ref.watch(myProvider); // Won't be paused
  }),
);
```

## Legacy Providers Moved

`StateProvider`, `StateNotifierProvider`, `ChangeNotifierProvider` require a new import:

```dart
// Before
import 'package:riverpod/riverpod.dart';
// After
import 'package:riverpod/legacy.dart';
// or flutter_riverpod/legacy.dart, hooks_riverpod/legacy.dart
```

## All Providers Use == for Updates

`StreamProvider`/`StreamNotifier` now filter by `==` (was `identical`). Override if needed:

```dart
@override
bool updateShouldNotify(AsyncValue<Todo> previous, AsyncValue<Todo> next) => true;
```

## Simplified Ref

Remove all Ref subclasses — use `Ref` directly:

```dart
// Before: int example(ExampleRef ref)
// After:
int example(Ref ref) { ... }
```

`Ref.state`, `Ref.listenSelf`, `Ref.future` moved to Notifier:

```dart
// Before: ref.state, ref.listenSelf, ref.future
// After: use Notifier class — state, listenSelf(), future
```

## AutoDispose Interfaces Removed

Replace `AutoDisposeNotifier` → `Notifier`, `AutoDisposeRef` → `Ref`, etc. Behavior unchanged — just interface unification.

## FamilyNotifier Removed

```dart
// Before
class CounterNotifier extends FamilyNotifier<int, String> {
  @override int build(String arg) => 0;
}

// After
class CounterNotifier extends Notifier<int> {
  CounterNotifier(this.arg);
  final String arg;
  @override int build() => 0;
}
```

## ProviderException Wrapping

Provider failures throw `ProviderException` instead of the raw error:

```dart
// Before
try { await ref.read(myProvider.future); }
on NotFoundException { ... }

// After
try { await ref.read(myProvider.future); }
on ProviderException catch (e) {
  if (e.exception is NotFoundException) { ... }
}
```

`AsyncValue.error` is unaffected — still receives the original error.

## ProviderObserver Interface Changed

```dart
// Before
void didAddProvider(ProviderBase provider, Object? value, ProviderContainer container)
// After
void didAddProvider(ProviderObserverContext context, Object? value)
```

## AsyncValue Changes

- `valueOrNull` renamed to `value` (old `value` removed)
- `AsyncValue` is now sealed — exhaustive pattern matching works
- New: `AsyncValue.isFromCache` (offline persistence flag)
- New: `AsyncLoading(progress: 0.5)` for progress tracking

<!--
Source references:
- https://riverpod.dev/docs/3.0_migration
- https://riverpod.dev/docs/whats_new
-->
