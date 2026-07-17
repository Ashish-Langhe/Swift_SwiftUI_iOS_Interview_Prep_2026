# Day 5: Sets

## Core Idea

A `Set` stores unique values with no guaranteed order.

```swift
var skills: Set<String> = ["Swift", "SwiftUI", "UIKit"]
skills.insert("Combine")
skills.insert("Swift")
```

The second `"Swift"` is ignored because sets keep unique values.

## Common Operations

```swift
let hasSwift = skills.contains("Swift")
skills.remove("UIKit")
```

Set algebra:

```swift
let iosSkills: Set = ["Swift", "UIKit", "SwiftUI"]
let backendSkills: Set = ["Swift", "Vapor", "SQL"]

let common = iosSkills.intersection(backendSkills)
let all = iosSkills.union(backendSkills)
let iosOnly = iosSkills.subtracting(backendSkills)
```

## Real iOS Use Cases

- Selected IDs
- Unique tags
- Deduplication
- Permission flags
- Fast membership checks

```swift
var selectedTransactionIds: Set<String> = []
selectedTransactionIds.insert("txn-1")
```

## Modern Swift 6.x Notes

Like dictionaries, sets rely on hashing. In concurrent Swift, prefer immutable set snapshots when sharing across tasks, or protect mutable shared sets with actor isolation.

## Interview Levels

Junior:

A set stores unique values.

Mid-level:

Sets are useful when duplicates are not allowed and membership checks are common.

Senior:

Sets are a good fit for identity membership and deduplication. I avoid sets when order matters or when duplicate counts are meaningful.

## Quick Notes

- Unique values
- No guaranteed order
- Fast `contains`
- Values must be `Hashable`
- Good for selection and deduplication

## Interview Depth

Junior answer: A set stores unique values.

Mid-level answer: Sets are useful when duplicates should not exist and when `contains` is a frequent operation. They do not preserve order.

Senior answer: Sets are excellent for selection state, permission flags, deduplication, and membership checks. If UI order matters, convert to a sorted array or maintain a separate ordered structure. A set communicates intent better than an array when uniqueness is part of the domain rule.

iOS use case:

```swift
struct SelectionState {
    private(set) var selectedIds: Set<String> = []

    mutating func toggle(_ id: String) {
        if selectedIds.contains(id) {
            selectedIds.remove(id)
        } else {
            selectedIds.insert(id)
        }
    }
}
```

Common mistakes:

- Expecting insertion order.
- Using set when duplicates matter.
- Forgeting custom types must be `Hashable`.

Practice:

1. Remove duplicate tags from an array.
2. Track selected row IDs.
3. Explain when a set beats an array.
