# Day 13: Type Erasure Introduction

## Core Idea

Type erasure hides a concrete generic type behind a stable wrapper.

```swift
struct AnyValidator<Input> {
    private let validateClosure: (Input) -> Bool

    init<V: Validator>(_ validator: V) where V.Input == Input {
        validateClosure = validator.validate
    }

    func validate(_ input: Input) -> Bool {
        validateClosure(input)
    }
}
```

## Why It Exists

Protocols with associated types can be hard to store directly. Type erasure gives one concrete wrapper type.

## Interview Levels

Junior: Type erasure hides the real type.

Senior: Type erasure trades static type detail for storage/API flexibility. Use it when `any` or generics do not fit the API shape.

## Quick Notes

- Hides concrete type.
- Common with associated type protocols.
- Adds indirection.
- Use only when needed.

## Interview Depth

Junior answer: Type erasure hides the specific concrete type behind a wrapper.

Mid-level answer: It is useful when protocols with associated types or complex generic types need to be stored in one concrete type.

Senior answer: Type erasure is an API design tradeoff. It gives storage flexibility but adds indirection and may hide useful static information. Use it when generics or `any` cannot express the shape cleanly.

iOS use case:

```swift
struct AnyAnalyticsTracker {
    private let trackClosure: (String) -> Void

    init(track: @escaping (String) -> Void) {
        self.trackClosure = track
    }

    func track(_ event: String) {
        trackClosure(event)
    }
}
```

Common mistakes: type erasing too early, losing test clarity, building wrappers for simple cases.

Practice: create AnyValidator, explain why needed, compare with `any`.
