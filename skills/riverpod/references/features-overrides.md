---
name: features-overrides
description: Provider overrides for testing, debugging, dependency injection, and scoping — overrideWith, overrideWithValue, overrideWithBuild methods
---

# Provider Overrides

All providers can be overridden to change their behavior. Used for testing, debugging, DI, and scoping.

## Syntax

Overrides are specified on `ProviderScope` or `ProviderContainer` via the `overrides` parameter:

```dart
// Flutter
runApp(
  ProviderScope(
    overrides: [
      counterProvider.overrideWith((ref) => 42),
    ],
    child: MyApp(),
  ),
);

// Pure Dart / Tests
final container = ProviderContainer(
  overrides: [
    counterProvider.overrideWith((ref) => 42),
  ],
);
```

## Override Methods

### overrideWith — Replace the entire provider logic

```dart
myProvider.overrideWith((ref) => mockValue);
```

### overrideWithValue — Replace with a static value

```dart
// For FutureProvider/StreamProvider
myFutureProvider.overrideWithValue(AsyncValue.data(42));
```

### overrideWithBuild — Replace only the build method (Notifiers)

Keeps the notifier's methods intact, only mocks initialization:

```dart
myNotifierProvider.overrideWithBuild((ref) {
  return 42; // Mock initial state, but increment() still works
});
```

## Testing Pattern

```dart
test('my test', () {
  final container = ProviderContainer.test(
    overrides: [
      repositoryProvider.overrideWith((ref) => MockRepository()),
    ],
  );

  expect(container.read(todoListProvider), isEmpty);
});
```

```dart
testWidgets('widget test', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        repositoryProvider.overrideWith((ref) => MockRepository()),
      ],
      child: MyApp(),
    ),
  );
});
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/overrides
-->
