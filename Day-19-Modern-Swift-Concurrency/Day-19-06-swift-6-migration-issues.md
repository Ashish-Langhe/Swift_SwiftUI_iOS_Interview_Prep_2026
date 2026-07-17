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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Swift 6 Migration Issues is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while token stores, caches, UI models, Swift 6 migration, and shared services. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Swift 6 Migration Issues in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Always separate task lifetime from object lifetime; a task can outlive the screen that started it.
- State crossing a concurrency boundary should be immutable, actor-isolated, or clearly Sendable.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Swift 6 Migration Issues correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Swift 6 Migration Issues, but it shows the kind of production shape you should connect this topic to:

```swift
actor UserCache {
    private var storage: [User.ID: User] = [:]

    func user(id: User.ID) -> User? {
        storage[id]
    }

    func insert(_ user: User) {
        storage[user.id] = user
    }
}
```

Modern Swift concurrency asks who owns mutable state. If many tasks need the same cache, actor isolation is clearer than hoping callers use it safely.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

