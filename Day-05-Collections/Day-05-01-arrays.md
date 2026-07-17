# Day 5: Arrays

## Core Idea

An `Array` stores ordered values of the same type.

```swift
let topics = ["Arrays", "Dictionaries", "Sets"]
var scores = [90, 85, 92]
```

Arrays preserve order and allow duplicate values.

```swift
let numbers = [1, 1, 2, 3]
```

## Common Operations

```swift
var names = ["Aarav", "Meera"]

names.append("Kabir")
names.insert("Riya", at: 1)
names.remove(at: 0)

let first = names.first
let count = names.count
let isEmpty = names.isEmpty
```

Array indexing can crash if the index is invalid.

```swift
if names.indices.contains(1) {
    print(names[1])
}
```

## Real iOS Use Cases

- Table view or SwiftUI list data
- Search results
- Transaction history
- Form fields
- Ordered navigation steps

```swift
struct Transaction {
    let title: String
    let amount: Decimal
}

let transactions = [
    Transaction(title: "Food", amount: 250),
    Transaction(title: "Travel", amount: 1200)
]
```

## Modern Swift 6.x Notes

Swift 6.2 introduced `InlineArray`, a fixed-size array-like storage feature for low-level and performance-sensitive code. Most iOS app code should continue using normal `Array`; mention `InlineArray` in interviews only as a modern systems/performance feature.

## Interview Levels

Junior:

An array stores multiple values in order.

Mid-level:

Arrays are ordered, index-based collections. They are good when order matters or duplicates are allowed.

Senior:

Arrays are excellent for ordered data and UI lists, but membership checks are linear unless additional indexing is used. For uniqueness or frequent lookup, a `Set` or `Dictionary` may be better.

## Quick Notes

- Ordered collection
- Allows duplicates
- Zero-based indexing
- Invalid index access crashes
- Best for ordered UI data

## Interview Depth

Junior answer: An array is an ordered list of values of the same type. You use it when you need to keep items in sequence.

Mid-level answer: Arrays are useful for UI lists, ordered API results, navigation steps, and any data where duplicate values are allowed. Access by index is fast, but searching by value requires scanning.

Senior answer: Arrays are often the right structure for ordered rendering, but they are not always the right structure for lookup. If code repeatedly calls `contains` on a large array, a `Set` may be better. If code repeatedly searches objects by ID, a `Dictionary` may be better. In production, I choose arrays for order and combine them with dictionaries when I need both order and fast lookup.

iOS use case:

```swift
struct FeedViewModel {
    private(set) var posts: [Post] = []

    mutating func replacePosts(_ newPosts: [Post]) {
        posts = newPosts
    }
}
```

Common mistakes:

- Force-indexing without checking bounds.
- Using arrays for uniqueness.
- Using arrays for repeated ID lookup.
- Mutating arrays from multiple concurrent contexts without isolation.

Practice:

1. Build an array of transactions and return only the first three.
2. Safely access the fifth element.
3. Explain why an array is good for a table view data source.
