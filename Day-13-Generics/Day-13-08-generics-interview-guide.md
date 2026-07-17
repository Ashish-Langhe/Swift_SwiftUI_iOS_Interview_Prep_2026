# Day 13: Generics Interview Guide

## One-Minute Interview Answer

Generics let Swift write reusable, type-safe code using placeholder types. Generic functions and types avoid duplication while preserving compile-time type information. Constraints like `T: Decodable` let generic code use required capabilities. Associated types make protocols generic-like, `where` clauses express advanced relationships, and type erasure can hide complex concrete types when storage or API boundaries require it.

## Modern Swift 6.x Notes

Swift 6.3 includes performance-control features for library APIs, including specialization-related attributes. For interviews, focus on why generics are type-safe and how constraints, `some`, `any`, and type erasure relate.

## Common Traps

- Using `Any` instead of generics.
- Overcomplicating simple APIs.
- Confusing associated types with generic parameters.
- Introducing type erasure too early.

## Topic-By-Topic Deep Dive

### Generic Functions

Generic functions let one implementation work with many concrete types.

```swift
func first<T>(from values: [T]) -> T? {
    values.first
}
```

Senior answer:

Generics are compile-time polymorphism. They preserve type information, unlike `Any`, so callers still get precise return types.

### Generic Types

```swift
struct Loadable<Value> {
    var value: Value?
    var isLoading: Bool
}
```

This can represent loading state for `User`, `[Product]`, `Image`, or any other value.

Senior use case:

Generic state containers and API response wrappers reduce duplication while staying type-safe.

### Type Constraints

```swift
func sortedValues<T: Comparable>(_ values: [T]) -> [T] {
    values.sorted()
}
```

The constraint gives the function access to comparison.

Senior answer:

Constraints should be as narrow as possible. Do not require `Hashable` if `Equatable` is enough.

### Associated Types With Generics

```swift
protocol Endpoint {
    associatedtype Response: Decodable
    var path: String { get }
}

func request<E: Endpoint>(_ endpoint: E) async throws -> E.Response {
    fatalError("Network implementation")
}
```

This keeps endpoint and response tied together at compile time.

### Generic Where Clauses

```swift
extension Array where Element: Identifiable {
    func ids() -> [Element.ID] {
        map(\.id)
    }
}
```

`where` clauses are excellent for conditional behavior.

### Type Erasure

Type erasure is useful when generic or associated-type-heavy APIs need one concrete storage type.

```swift
struct AnyDataSource<Item> {
    private let loadClosure: () async throws -> [Item]

    init(load: @escaping () async throws -> [Item]) {
        self.loadClosure = load
    }

    func load() async throws -> [Item] {
        try await loadClosure()
    }
}
```

Senior caution:

Type erasure adds indirection and can hide useful type information. Reach for it when the API genuinely needs storage flexibility.

## Production Decision Table

| Need | Generic Tool |
| --- | --- |
| Reusable algorithm | Generic function |
| Reusable container/state | Generic type |
| Need capability | Type constraint |
| Protocol placeholder | Associated type |
| Advanced relationship | `where` clause |
| Store complex generic abstraction | Type erasure |

## Final Revision

- Generic function: reusable behavior.
- Generic type: reusable container/model.
- Constraint: required capability.
- Associated type: protocol placeholder.
- Where clause: advanced constraint.
- Type erasure: hide concrete type.
