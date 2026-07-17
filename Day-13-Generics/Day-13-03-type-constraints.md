# Day 13: Type Constraints

## Core Idea

Constraints limit generic types to required capabilities.

```swift
func printSorted<T: Comparable>(_ values: [T]) {
    print(values.sorted())
}
```

## Real iOS Use Cases

```swift
func save<T: Encodable>(_ value: T) throws {
    let data = try JSONEncoder().encode(value)
    print(data)
}
```

## Interview Levels

Junior: Constraints require a generic type to conform to a protocol.

Senior: Constraints let generic code use specific operations while staying flexible and type-safe.

## Quick Notes

- `T: Protocol`.
- Enables methods from constraint.
- Keeps generic code safe.

## Interview Depth

Junior answer: A type constraint says the generic type must follow a protocol or inherit from a class.

Mid-level answer: Constraints let generic code use specific operations, like sorting `Comparable` values or encoding `Encodable` values.

Senior answer: Constraints should be as specific as necessary and as narrow as possible. Over-constraining reduces API flexibility.

iOS use case:

```swift
func cacheKey<T: Identifiable>(for value: T) -> String {
    String(describing: value.id)
}
```

Common mistakes: requiring `Hashable` when `Equatable` is enough, using class constraints unnecessarily, unreadable generic signatures.

Practice: constrain to Decodable, constrain to Identifiable, explain narrow constraints.
