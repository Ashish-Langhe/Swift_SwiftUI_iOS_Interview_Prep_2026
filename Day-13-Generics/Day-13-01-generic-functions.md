# Day 13: Generic Functions

## Core Idea

Generic functions work with placeholder types.

```swift
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}
```

## Real iOS Use Cases

- Reusable decoding helpers
- Generic cache helpers
- Common collection utilities

```swift
func decode<T: Decodable>(_ type: T.Type, from data: Data) throws -> T {
    try JSONDecoder().decode(T.self, from: data)
}
```

## Interview Levels

Junior: Generics let one function work with many types.

Senior: Generics preserve type safety and static dispatch while avoiding duplicate code.

## Quick Notes

- Placeholder type like `T`.
- Type-safe reuse.
- Often constrained with protocols.

## Interview Depth

Junior answer: A generic function works with more than one type.

Mid-level answer: Generics avoid duplicate functions while keeping type safety.

Senior answer: Generic functions are compile-time abstraction. They are often better than `Any` because the compiler preserves input and output type relationships.

iOS use case:

```swift
func loadJSON<T: Decodable>(_ type: T.Type, from data: Data) throws -> T {
    try JSONDecoder().decode(T.self, from: data)
}
```

Common mistakes: using `Any`, overgeneric APIs, constraints that are too broad.

Practice: generic first element, generic decode, add constraint.
