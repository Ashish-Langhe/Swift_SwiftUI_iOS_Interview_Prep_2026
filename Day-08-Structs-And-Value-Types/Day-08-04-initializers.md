# Day 8: Initializers

## Core Idea

Initializers create valid instances.

```swift
struct User {
    let id: String
    let name: String

    init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}
```

Structs often get a memberwise initializer automatically.

## Validation

```swift
struct Email {
    let value: String

    init?(_ value: String) {
        guard value.contains("@") else { return nil }
        self.value = value
    }
}
```

## Interview Levels

Junior:

Initializers set up values.

Senior:

Initializers should establish invariants. Failable initializers are useful when invalid input should not create an instance.

## Quick Notes

- `init` creates instances
- Structs get memberwise init
- Use failable init for invalid input
- Initialization should create valid state

## Interview Depth

Junior answer: An initializer creates an instance and sets its properties.

Mid-level answer: Structs get a memberwise initializer automatically, but custom initializers are useful for validation, transformation, or defaults.

Senior answer: Initializers should establish invariants. If an instance cannot be valid with bad input, use a failable or throwing initializer instead of creating a broken object.

iOS use case:

```swift
struct EmailAddress {
    let value: String

    init?(_ value: String) {
        guard value.contains("@") else { return nil }
        self.value = value
    }
}
```

Common mistakes:

- Allowing invalid model state.
- Doing too much work in init.
- Force-unwrapping inside init.

Practice:

1. Create failable initializer.
2. Add default parameter in init.
3. Explain invariant.
