---
name: features-retry
description: Automatic retry with exponential backoff for failing providers — default behavior, custom retry logic, per-provider and global configuration, disabling retry, interaction with async awaiting
---

# Automatic Retry

Riverpod 3.0 automatically retries providers that fail with exponential backoff.

## Default Behavior

- Up to 10 retries
- Exponential backoff: 200ms → 6.4s
- Does **not** retry `Error`s (bugs, not recoverable)
- Does **not** retry `ProviderException`s (failures from dependencies — retry the root cause instead)

## Custom Retry Logic

A retry function returns `Duration?`: the delay before next retry, or `null` to stop.

```dart
Duration? myRetry(int retryCount, Object error) {
  if (retryCount >= 5) return null;
  if (error is ProviderException) return null;
  return Duration(milliseconds: 200 * (1 << retryCount));
}
```

### Per-Provider

```dart
@Riverpod(retry: myRetry)
int myProvider(Ref ref) => 0;

// Manual
final myProvider = Provider<int>(
  retry: myRetry,
  (ref) => 0,
);
```

### Global

```dart
runApp(
  ProviderScope(
    retry: myRetry,
    child: MyApp(),
  ),
);

// or ProviderContainer
final container = ProviderContainer(retry: myRetry);
```

## Disabling Retry

```dart
// Globally
ProviderScope(
  retry: (retryCount, error) => null,
  child: MyApp(),
);

// Per-provider
@Riverpod(retry: (_, __) => null)
```

## Awaiting with Retries

When using `.future`, it waits until either all retries are exhausted or the provider succeeds:

```dart
final value = await ref.watch(myProvider.future);
// Skips intermediate failures automatically
```

## ProviderException Propagation

If `providerA` fails and `providerB` watches it, `providerB` throws a `ProviderException` (not the original error). Only `providerA` is retried — retrying `providerB` would be pointless since the root cause is in `providerA`.

<!--
Source references:
- https://riverpod.dev/docs/concepts2/retry
-->
