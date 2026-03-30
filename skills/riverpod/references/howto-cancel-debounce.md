---
name: howto-cancel-debounce
description: Cancel and debounce network requests in Riverpod — using ref.onDispose with HTTP client cancellation, debouncing with delay + abort pattern, reusable extension method
---

# Cancel & Debounce Network Requests

## Cancelling Requests

Cancel in-flight requests when the user navigates away (provider disposed via auto-dispose):

```dart
@riverpod
Future<Activity> activity(Ref ref) async {
  final client = http.Client();
  ref.onDispose(client.close); // Cancel on dispose

  final response = await client.get(
    Uri.parse('https://api.example.com/activity'),
  );
  return Activity.fromJson(jsonDecode(response.body));
}
```

Key: `ref.onDispose` fires when the provider is no longer used, closing the HTTP client.

## Debouncing Requests

Delay requests until user stops triggering (e.g., search-as-you-type):

```dart
@riverpod
Future<Activity> activity(Ref ref) async {
  // Wait 500ms. If provider is invalidated during this time,
  // the disposed flag is set to true, causing us to bail out after the delay.
  var disposed = false;
  ref.onDispose(() => disposed = true);
  await Future.delayed(const Duration(milliseconds: 500));
  if (disposed) throw Exception('Cancelled');

  final response = await http.get(Uri.parse('https://api.example.com/activity'));
  return Activity.fromJson(jsonDecode(response.body));
}
```

It's safe to throw inside providers after disposal — Riverpod catches and ignores the exception.

## Reusable Extension (Both at Once)

```dart
extension DebounceAndCancelExtension on Ref {
  Future<void> debounceAndCancel({
    Duration duration = const Duration(milliseconds: 500),
  }) async {
    var disposed = false;
    onDispose(() => disposed = true);
    await Future.delayed(duration);
    if (disposed) throw StateError('Cancelled');
  }
}

// Usage
@riverpod
Future<Activity> activity(Ref ref) async {
  await ref.debounceAndCancel();

  final client = http.Client();
  ref.onDispose(client.close);

  final response = await client.get(Uri.parse('https://api.example.com/activity'));
  return Activity.fromJson(jsonDecode(response.body));
}
```

<!--
Source references:
- https://riverpod.dev/docs/how_to/cancel
-->
