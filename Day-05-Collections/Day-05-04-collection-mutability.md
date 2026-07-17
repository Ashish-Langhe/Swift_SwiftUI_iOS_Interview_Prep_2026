# Day 5: Collection Mutability

## Core Idea

Collection mutability depends on `let` vs `var`.

```swift
let fixed = [1, 2, 3]
var editable = [1, 2, 3]

editable.append(4)
```

You cannot mutate a collection declared with `let`.

## Struct Value Semantics

Swift collections are value types with copy-on-write optimization.

```swift
var first = [1, 2, 3]
var second = first
second.append(4)

print(first)  // [1, 2, 3]
print(second) // [1, 2, 3, 4]
```

## Real iOS Use Cases

Use `let` for data snapshots:

```swift
let visibleRows = viewModel.rows
```

Use `var` for changing state:

```swift
var selectedIds: Set<String> = []
```

## Modern Swift 6.x Notes

Swift 6's concurrency safety push makes collection mutability more important. Mutable shared collections can create data races. Prefer immutable snapshots, actor-isolated state, or main-actor view models for UI state.

## Interview Levels

Junior:

Use `let` for fixed collections and `var` for collections that change.

Mid-level:

Swift arrays, dictionaries, and sets are value types. Assigning them creates independent values logically, with copy-on-write for efficiency.

Senior:

Collection mutability is an architecture decision. Immutable snapshots simplify UI rendering and concurrency. Shared mutable collections should have clear ownership or actor isolation.

## Quick Notes

- `let` collection cannot mutate
- `var` collection can mutate
- Swift collections are value types
- Copy-on-write avoids unnecessary copying
- Avoid shared mutable state

## Interview Depth

Junior answer: Use `let` when the collection should not change and `var` when it should change.

Mid-level answer: Swift collections are value types. Assigning an array, set, or dictionary creates a logically separate value. Swift uses copy-on-write internally so this is efficient.

Senior answer: Mutability is part of state ownership. In SwiftUI and concurrent Swift, immutable collection snapshots are easier to reason about. Mutable shared collections should be isolated in an actor, owned by a main-actor view model, or protected by a clear synchronization strategy.

iOS use case:

```swift
@MainActor
final class TransactionsViewModel {
    private(set) var rows: [TransactionRow] = []

    func update(rows newRows: [TransactionRow]) {
        rows = newRows
    }
}
```

Common mistakes:

- Making every collection `var`.
- Mutating public collections from outside the owning type.
- Sharing mutable collections across tasks.

Practice:

1. Explain copy-on-write.
2. Refactor public `var items` to `private(set)`.
3. Explain why immutable snapshots help UI rendering.
