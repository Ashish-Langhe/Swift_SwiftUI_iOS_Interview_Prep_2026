# Day 5: Collection Performance Basics

## Core Idea

Different collections have different performance characteristics.

| Operation | Array | Set | Dictionary |
| --- | --- | --- | --- |
| Keep order | Yes | No | No guaranteed business order |
| Allow duplicates | Yes | No | Keys no, values yes |
| Lookup by index | Fast | No | No |
| Lookup by key/value | Linear for value | Fast membership | Fast by key |

## Array

Good for ordered data.

```swift
let first = items[0]
```

Searching by value is usually linear.

```swift
items.contains(target)
```

## Set

Good for membership checks.

```swift
selectedIds.contains(id)
```

## Dictionary

Good for lookup by key.

```swift
let user = usersById[id]
```

## Copy-On-Write

Swift collections are value types optimized with copy-on-write.

```swift
var a = [1, 2, 3]
var b = a
b.append(4)
```

Swift avoids copying storage until mutation is needed.

## Modern Swift 6.x Notes

Swift 6.2 added `InlineArray` and `Span` for safe systems programming and fixed/contiguous memory scenarios. They are not replacements for normal collections in everyday iOS UI code, but senior candidates should know they exist.

## Interview Levels

Junior:

Use arrays for lists, sets for unique values, dictionaries for key-value data.

Mid-level:

Choose collections based on lookup, ordering, and uniqueness needs.

Senior:

Performance is about access patterns. Repeated `contains` on a large array may be worse than using a set. Repeated lookup by ID should usually use a dictionary. Avoid premature optimization, but choose the right collection for the data shape.

## Quick Notes

- Array preserves order
- Set gives fast membership
- Dictionary gives fast key lookup
- Swift collections use copy-on-write
- Modern low-level APIs include `InlineArray` and `Span`

## Interview Depth

Junior answer: Different collections are good at different things.

Mid-level answer: Arrays are good for order, sets are good for uniqueness and membership checks, and dictionaries are good for key lookup.

Senior answer: Performance depends on access pattern. A small array is fine for simple searches. But repeated membership checks against thousands of IDs should use a set. Repeated lookup by model ID should use a dictionary. Do not optimize blindly, but choose the structure that matches the operation.

iOS use case:

```swift
let visibleIds: [String] = users.map(\.id)
let usersById = Dictionary(uniqueKeysWithValues: users.map { ($0.id, $0) })
```

Common mistakes:

- Treating Big-O as the only performance factor.
- Optimizing before knowing data size.
- Ignoring sorting cost.
- Forgetting UI updates may dominate collection cost.

Practice:

1. Explain lookup cost for array vs dictionary.
2. Explain why `Set.contains` is useful for selected IDs.
3. Describe copy-on-write.
