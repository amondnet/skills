---
name: best-practices-testing
description: Testing Riverpod providers — unit tests with ProviderContainer.test, widget tests with ProviderScope, mocking providers, spying on changes, async testing, notifier mocking patterns
---

# Testing Providers

Riverpod makes testing straightforward with isolated state per test, built-in mocking, and no shared state between tests.

## Unit Tests

Use `ProviderContainer.test()` — auto-disposes after the test:

```dart
test('counter increments', () {
  final container = ProviderContainer.test();

  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);
});
```

### Auto-Dispose Caution

With auto-dispose providers, use `container.listen` instead of `container.read` to keep the provider alive during the test:

```dart
test('auto-dispose provider', () {
  final container = ProviderContainer.test();
  final sub = container.listen(myProvider, (_, __) {});

  expect(sub.read(), expectedValue);
});
```

## Widget Tests

Wrap with `ProviderScope`:

```dart
testWidgets('shows counter', (tester) async {
  await tester.pumpWidget(
    ProviderScope(child: MyApp()),
  );
  expect(find.text('0'), findsOneWidget);
});
```

Access container via `tester.container()`:

```dart
testWidgets('can read providers', (tester) async {
  await tester.pumpWidget(ProviderScope(child: MyApp()));
  final container = tester.container();
  expect(container.read(counterProvider), 0);
});
```

## Mocking Providers

Use `overrides` — no setup needed:

```dart
final container = ProviderContainer.test(
  overrides: [
    repositoryProvider.overrideWith((ref) => MockRepository()),
  ],
);
```

### overrideWithBuild (Notifiers)

Mock only initialization, keep methods intact:

```dart
overrides: [
  counterProvider.overrideWithBuild((ref) => 42),
  // increment() still works, just starts at 42
]
```

### overrideWithValue (Future/Stream)

```dart
overrides: [
  myFutureProvider.overrideWithValue(AsyncData(42)),
]
```

## Spying on Changes

```dart
test('spy on changes', () {
  final container = ProviderContainer.test();
  final changes = <int>[];

  container.listen(counterProvider, (prev, next) {
    changes.add(next);
  });

  container.read(counterProvider.notifier).increment();
  expect(changes, [1]);
});
```

## Async Providers

Await `.future` to wait for async completion:

```dart
test('async provider', () async {
  final container = ProviderContainer.test();
  final value = await container.read(myAsyncProvider.future);
  expect(value, expectedValue);
});
```

## Mocking Notifiers

Discouraged — prefer mocking dependencies (repositories, services) instead.

If needed, your mock must subclass the Notifier base class and use `with Mock` (from `mockito`):

```dart
// With codegen — must be in same file as the notifier
// Requires mockito in dev_dependencies
class MyNotifierMock extends _$MyNotifier with Mock implements MyNotifier {}

// Without codegen
class MyNotifierMock extends Notifier<int> with Mock implements MyNotifier {}

// Usage
overrides: [
  myProvider.overrideWith(MyNotifierMock.new),
]
```

<!--
Source references:
- https://riverpod.dev/docs/how_to/testing
-->
