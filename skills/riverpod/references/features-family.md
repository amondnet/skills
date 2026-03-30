---
name: features-family
description: Family providers — parameterized providers that create independent state per parameter combination, usage patterns, parameter requirements, overriding in tests
---

# Family (Parameterized Providers)

Family allows a provider to hold multiple independent states based on unique parameter combinations. Think of it as a Map where keys are parameters and values are provider states.

## Defining a Family

### Code Generation — Natural Parameters

```dart
@riverpod
Future<User> user(Ref ref, String userId) async {
  return fetchUser(userId);
}

// Class-based
@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  Future<User> build(String userId) async {
    return fetchUser(userId);
  }
}
```

With codegen, any number of parameters are supported — named, optional, default values.

### Manual — Family Modifier

```dart
final userProvider = FutureProvider.family<User, String>((ref, userId) async {
  return fetchUser(userId);
});
```

## Using a Family

Pass parameters when watching:

```dart
final user = ref.watch(userProvider('123'));
```

Multiple reads are independent:

```dart
final user1 = ref.watch(userProvider('123'));
final user2 = ref.watch(userProvider('456'));
// user1 and user2 are completely independent states
```

## Parameter Requirements

Parameters **must** have consistent `==`/`hashCode`. Avoid mutable collections:

```dart
// BAD — [1, 2, 3] != [1, 2, 3] (different instances)
ref.watch(myProvider([1, 2, 3]));

// GOOD — use const or cached values
ref.watch(myProvider(const [1, 2, 3]));
```

Enable `riverpod_lint`'s `provider_parameters` rule to catch this.

## Auto-Dispose with Family

Always enable auto-dispose for families to prevent memory leaks (one state per parameter = potential unbounded growth):

```dart
@riverpod // auto-dispose is on by default with codegen
Future<User> user(Ref ref, String userId) async { ... }
```

## Overriding in Tests

Override specific parameter combination:

```dart
final container = ProviderContainer.test(
  overrides: [
    userProvider('123').overrideWith((ref) => mockUser),
  ],
);
```

Override all parameter combinations:

```dart
final container = ProviderContainer.test(
  overrides: [
    userProvider.overrideWith((ref, userId) => mockUser),
  ],
);
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/family
-->
