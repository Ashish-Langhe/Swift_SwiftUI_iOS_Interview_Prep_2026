# Day 5: Collections Interview Guide

## One-Minute Interview Answer

Swift has three core collection types: `Array`, `Dictionary`, and `Set`. Arrays are ordered and allow duplicates, dictionaries store key-value pairs for fast lookup by key, and sets store unique values for fast membership checks. Swift collections are value types with copy-on-write optimization, so they are safe and efficient for most app code. I choose collections based on access pattern: arrays for ordered UI lists, sets for selected IDs or uniqueness, and dictionaries for caches or lookup by ID.

## Junior Questions

What is an array?

An ordered collection of values.

What is a dictionary?

A key-value collection.

What is a set?

A collection of unique values.

## Mid-Level Questions

Why does dictionary lookup return optional?

Because the key may not exist.

When use `compactMap`?

When transformation may return nil and you want only non-nil results.

What is copy-on-write?

Swift shares storage until one copy mutates, then performs a real copy.

## Senior Questions

How do you choose collections?

I choose based on ordering, uniqueness, lookup needs, mutation patterns, and performance. Arrays are not always ideal for repeated membership checks; a set is better. Repeated lookup by ID should often be dictionary-backed.

How do collections relate to concurrency?

Immutable snapshots are easier to pass safely. Mutable shared collections should be actor-isolated or otherwise protected.

## Modern Swift 6.x Notes

- Swift 6.2 introduced `InlineArray` for fixed-size inline storage.
- Swift 6.2 introduced `Span` for safe contiguous memory access.
- These are important for systems/performance interviews, but normal iOS apps mostly use `Array`, `Dictionary`, and `Set`.

## Common Traps

- Depending on dictionary order
- Force-indexing arrays without checking bounds
- Using arrays for frequent membership tests
- Using dictionaries when order is required
- Forgetting that dictionary values are optional on lookup

## Topic-By-Topic Deep Dive

### Arrays

Arrays are ordered and index-based.

```swift
let orderedSteps = ["Login", "Verify OTP", "Home"]
```

Use arrays when order matters, duplicates are meaningful, or UI renders rows in a stable sequence.

Senior note:

Array lookup by index is fast, but searching by value is linear. If you repeatedly check membership in a large array, you probably want a `Set`.

### Dictionaries

Dictionaries are key-value lookup structures.

```swift
let usersById: [String: User] = Dictionary(
    uniqueKeysWithValues: users.map { ($0.id, $0) }
)
```

Senior note:

Dictionaries are excellent for normalized state. Many production apps keep ordered IDs in an array and entities in a dictionary.

```swift
let orderedUserIds: [String]
let usersById: [String: User]
```

This gives both stable UI order and fast lookup.

### Sets

Sets are for uniqueness and membership.

```swift
var selectedIds: Set<String> = []
selectedIds.insert("txn-1")
```

Senior note:

Sets are perfect for selection state, permission flags, and deduplication, but they do not preserve display order.

### Collection Mutability

```swift
let snapshot = transactions
var editableDraft = transactions
```

Senior note:

In SwiftUI, immutable snapshots often produce clearer rendering. Mutating collections in many places makes state flow harder to debug.

### Iteration Patterns

Use direct iteration when possible:

```swift
for transaction in transactions {
    print(transaction.title)
}
```

Use `enumerated()` when index is part of display or logic:

```swift
for (index, step) in steps.enumerated() {
    print("\(index + 1). \(step)")
}
```

Senior note:

Use a loop when you need `break`, `continue`, or early return. Use transformations when expressing data conversion.

### Sorting, Filtering, Mapping

```swift
let rows = users
    .filter(\.isActive)
    .sorted { $0.name < $1.name }
    .map { UserRow(title: $0.name) }
```

Senior note:

Transformation chains are clean for simple pipelines. If closures become long, extract named functions. For huge collections, think about lazy sequences and intermediate allocations.

### Performance Basics

Choose by access pattern:

| Access Pattern | Best Collection |
| --- | --- |
| Ordered display | Array |
| Fast membership | Set |
| Fast lookup by ID | Dictionary |
| Ordered display plus lookup | Array of IDs + Dictionary |

### Array Vs Set Vs Dictionary

Interview-ready answer:

Array answers "what order?" Set answers "is this included?" Dictionary answers "what value belongs to this key?"

## Practice Exercises

```swift
func uniqueTags(from tags: [String]) -> [String] {
    Array(Set(tags)).sorted()
}
```

```swift
func usersById(_ users: [User]) -> [String: User] {
    Dictionary(uniqueKeysWithValues: users.map { ($0.id, $0) })
}
```

## Final Revision

- Array: ordered, duplicates
- Set: unique, fast membership
- Dictionary: key-value lookup
- `map`: transform
- `filter`: select
- `reduce`: combine
- `compactMap`: transform and remove nil
- Use the collection that matches access patterns
