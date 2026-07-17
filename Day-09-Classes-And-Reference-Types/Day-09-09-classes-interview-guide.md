# Day 9: Classes And Reference Types Interview Guide

## One-Minute Interview Answer

Classes are reference types in Swift. Multiple variables can point to the same object, so mutation through one reference is visible through others. Classes support identity, inheritance, overriding, deinitialization, and ARC-managed memory. I use classes when identity, shared mutable state, UIKit subclassing, or Objective-C interoperability is needed. For simple data models, structs are usually safer.

## Modern Swift 6.x Notes

Swift 6's concurrency model makes shared mutable reference types important to design carefully. Use `@MainActor` for UI-bound classes, actors for isolated mutable state, and immutable value snapshots where possible.

## Junior Questions

What is a class?

A reference type with properties and methods.

What is ARC?

Automatic Reference Counting manages class memory.

## Senior Questions

When choose class over struct?

When identity, shared mutation, inheritance, deinit, or framework requirements matter.

How avoid retain cycles?

Use weak delegates, capture lists, and clear ownership.

## Common Traps

- Using classes for every model
- Forgetting closure retain cycles
- Deep inheritance hierarchies
- Missing `final`
- Confusing `==` and `===`

## Topic-By-Topic Deep Dive

### Class Declaration

```swift
final class SessionManager {
    var currentUser: User?
}
```

Senior note:

Use `final` unless the class is intentionally designed for inheritance.

### Reference Semantics

```swift
let first = SessionManager()
let second = first
second.currentUser = user
```

Both references see the same object.

Senior answer:

Reference semantics are useful for identity and shared state, but they increase ownership and concurrency complexity.

### Identity Vs Equality

`===` checks same object instance. `==` checks value equality.

```swift
if controllerA === controllerB {
    print("Same screen object")
}
```

### Initializers And Deinitializers

`deinit` helps confirm lifetime:

```swift
deinit {
    print("ProfileViewModel deallocated")
}
```

If it never prints, investigate retain cycles.

### Inheritance

Inheritance is common in UIKit:

```swift
final class ProfileViewController: UIViewController { }
```

Senior note:

For app business logic, prefer composition unless a true subtype relationship exists.

### Method Overriding

```swift
override func viewDidLoad() {
    super.viewDidLoad()
}
```

Call `super` when framework contracts require it.

### Final

`final` documents that subclassing is not supported. It also gives the compiler optimization opportunities.

### Memory Implications

Classes use ARC. Watch strong reference cycles:

```swift
closure = { [weak self] in
    self?.refresh()
}
```

## Production Decision Table

| Need | Use |
| --- | --- |
| Identity | Class |
| Shared mutable service | Class or actor |
| UIKit screen | Class |
| Simple data | Struct |
| Prevent subclassing | `final` |
| Avoid retain cycle | `weak` / capture list |
| Isolated mutable state | Actor |

## Final Revision

- Classes are reference types
- `===` checks identity
- Inheritance needs `override`
- `final` prevents subclassing
- ARC manages memory
- Weak references avoid cycles
