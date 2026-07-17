# Day 13: Generic Types

## Core Idea

Types can be generic.

```swift
struct Box<Value> {
    let value: Value
}
```

## Real iOS Use Cases

```swift
struct APIResponse<Payload: Decodable>: Decodable {
    let data: Payload
}
```

## Interview Levels

Junior: A generic type can store different types safely.

Senior: Generic types are useful for reusable containers, response wrappers, data sources, and state machines while preserving compile-time type information.

## Quick Notes

- Generic placeholder on type.
- Strongly typed.
- Common in networking and state.

## Interview Depth

Junior answer: A generic type can hold or work with different types.

Mid-level answer: Generic types are useful for wrappers like `APIResponse<User>` or `LoadState<[Product]>`.

Senior answer: Generic types model reusable structure while preserving specific payload type. This is common in networking, caching, UI state, and repositories.

iOS use case:

```swift
enum LoadState<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(Error)
}
```

Common mistakes: using `Any` payload, making generic names unclear, generic where a concrete type is simpler.

Practice: create `Box<T>`, create `LoadState<Value>`, explain type safety.
