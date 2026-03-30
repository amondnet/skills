---
name: core-providers
description: Riverpod provider types (Provider, FutureProvider, StreamProvider, NotifierProvider, AsyncNotifierProvider, StreamNotifierProvider), creation syntax for both codegen and manual approaches
---

# Providers

Providers are the central feature of Riverpod — memoized functions with built-in caching, error handling, and lifecycle management.

## Provider Types (6 variants)

|              | Synchronous        | Future                  | Stream                   |
|--------------|---------------------|-------------------------|--------------------------|
| Unmodifiable | `Provider`          | `FutureProvider`        | `StreamProvider`         |
| Modifiable   | `NotifierProvider`  | `AsyncNotifierProvider` | `StreamNotifierProvider` |

- **Unmodifiable** (functional): Cannot be externally modified by widgets. Like a private setter.
- **Modifiable** (notifier): Expose public methods for external mutation. Like a public setter.

Pick based on return type + whether UI needs to modify state. `FutureProvider` and `AsyncNotifierProvider` are the most common.

## Creating Providers

### Functional (Unmodifiable) — Code Generation

```dart
@riverpod
Future<User> user(Ref ref) async {
  final response = await http.get('https://api.example.com/user/123');
  return User.fromJson(response.body);
}
// Generated: userProvider
```

### Functional (Unmodifiable) — Manual

```dart
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get('https://api.example.com/user/123');
  return User.fromJson(response.body);
});
```

### Notifier (Modifiable) — Code Generation

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
// Generated: counterProvider
```

### Notifier (Modifiable) — Manual

```dart
final counterProvider = NotifierProvider<Counter, int>(Counter.new);

class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
}
```

## Key Rules

- Providers must be **top-level final variables** (global). This is safe — they are fully immutable.
- Multiple providers can expose the same type without conflict.
- The `build` method is where initialization logic goes. Do not use constructors for logic.
- Wrap your app with `ProviderScope` before using providers:

```dart
void main() {
  runApp(ProviderScope(child: MyApp()));
}
```

## Notifier Gotchas

- **Never put logic in Notifier constructors** — `ref` is not available there. Use `build` instead.
- Public methods on notifiers are accessible via `ref.read(provider.notifier).method()`.

<!--
Source references:
- https://riverpod.dev/docs/concepts2/providers
-->
