---
name: riverpod
description: Riverpod reactive caching and state management for Dart/Flutter — providers, refs, consumers, auto-dispose, family, mutations, offline persistence, testing, and code generation
metadata:
  author: Minsu Lee
  version: "2026.3.30"
  source: Generated from https://github.com/rrousselGit/riverpod, scripts located at https://github.com/amondnet/skills
---

> The skill is based on Riverpod v3.3.1 (flutter_riverpod), generated at 2026-03-30.

Riverpod is a reactive caching and data-binding framework for Dart and Flutter. It provides compile-safe providers that are testable, composable, and support automatic disposal. Riverpod 3.0 introduces automatic retry, mutations, offline persistence, pause/resume, and a simplified unified API.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Providers | 6 provider types, creation syntax (codegen & manual), functional vs notifier | [core-providers](references/core-providers.md) |
| Refs | watch, listen, read, invalidate, refresh, mounted, lifecycle hooks | [core-refs](references/core-refs.md) |
| Consumers | Consumer, ConsumerWidget, ConsumerStatefulWidget, HookConsumerWidget | [core-consumers](references/core-consumers.md) |
| Containers | ProviderContainer, ProviderScope, testing containers, state storage | [core-containers](references/core-containers.md) |

## Features

### State Management

| Topic | Description | Reference |
|-------|-------------|-----------|
| Auto-Dispose | Automatic disposal, keepAlive, cacheFor extension pattern | [features-auto-dispose](references/features-auto-dispose.md) |
| Family | Parameterized providers, independent state per parameter combination | [features-family](references/features-family.md) |
| Mutations | Experimental: track side-effect state (loading/error/success) in UI | [features-mutations](references/features-mutations.md) |
| Offline Persistence | Experimental: persist provider state to local database | [features-offline](references/features-offline.md) |
| Automatic Retry | Exponential backoff retry for failing providers | [features-retry](references/features-retry.md) |

### Configuration & DI

| Topic | Description | Reference |
|-------|-------------|-----------|
| Overrides | Provider mocking for testing, DI, and environment configuration | [features-overrides](references/features-overrides.md) |
| Scoping | Change provider behavior for widget subtrees | [features-scoping](references/features-scoping.md) |
| Observers | ProviderObserver for logging, analytics, and debugging | [features-observers](references/features-observers.md) |
| Code Generation | @riverpod annotation, codegen syntax, auto-picked provider types | [features-codegen](references/features-codegen.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| DO/DON'T | Official guidelines — avoid widget init, ephemeral state, dynamic providers | [best-practices-do-dont](references/best-practices-do-dont.md) |
| Testing | Unit/widget tests, mocking, spying, async testing, notifier mocks | [best-practices-testing](references/best-practices-testing.md) |

## How-To Guides

| Topic | Description | Reference |
|-------|-------------|-----------|
| Cancel & Debounce | Cancel/debounce network requests with onDispose | [howto-cancel-debounce](references/howto-cancel-debounce.md) |
| Select | Reduce rebuilds by watching specific properties | [howto-select](references/howto-select.md) |
| Pull-to-Refresh | RefreshIndicator with ref.refresh and AsyncValue | [howto-pull-to-refresh](references/howto-pull-to-refresh.md) |
| Eager Initialization | Force lazy providers to initialize at app startup | [howto-eager-init](references/howto-eager-init.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| flutter_hooks | Hooks integration for local widget state | [advanced-hooks](references/advanced-hooks.md) |
| Migration to 3.0 | Breaking changes, unified API, new features in Riverpod 3.0 | [advanced-migration-v3](references/advanced-migration-v3.md) |
