# Day 8: Value Semantics

## Core Idea

Value semantics means each variable has its own logical value.

```swift
var first = User(name: "Aarav")
var second = first
second.name = "Meera"

print(first.name)  // Aarav
print(second.name) // Meera
```

## Why It Matters

Value semantics make code predictable.

## Real iOS Use Cases

- App state snapshots
- View models
- API responses
- Form drafts

## Modern Swift 6.x Notes

Value semantics work well with Swift concurrency because immutable values reduce shared mutable state risks. Use `Sendable` where values cross concurrency boundaries.

## Interview Levels

Junior:

Copying a struct gives a separate value.

Senior:

Value semantics reduce hidden sharing and make state transitions easier to reason about, especially in SwiftUI and concurrent code.

## Quick Notes

- Structs are value types
- Copies are logically independent
- Great for predictable state
- Helps concurrency reasoning

## Interview Depth

Junior answer: Value semantics means copies do not affect each other.

Mid-level answer: Structs, enums, and tuples have value semantics. This makes state easier to reason about than shared references.

Senior answer: Value semantics are central to SwiftUI and modern Swift architecture. State can be treated as snapshots, reducing hidden mutation and making concurrency safer.

iOS use case:

```swift
var original = ProfileDraft(name: "Aarav")
var edited = original
edited.name = "Meera"
```

Common mistakes:

- Expecting struct copies to share mutation.
- Assuming value semantics means expensive copying every time.
- Mixing reference properties inside value types without thought.

Practice:

1. Demonstrate copy independence.
2. Explain value vs reference semantics.
3. Explain value semantics in SwiftUI.
