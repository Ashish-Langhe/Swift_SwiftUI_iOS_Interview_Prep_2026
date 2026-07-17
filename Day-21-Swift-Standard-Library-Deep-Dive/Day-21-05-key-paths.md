# Day 21: Key Paths

## What Key Paths Are

Key paths are references to properties.

```swift
struct User {
    let name: String
    let age: Int
}

let nameKeyPath = \\User.name
let name = user[keyPath: nameKeyPath]
```

They let you pass property access as a value.

## Common Shorthand

```swift
let names = users.map(\\.name)
let activeUsers = users.filter(\\.isActive)
```

This is clean when the operation is simple property access.

## Writable Key Paths

```swift
struct Settings {
    var isNotificationsEnabled: Bool
}

var settings = Settings(isNotificationsEnabled: false)
settings[keyPath: \\Settings.isNotificationsEnabled] = true
```

Writable key paths can modify mutable properties.

## Real iOS Use Case: Sorting

```swift
func sort<Value: Comparable>(
    _ users: [User],
    by keyPath: KeyPath<User, Value>
) -> [User] {
    users.sorted { $0[keyPath: keyPath] < $1[keyPath: keyPath] }
}

let sortedByName = sort(users, by: \\.name)
let sortedByAge = sort(users, by: \\.age)
```

This avoids writing many nearly identical sort functions.

## Real iOS Use Case: Binding-Like Updates

```swift
func update<Value>(
    _ value: Value,
    at keyPath: WritableKeyPath<Settings, Value>,
    in settings: inout Settings
) {
    settings[keyPath: keyPath] = value
}

update(true, at: \\.isNotificationsEnabled, in: &settings)
```

## Senior iOS Engineer Artifact

```text
Artifact: Key Path Usage Decision
Topic: Key Paths
Use case: mapping, sorting, updating, binding, observation
Key path type: KeyPath / WritableKeyPath / ReferenceWritableKeyPath
Readability: Does shorthand clarify or obscure?
Safety: Is property access still type-safe?
Alternative: closure or explicit method
```

Senior lens:

- Key paths are best for simple property access.
- Closures are clearer when logic is more than access.
- Writable key paths can make generic update helpers powerful.
- Avoid clever key-path APIs that hide business rules.

## More Coding Examples

### Example 1: Extract Values

```swift
let emails = users.map(\\.email)
```

### Example 2: Generic Label Builder

```swift
func labels<Value>(
    for users: [User],
    keyPath: KeyPath<User, Value>
) -> [String] {
    users.map { String(describing: $0[keyPath: keyPath]) }
}
```

## Common Mistakes

- Using key paths for logic that deserves a closure.
- Confusing value writable and reference writable key paths.
- Over-designing generic update APIs.
- Making call sites too abstract.

## Interview Guide

Junior:

A key path is a type-safe reference to a property.

Mid-level:

Key paths are useful with `map`, sorting, and generic property access helpers.

Senior:

Key paths let APIs accept property access as data. I use them when they improve reuse and readability, but I switch to closures or explicit methods when business logic is involved.

## Practice

1. Map users to names using `\\.name`.
2. Sort by a provided key path.
3. Write a generic settings update helper.
4. Explain when a closure is better than a key path.
