---
name: features-codegen
description: Riverpod code generation with @riverpod annotation — syntax for functional/class-based providers, auto-dispose default, family parameters, migration from manual to codegen
---

# Code Generation

Riverpod supports optional code generation via `riverpod_generator` and the `@riverpod` annotation. Recommended only if your project already uses codegen (Freezed, json_serializable).

## When to Use

- **Use if** your project already has `build_runner` for Freezed, json_serializable, etc.
- **Skip if** you don't use codegen elsewhere — the build step overhead may not be worth it for Riverpod alone.

## Benefits

- Simpler syntax with less boilerplate
- Auto-picks the correct provider type based on return type
- Unrestricted family parameters (named, optional, defaults)
- Stateful hot-reload support
- Better debugging metadata

## Syntax

### Functional (unmodifiable)

```dart
// Sync → generates Provider
@riverpod
String helloWorld(Ref ref) => 'Hello world!';

// Async Future → generates FutureProvider
@riverpod
Future<User> user(Ref ref) async => fetchUser();

// Async Stream → generates StreamProvider
@riverpod
Stream<int> tick(Ref ref) => Stream.periodic(Duration(seconds: 1), (i) => i);
```

### Class-based (modifiable / notifier)

```dart
// Sync → generates NotifierProvider
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  void increment() => state++;
}

// Async → generates AsyncNotifierProvider
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async => fetchTodos();
  Future<void> add(Todo todo) async { ... }
}
```

### Auto-Dispose (default with codegen)

```dart
// Disable auto-dispose
@Riverpod(keepAlive: true)
String persistent(Ref ref) => 'kept alive';
```

### Family Parameters

```dart
// Any parameters — named, optional, defaults all supported
@riverpod
Future<User> user(Ref ref, String userId, {bool refresh = false}) async {
  return fetchUser(userId, refresh: refresh);
}

// Usage
ref.watch(userProvider('123', refresh: true));
```

### Generic Support (Riverpod 3.0)

```dart
@riverpod
T multiply<T extends num>(Ref ref, T a, T b) => a * b;

// Usage
int result = ref.watch(multiplyProvider<int>(2, 3));
```

## Generated Names

- Function `myFunction` → `myFunctionProvider`
- Class `MyNotifier` → `myNotifierProvider`

## Riverpod 3.0: Simplified Ref

No more per-provider Ref types. Use `Ref` directly:

```dart
// Before (2.x)
int example(ExampleRef ref) { ... }

// After (3.0)
int example(Ref ref) { ... }
```

<!--
Source references:
- https://riverpod.dev/docs/concepts/about_code_generation
-->
