# Day 9: Reference Semantics

## Core Idea

Reference semantics means multiple variables can refer to the same object.

```swift
final class Counter {
    var value = 0
}

let first = Counter()
let second = first
second.value = 10

print(first.value) // 10
```

## Real iOS Use Cases

- Shared services
- Observable objects
- View controllers
- Coordinators

## Interview Levels

Junior:

Classes are shared by reference.

Senior:

Reference semantics are powerful but can create hidden mutation. Ownership and threading must be clear.

## Quick Notes

- Variables share same object
- Mutating through one reference affects others
- Useful for shared identity
- Risky for concurrency if mutable

## Interview Depth

Junior answer: Reference semantics means two variables can refer to the same object.

Mid-level answer: If one reference changes a class instance, other references see that change.

Senior answer: Reference semantics are useful for identity and shared coordination, but hidden mutation can make bugs harder. Ownership should be explicit, especially in async code.

iOS use case:

```swift
let sharedSession = SessionStore()
let coordinatorSession = sharedSession
```

Both references point to the same store.

Common mistakes:

- Expecting class assignment to copy.
- Mutating shared state from many owners.
- Forgetting weak references for delegates.

Practice:

1. Demonstrate two references to same object.
2. Explain hidden mutation.
3. Compare with struct assignment.
