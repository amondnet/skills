---
name: features-offline
description: Experimental offline persistence — persisting provider state to local database (riverpod_sqflite), Storage interface, persist keys, cache duration, destroy keys for migration, testing with in-memory storage
---

# Offline Persistence (Experimental)

Persist provider state to a local database so it survives app restarts and works offline.

> API may change in breaking ways without a major version bump.

## Two-Part System

1. **Storage** — interface to your database (e.g., `riverpod_sqflite` for SQLite)
2. **AnyNotifier.persist** — opt-in method called in `build`

## Setup with SQFlite

```bash
dart pub add riverpod_sqflite sqflite
```

```dart
@riverpod
Future<JsonSqFliteStorage> storage(Ref ref) async {
  return JsonSqFliteStorage.open(
    join(await getDatabasesPath(), 'riverpod.db'),
  );
}
```

## Persisting a Notifier

### Manual serialization

```dart
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  FutureOr<List<Todo>> build() async {
    persist(
      ref.watch(storageProvider.future),
      key: 'todos',
      encode: jsonEncode,
      decode: (json) => (jsonDecode(json) as List)
          .map((e) => Todo.fromJson(e as Map<String, Object?>))
          .toList(),
    );
    return await fetchTodos();
  }
}
```

### With @JsonPersist (code generation)

```dart
@riverpod
@JsonPersist()
class TodosNotifier extends _$TodosNotifier {
  @override
  FutureOr<List<Todo>> build() async {
    persist(ref.watch(storageProvider.future));
    return await fetchTodos();
  }
}
```

## Persist Key Rules

- **Unique** across all persisted providers (assertion thrown on duplicates)
- **Stable** across app restarts (don't change keys)
- **Include family parameters** when using family providers

## Cache Duration

Default: 2 days. Customize:

```dart
persist(
  storage,
  key: 'preferences',
  options: const StorageOptions(cacheTime: StorageCacheTime.unsafe_forever),
  // ...
);
```

If using infinite cache, manually delete from DB when provider is removed.

## Destroy Keys (Simple Migration)

When data structure changes, use a destroy key to discard old data:

```dart
persist(
  storage,
  key: 'todos',
  destroyKey: 'v2', // Change when schema changes
  // ...
);
```

## Awaiting Persisted State

By default, `persist` runs without await (network requests start immediately). To initialize from cache:

```dart
await persist(...).future;
// Now this.state contains the persisted value
```

## Testing with In-Memory Storage

```dart
final container = ProviderContainer.test(
  overrides: [
    storageProvider.overrideWithValue(AsyncData(Storage.inMemory())),
  ],
);
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/offline
-->
