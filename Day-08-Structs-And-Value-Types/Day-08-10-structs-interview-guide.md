# Day 8: Structs And Value Types Interview Guide

## One-Minute Interview Answer

Structs are value types in Swift. They can have stored properties, computed properties, initializers, methods, and mutating methods. Swift prefers structs because value semantics make state predictable. When a struct is copied, changes to one value do not affect the other. Swift collections use copy-on-write to keep value semantics efficient. I use structs for models, view models, configuration, and state snapshots, and classes when identity or shared mutable reference behavior is required.

## Modern Swift 6.x Notes

Swift 6's concurrency safety direction makes immutable structs and `Sendable` models increasingly important. Value snapshots are easier to pass across tasks than shared mutable class instances.

## Junior Questions

What is a struct?

A type that groups data and behavior.

What is `mutating`?

A keyword required when a struct method changes stored properties.

## Senior Questions

Why prefer structs?

Predictable value semantics, safer state, easier concurrency reasoning, and great fit for SwiftUI.

When use classes?

When identity, inheritance, shared mutable state, or Objective-C interoperability is needed.

## Common Traps

- Forgetting `mutating`
- Using classes for simple models
- Assuming copy-on-write changes behavior
- Putting UI formatting everywhere in domain models

## Topic-By-Topic Deep Dive

### Struct Declaration

```swift
struct UserProfile {
    let id: String
    var displayName: String
}
```

Senior note:

Structs are usually the default for data because their value semantics avoid hidden sharing.

### Stored Properties

Stored properties define state.

```swift
let id: String
var displayName: String
```

Make properties `let` unless domain behavior requires mutation.

### Computed Properties

```swift
var initials: String {
    displayName.split(separator: " ").compactMap(\.first).map(String.init).joined()
}
```

Use for cheap derived values. Avoid hiding expensive work.

### Initializers

Initializers should create valid values.

```swift
init?(email: String) {
    guard email.contains("@") else { return nil }
    self.email = email
}
```

Senior note:

Use failable initializers or throwing initializers to protect invariants.

### Methods

Methods express behavior owned by the value.

```swift
func isOwned(by userId: String) -> Bool {
    ownerId == userId
}
```

### Mutating Methods

```swift
mutating func markCompleted() {
    isCompleted = true
}
```

Mutation is explicit and requires a `var` instance.

### Value Semantics

```swift
var draft = profile
draft.displayName = "New Name"
```

Changing `draft` does not change `profile`.

### Copy-On-Write

Swift collections share storage until mutation. You get logical copying without paying full copy cost immediately.

### Why Swift Prefers Structs

Structs work beautifully with SwiftUI because the UI can render state snapshots.

Senior answer:

Prefer structs for data and classes for identity.

## Production Decision Table

| Need | Use |
| --- | --- |
| Simple model | Struct |
| Shared identity | Class |
| Local state update | Mutating method |
| Derived display value | Computed property |
| Validated construction | Failable/throwing init |
| Large collection value behavior | Copy-on-write |

## Final Revision

- Structs are value types
- Stored properties hold data
- Computed properties derive data
- Initializers create valid instances
- `mutating` changes value state
- Copy-on-write optimizes collections
- Swift prefers structs for predictable state
