# Day 5: Dictionaries

## Core Idea

A `Dictionary` stores key-value pairs.

```swift
var scores: [String: Int] = [
    "Aarav": 90,
    "Meera": 85
]
```

Keys must be unique and conform to `Hashable`.

## Common Operations

```swift
scores["Kabir"] = 92
scores["Meera"] = 88
scores["Aarav"] = nil

let meeraScore = scores["Meera"]
```

Dictionary lookup returns an optional because the key may not exist.

```swift
if let score = scores["Meera"] {
    print(score)
}
```

## Real iOS Use Cases

- Cache by ID
- Lookup table
- API JSON-like payloads
- Grouped data
- Feature flags

```swift
let usersById: [String: User] = [
    "u1": User(name: "Aarav"),
    "u2": User(name: "Meera")
]
```

## Dictionary Iteration

```swift
for (name, score) in scores {
    print("\(name): \(score)")
}
```

Do not depend on dictionary order for business logic. Sort when order matters.

```swift
for key in scores.keys.sorted() {
    print(key)
}
```

## Modern Swift 6.x Notes

The basic `Dictionary` API has not changed dramatically in Swift 6.3, but Swift 6's concurrency direction makes immutable dictionary snapshots useful when passing data across tasks. Use value types and `Sendable`-friendly models where possible.

## Interview Levels

Junior:

A dictionary stores values using keys.

Mid-level:

Dictionaries are useful for fast lookup by key. Access returns an optional because the key may be missing.

Senior:

Dictionaries are ideal for identity-based lookup, caches, and grouping. I avoid using them when stable order matters unless I explicitly sort or maintain a separate ordered structure.

## Quick Notes

- Key-value collection
- Keys are unique
- Key must be `Hashable`
- Lookup returns optional
- Great for fast lookup

## Interview Depth

Junior answer: A dictionary stores values using keys. You use the key to get the value.

Mid-level answer: Dictionary lookup returns an optional because a key may not exist. Dictionaries are useful for caches, lookup tables, grouped API data, and normalized state.

Senior answer: Dictionaries are about access patterns. If a screen frequently needs `User` by `id`, storing `[String: User]` avoids repeated array searches. When UI order also matters, keep ordered IDs separately from the dictionary. This pattern is common in Redux-style stores, SwiftUI state containers, and offline caches.

iOS use case:

```swift
struct UserStore {
    private var usersById: [String: User] = [:]

    func user(id: String) -> User? {
        usersById[id]
    }
}
```

Common mistakes:

- Assuming dictionary iteration order is business-stable.
- Force-unwrapping dictionary lookups.
- Using `String: Any` instead of typed models.
- Forgetting keys must be unique.

Practice:

1. Convert `[User]` to `[String: User]`.
2. Safely read a missing key.
3. Explain dictionary vs array for user lookup.
