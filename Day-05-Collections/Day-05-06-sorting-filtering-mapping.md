# Day 5: Sorting, Filtering, And Mapping

## Core Idea

Swift collections support expressive transformations.

```swift
let names = ["Meera", "Aarav", "Kabir"]
let sortedNames = names.sorted()
```

## Map

Transforms each element.

```swift
let uppercased = names.map { $0.uppercased() }
```

## Filter

Keeps matching elements.

```swift
let longNames = names.filter { $0.count > 5 }
```

## Sort

Returns sorted copy:

```swift
let sorted = names.sorted()
```

Sorts in place:

```swift
var editableNames = names
editableNames.sort()
```

## CompactMap

Transforms and removes nil.

```swift
let rawIds = ["1", "abc", "2"]
let ids = rawIds.compactMap { Int($0) }
```

## Real iOS Use Cases

```swift
let activeRows = users
    .filter { $0.isActive }
    .sorted { $0.name < $1.name }
    .map { UserRow(title: $0.name) }
```

## Modern Swift 6.x Notes

These APIs remain core Swift. In performance-sensitive code, be aware that chained transformations may create intermediate arrays unless optimized. For huge datasets, consider lazy sequences or streaming approaches.

## Interview Levels

Junior:

`map` changes values, `filter` keeps values, and `sorted` orders values.

Mid-level:

Use transformations when they make intent clear. Use `compactMap` for optional-producing transforms.

Senior:

I balance readability and performance. Transformation chains are excellent for view model mapping, but for very large collections or complex logic, a loop, lazy sequence, or dedicated algorithm can be clearer and cheaper.

## Quick Notes

- `map` transforms
- `filter` selects
- `sorted` returns a new array
- `sort` mutates
- `compactMap` removes nil

## Interview Depth

Junior answer: `map` changes each value, `filter` keeps matching values, and `sorted` orders values.

Mid-level answer: These functions make collection transformations concise. `compactMap` is useful when parsing or converting optional values.

Senior answer: Transformation chains are expressive, but each step should remain readable. For large datasets, consider intermediate allocations, lazy sequences, and whether a single loop would be clearer. Avoid `map` for side effects.

iOS use case:

```swift
let rows = products
    .filter { $0.isAvailable }
    .sorted { $0.title < $1.title }
    .map { ProductRow(title: $0.title, price: $0.price.formatted()) }
```

Common mistakes:

- Using `map` just to call `print`.
- Creating long chains with complex closures.
- Sorting every render instead of caching derived rows.

Practice:

1. Map API models to row view models.
2. Filter active users.
3. Explain `sort` vs `sorted`.
