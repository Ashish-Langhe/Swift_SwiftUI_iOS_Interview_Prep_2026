# Day 7: Named Tuple Elements

## Core Idea

Tuple elements can have names.

```swift
let user = (id: 101, name: "Aarav")
print(user.id)
print(user.name)
```

Named elements improve readability.

## Destructuring

```swift
let (id, name) = user
print(id, name)
```

## Real iOS Use Cases

```swift
func screenSize() -> (width: Double, height: Double) {
    (width: 390, height: 844)
}
```

## Interview Levels

Junior:

Named tuple elements let you access values by name.

Senior:

Named tuples improve local clarity, but they do not replace proper domain types when behavior, conformance, or long-term API stability is needed.

## Quick Notes

- Names improve readability
- Destructuring is supported
- Use for small local results
- Use structs for public APIs

## Interview Depth

Junior answer: Named tuple elements let you access values using names instead of `.0` and `.1`.

Mid-level answer: Names improve readability for small grouped results, especially function returns.

Senior answer: Named tuples are still structurally lightweight and do not provide long-term API strength. Use them for local clarity, not durable domain modeling.

iOS use case:

```swift
func splitFullName(_ fullName: String) -> (first: String, last: String?) {
    let parts = fullName.split(separator: " ")
    return (String(parts.first ?? ""), parts.dropFirst().first.map(String.init))
}
```

Common mistakes:

- Thinking named tuple is same as struct.
- Using tuple names inconsistently.
- Returning optional-heavy tuples instead of modeling state.

Practice:

1. Return named tuple from a function.
2. Access tuple by name.
3. Refactor a tuple into a struct.
