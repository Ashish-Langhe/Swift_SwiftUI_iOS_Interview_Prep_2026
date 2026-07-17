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
