# Day 12: Associated Types

## Core Idea

Associated types are placeholders inside protocols.

```swift
protocol Repository {
    associatedtype Entity
    func all() -> [Entity]
}
```

Conforming type chooses the actual type.

```swift
struct UserRepository: Repository {
    func all() -> [User] { [] }
}
```

## Interview Levels

Junior: Associated type lets a protocol use a placeholder type.

Senior: Associated types make protocols generic-like, but they complicate existential use. This is where `some`, `any`, and type erasure become important.

## Quick Notes

- Declared with `associatedtype`.
- Conformer supplies concrete type.
- Common in collections and repositories.

## Interview Depth

Junior answer: Associated types are placeholder types inside protocols.

Mid-level answer: The conforming type decides what the associated type actually is.

Senior answer: Associated types model relationships. They are powerful for repositories, data sources, sequences, and parsers, but they make existential storage more complex.

iOS use case:

```swift
protocol Endpoint {
    associatedtype Response: Decodable
    var path: String { get }
}
```

Common mistakes: trying to use PATs as simple existentials without `any`/type erasure understanding, adding associated type when generic function is enough.

Practice: create repository protocol, use associated response, explain limitation.
