# Day 13: Generics Interview Examples

## Example 1: Generic Decode

```swift
func decode<T: Decodable>(_ type: T.Type, from data: Data) throws -> T {
    try JSONDecoder().decode(T.self, from: data)
}
```

## Example 2: Generic Result View State

```swift
enum LoadState<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(Error)
}
```

## Example 3: Generic Cache

```swift
struct Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]

    subscript(key: Key) -> Value? {
        get { storage[key] }
        set { storage[key] = newValue }
    }
}
```

## Interview Levels

Junior: Generics avoid writing the same code many times.

Senior: Generics are compile-time abstraction. They keep APIs reusable without sacrificing type safety.

## Modern Swift 6.x Notes

Swift 6.3 adds performance-control attributes for library authors such as specialization controls. Normal app developers mostly need to understand generic constraints, not micro-optimization attributes.

## Interview Depth

Junior answer: Generics help write reusable code without losing type information.

Mid-level answer: Common interview examples include generic decoding, generic state containers, generic caches, and generic repository helpers.

Senior answer: In real code, generics should improve correctness and reduce duplication. If a generic abstraction makes call sites harder to understand, a concrete type may be better.

iOS use case:

```swift
struct PagedResponse<Item: Decodable>: Decodable {
    let items: [Item]
    let nextPage: Int?
}
```

Common mistakes: generic for its own sake, replacing clear domain names with `T` everywhere, overusing type erasure.

Practice: explain `APIResponse<User>`, build generic cache, compare generic vs protocol.
