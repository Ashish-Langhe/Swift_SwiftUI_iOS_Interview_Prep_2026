# Day 21: Collection Protocols

## What Collection Protocols Are

Swift collections are built on protocols such as `Sequence`, `Collection`, `BidirectionalCollection`, `RandomAccessCollection`, `MutableCollection`, and `RangeReplaceableCollection`.

These protocols describe capabilities:

- Can the type be iterated?
- Can it be indexed more than once?
- Can it move backward?
- Can it jump to an index efficiently?
- Can elements be mutated?
- Can elements be inserted or removed?

This matters because APIs can depend on capability instead of a concrete type.

```swift
func firstTitle<C: Collection>(_ titles: C) -> C.Element? where C.Element == String {
    titles.first
}
```

The function works with arrays, slices, sets, and other collections that satisfy the contract.

## Protocol Ladder

```text
Sequence
  Collection
    BidirectionalCollection
      RandomAccessCollection

MutableCollection
RangeReplaceableCollection
```

Important idea:

`Array` is not the only collection. Many APIs should accept the protocol they actually need.

## `Sequence` vs `Collection`

`Sequence` can be iterated. It may be single-pass.

`Collection` can be traversed multiple times and has stable indices.

```swift
func printAll<S: Sequence>(_ values: S) {
    for value in values {
        print(value)
    }
}

func middle<C: Collection>(_ values: C) -> C.Element? {
    guard !values.isEmpty else { return nil }
    let index = values.index(values.startIndex, offsetBy: values.count / 2)
    return values[index]
}
```

Use `Collection` when you need indexing, count, or repeated traversal.

## `RandomAccessCollection`

`RandomAccessCollection` promises efficient index movement.

```swift
func page<C: RandomAccessCollection>(
    _ values: C,
    page: Int,
    size: Int
) -> Array<C.Element> {
    let startOffset = page * size
    guard startOffset < values.count else { return [] }

    let start = values.index(values.startIndex, offsetBy: startOffset)
    let end = values.index(start, offsetBy: size, limitedBy: values.endIndex) ?? values.endIndex
    return Array(values[start..<end])
}
```

This is a good constraint when paging arrays or other random-access data.

## `MutableCollection`

Use `MutableCollection` when you need to modify existing positions.

```swift
func markAllRead<C: MutableCollection>(_ messages: inout C)
where C.Element == Message {
    for index in messages.indices {
        messages[index].isRead = true
    }
}
```

This mutates elements in place but does not insert or remove elements.

## `RangeReplaceableCollection`

Use `RangeReplaceableCollection` when you need to add or remove elements.

```swift
func removeInactive<C: RangeReplaceableCollection>(_ users: inout C)
where C.Element == User {
    users.removeAll { !$0.isActive }
}
```

Arrays conform; many custom list-like types can conform too.

## Real iOS Use Case

Imagine a reusable filter helper:

```swift
func activeUsers<C: Collection>(_ users: C) -> [User]
where C.Element == User {
    users.filter(\\.isActive)
}
```

This accepts an `Array<User>`, an `ArraySlice<User>`, or another collection without forcing callers to convert early.

## Senior iOS Engineer Artifact

```text
Artifact: Collection Capability Decision
Topic: Collection Protocols
Concrete type used today: Array<User>
Minimum capability needed: Collection / RandomAccessCollection / MutableCollection
Reason: Need indexing? mutation? insertion? repeated traversal?
Performance assumption: O(1) indexing required or not?
API stability: Can this stay generic without confusing callers?
```

Senior lens:

- Avoid accepting `Array` when any `Collection` works.
- Avoid over-generalizing if it makes call sites hard to read.
- Match protocol constraints to actual algorithm needs.
- Think about indexing cost before using `count`, `offsetBy`, or subscripts.

## More Coding Examples

### Example 1: Render Any Collection

```swift
func displayNames<C: Collection>(for users: C) -> [String]
where C.Element == User {
    users.map(\\.name)
}
```

This works for arrays and slices.

### Example 2: Require Random Access For Pagination

```swift
func item<C: RandomAccessCollection>(at offset: Int, in values: C) -> C.Element? {
    guard offset >= 0, offset < values.count else { return nil }
    let index = values.index(values.startIndex, offsetBy: offset)
    return values[index]
}
```

The constraint communicates the performance expectation.

## Common Mistakes

- Accepting `Array` when `Collection` is enough.
- Accepting `Sequence` when the algorithm needs multiple passes.
- Assuming all collections have integer indices.
- Assuming `count` and offset movement are equally cheap everywhere.
- Returning slices without considering retained storage.

## Interview Guide

Junior:

Collection protocols describe what operations a collection supports.

Mid-level:

Use `Sequence` for one-pass iteration, `Collection` for stable indexing/repeated traversal, and `RandomAccessCollection` when efficient index jumps matter.

Senior:

Collection protocols let APIs express algorithmic requirements. I choose the weakest useful constraint, but I do not make APIs generic just for style. I also think about index semantics and performance.

## Practice

1. Write a function that accepts any `Collection` of users.
2. Rewrite it to require `RandomAccessCollection` only when offset lookup is needed.
3. Explain why `String` collection indexing is different from array indexing.
4. Build a review checklist for collection protocol constraints.
