# Day 19: Swift 6 Migration Issues

## Why Migration Can Be Hard

Swift 6's stricter concurrency checks reveal hidden design problems.

Common existing code patterns:

- Mutable singleton services
- Completion handlers called from unknown queues
- UIKit view models without `@MainActor`
- Shared mutable caches
- Non-sendable objects captured in tasks
- Protocols missing isolation decisions

## Migration Strategy

Do not fix everything by adding annotations randomly.

Use this order:

1. Identify UI state and mark it `@MainActor`.
2. Identify shared mutable services and consider actors.
3. Convert callback APIs to async where practical.
4. Make value DTOs and domain models `Sendable`.
5. Replace reference sharing with immutable snapshots.
6. Use escape hatches only with documentation.

## Common Error: Main Actor Access

```swift
@MainActor
final class ViewModel {
    var title = ""
}

func update(viewModel: ViewModel) {
    // viewModel.title = "Done"
}
```

Fix:

```swift
func update(viewModel: ViewModel) {
    Task { @MainActor in
        viewModel.title = "Done"
    }
}
```

Or make the caller main-actor isolated.

## Common Error: Non-Sendable Capture

```swift
final class SearchRequest {
    var query: String
}

Task {
    await service.search(request)
}
```

Better:

```swift
struct SearchRequest: Sendable {
    let query: String
}
```

## Common Error: Protocol Isolation

```swift
protocol ProfilePresenting {
    func show(profile: Profile)
}
```

If this updates UI, it may need:

```swift
@MainActor
protocol ProfilePresenting {
    func show(profile: Profile)
}
```

But this affects all conformers and callers.

## Escape Hatches

Use carefully:

- `@preconcurrency import`
- `nonisolated`
- `@unchecked Sendable`
- `Task.detached`

These can be valid but should not become default migration tools.

## Real iOS Migration Example

Before:

```swift
final class FeedViewModel: ObservableObject {
    @Published var items: [FeedItem] = []

    func load() {
        service.load { items in
            self.items = items
        }
    }
}
```

After:

```swift
@MainActor
final class FeedViewModel: ObservableObject {
    @Published private(set) var items: [FeedItem] = []

    func load() async {
        do {
            items = try await service.items()
        } catch {
            items = []
        }
    }
}
```

## Common Mistakes

- Marking all services `@MainActor`
- Making every class `@unchecked Sendable`
- Using detached tasks to avoid actor warnings
- Ignoring third-party package warnings forever
- Changing public API isolation without considering clients
- Migrating without tests around async behavior

## Modern Swift 6.2 Notes

Swift 6.2 includes migration support for upcoming features and approachable concurrency changes that reduce annotation burden.

Important features to understand:

- Default actor isolation
- Nonisolated async caller-context behavior
- `@concurrent`
- Improved async debugging

## Junior Interview Answer

Swift 6 migration often means fixing stricter concurrency warnings and errors.

## Mid-Level Interview Answer

I start by marking UI state `@MainActor`, making data models `Sendable`, and converting shared mutable services to actors or safe synchronization.

## Senior Interview Answer

Swift 6 migration is an architecture cleanup, not just annotation work. I classify state ownership, isolate UI and shared mutable state, design sendable boundaries, review public API isolation, and use escape hatches only where legacy or third-party constraints require them.

## Practice

1. Migrate a callback view model to async/await and `@MainActor`.
2. Replace a mutable request class with a `Sendable` struct.
3. Decide whether a warning needs actor, snapshot, or annotation.
4. Explain why `@unchecked Sendable` is dangerous.
