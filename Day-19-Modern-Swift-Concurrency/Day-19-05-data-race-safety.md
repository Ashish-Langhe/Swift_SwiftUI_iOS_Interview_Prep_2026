# Day 19: Data-Race Safety

## What A Data Race Is

A data race happens when:

- Two or more concurrent operations access the same memory
- At least one access is a write
- There is no safe synchronization or isolation

Example:

```swift
final class Counter {
    var value = 0
}

let counter = Counter()

Task { counter.value += 1 }
Task { counter.value += 1 }
```

This is unsafe because both tasks can mutate the same state.

## Swift's Safety Goal

Swift has long protected memory safety. Swift 6 extends safety toward data-race prevention.

The compiler uses:

- Actor isolation
- `Sendable`
- Global actors
- Structured concurrency
- Transfer checking
- Strict concurrency diagnostics

## Safe Pattern: Actor

```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func currentValue() -> Int {
        value
    }
}
```

Access:

```swift
await counter.increment()
```

## Safe Pattern: Immutable Values

```swift
struct UserSnapshot: Sendable {
    let id: String
    let name: String
}
```

Immutable values can safely cross tasks.

## Safe Pattern: Main Actor

```swift
@MainActor
final class FeedViewModel: ObservableObject {
    @Published var items: [FeedItem] = []
}
```

All UI state updates are serialized through the main actor.

## Unsafe Pattern: Shared Mutable Class

```swift
final class Cart {
    var items: [Item] = []
}
```

If shared across tasks, this needs isolation.

Better:

```swift
actor CartStore {
    private var items: [Item] = []

    func add(_ item: Item) {
        items.append(item)
    }
}
```

## Data Race vs Race Condition

Data race:

- Unsafe simultaneous memory access
- Compiler can often help

Race condition:

- Result depends on timing
- Can happen even without a low-level data race

Example actor race condition:

```swift
actor Inventory {
    private var count = 1

    func reserve() async -> Bool {
        guard count > 0 else { return false }
        await logAttempt()
        count -= 1
        return true
    }
}
```

No direct data race, but reentrancy can create logic bugs.

## Common Mistakes

- Assuming actor means no logic races
- Sharing mutable classes into tasks
- Returning mutable references from actors
- Using locks incorrectly
- Using `@unchecked Sendable` without a real synchronization strategy
- Confusing data race safety with business correctness

## Modern Swift 6.x Notes

Swift 6 language mode is the key modern milestone for data-race safety. Swift 6.2 focuses on making the model easier to use through default actor isolation and clearer explicit concurrency.

## Junior Interview Answer

A data race is unsafe concurrent access to the same mutable data. Swift uses actors and `Sendable` to help prevent it.

## Mid-Level Interview Answer

I avoid shared mutable classes across tasks. I use actors for shared mutable state, `@MainActor` for UI state, and immutable `Sendable` values for transfer.

## Senior Interview Answer

Data-race safety is about memory access correctness, while race-condition safety is about logical ordering. Swift 6 helps catch data races, but senior design still needs careful actor API design, reentrancy awareness, cancellation handling, and invariant protection.

## Practice

1. Convert a shared mutable class to an actor.
2. Explain data race vs race condition.
3. Find the reentrancy bug in an actor method.
4. Design a sendable snapshot for a mutable model.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Data-Race Safety is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Data-Race Safety in an app feature:

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

1. Write a minimal example that shows Data-Race Safety correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Data-Race Safety, but it shows the kind of production shape you should connect this topic to:

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

