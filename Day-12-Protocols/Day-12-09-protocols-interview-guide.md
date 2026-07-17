# Day 12: Protocols Interview Guide

## One-Minute Interview Answer

Protocols define contracts in Swift. A protocol can require properties, methods, initializers, and associated types. Types conform by implementing requirements. Protocol extensions provide default behavior. `any Protocol` represents an existential value, while `some Protocol` returns one hidden concrete type. Protocol-oriented programming is useful for capabilities, dependency injection, and testing, but protocols should be small and purposeful.

## Modern Swift 6.x Notes

Modern Swift makes existential usage explicit with `any`. Swift 6.2 improved Embedded Swift support for some `any` cases. Swift concurrency may require protocol values or conforming types to be `Sendable` when crossing tasks.

## Junior Questions

What is a protocol? A contract of requirements.

What is conformance? A type implementing the protocol.

## Senior Questions

When use `any` vs `some`?

Use `any` for storing arbitrary conforming values. Use `some` when returning a single hidden concrete type while preserving static type information.

When avoid protocols?

When there is only one implementation and no real abstraction boundary.

## Common Traps

- Huge protocols.
- Protocols with associated types used as existentials without understanding limitations.
- Confusing `any` and `some`.
- Overusing protocol-oriented programming.

## Topic-By-Topic Deep Dive

### Protocol Declaration

A protocol declares a capability.

```swift
protocol ImageLoading {
    func loadImage(from url: URL) async throws -> Data
}
```

Senior framing:

Do not create protocols only because "protocol-oriented programming is good." Create a protocol when you need an abstraction boundary, multiple implementations, test substitution, or capability modeling.

### Requirements

Requirements are the promises conforming types must fulfill.

```swift
protocol UserSessionProviding {
    var currentUser: User? { get }
    func refreshSession() async throws
}
```

Keep requirements small. A protocol with ten unrelated methods becomes hard to mock and violates interface segregation.

### Protocol Conformance

```swift
final class RemoteImageLoader: ImageLoading {
    func loadImage(from url: URL) async throws -> Data {
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}
```

Conformance says this concrete type can be used wherever `ImageLoading` behavior is needed.

### Protocol Extensions

Protocol extensions provide shared behavior.

```swift
extension ImageLoading {
    func loadThumbnail(from url: URL) async throws -> Data {
        try await loadImage(from: url)
    }
}
```

Senior caution:

Default implementations are useful, but they can hide behavior. Be especially careful when methods are not actual protocol requirements because dispatch behavior can surprise candidates.

### Associated Types

Associated types let a protocol refer to a placeholder selected by the conformer.

```swift
protocol Repository {
    associatedtype Entity
    func fetchAll() async throws -> [Entity]
}
```

Senior explanation:

This is powerful because the protocol captures a relationship between the repository and its entity type. It also means the protocol is not always easy to use as a simple stored existential.

### Existentials With `any`

```swift
let service: any ImageLoading = RemoteImageLoader()
```

`any` means the concrete type is hidden behind a protocol box. This is flexible but may lose static type relationships and can involve dynamic dispatch.

### Opaque Types With `some`

```swift
func makeView() -> some View {
    Text("Hello")
}
```

`some` hides the concrete type from the caller, but the compiler still knows the exact type.

Simple comparison:

- `any`: many possible conforming runtime types
- `some`: one specific hidden concrete type

### Protocol-Oriented Programming

Good use:

```swift
protocol AnalyticsTracking {
    func track(_ event: String)
}
```

This makes view models testable because tests can inject a mock tracker.

Bad use:

Creating a protocol for every single concrete type with no alternate implementation and no testing boundary.

## Production Decision Table

| Need | Protocol Feature |
| --- | --- |
| Define capability | Protocol |
| Share default behavior | Protocol extension |
| Placeholder type in protocol | Associated type |
| Store any conforming value | `any Protocol` |
| Hide return concrete type | `some Protocol` |
| Test dependency | Small focused protocol |

## Final Revision

- Protocol = contract.
- Extension = default behavior.
- Associated type = placeholder.
- `any` = existential.
- `some` = opaque concrete type.
