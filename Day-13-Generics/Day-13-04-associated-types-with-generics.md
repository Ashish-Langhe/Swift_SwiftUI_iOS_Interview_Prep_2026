# Day 13: Associated Types With Generics

## Core Idea

Associated types and generics often work together.

```swift
protocol DataSource {
    associatedtype Item
    func item(at index: Int) -> Item
}
```

Generic consumer:

```swift
func firstItem<S: DataSource>(from source: S) -> S.Item {
    source.item(at: 0)
}
```

## Interview Levels

Junior: Associated types are placeholders in protocols.

Senior: Associated types allow protocols to describe families of types. Generics can preserve those relationships at compile time.

## Quick Notes

- Associated types belong to protocols.
- Generics can refer to them.
- Useful for data sources and repositories.

## Interview Depth

Junior answer: Associated types are placeholders in protocols, and generics can use those placeholders.

Mid-level answer: This keeps relationships between types intact, such as an endpoint and its response model.

Senior answer: Associated types plus generics create powerful static guarantees. For example, requesting `UserEndpoint` can return `User` without casting.

iOS use case:

```swift
protocol Endpoint {
    associatedtype Response: Decodable
}

func execute<E: Endpoint>(_ endpoint: E) async throws -> E.Response {
    fatalError()
}
```

Common mistakes: erasing too early, confusing associated types with generic type parameters, making protocols hard to store.

Practice: build endpoint protocol, generic executor, explain static relationship.
