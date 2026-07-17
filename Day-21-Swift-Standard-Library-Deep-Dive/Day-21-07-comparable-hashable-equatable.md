# Day 21: Comparable, Hashable, Equatable

## Why These Protocols Matter

`Equatable`, `Hashable`, and `Comparable` define equality, identity-like hashing, and ordering behavior.

They affect:

- `==`
- `Set`
- Dictionary keys
- Sorting
- Diffing
- SwiftUI identity patterns
- Tests

## Equatable

`Equatable` means values can be compared for equality.

```swift
struct User: Equatable {
    let id: String
    let name: String
}

if oldUser == newUser {
    print("No change")
}
```

Swift can synthesize conformance when all stored properties are equatable.

## Hashable

`Hashable` means a value can be used in sets and dictionary keys.

```swift
struct ProductID: Hashable {
    let rawValue: String
}

var selectedIDs: Set<ProductID> = []
```

Hashable includes Equatable.

Important rule:

If two values are equal, they must produce the same hash.

## Comparable

`Comparable` defines ordering.

```swift
struct AppVersion: Comparable {
    let major: Int
    let minor: Int

    static func < (lhs: AppVersion, rhs: AppVersion) -> Bool {
        (lhs.major, lhs.minor) < (rhs.major, rhs.minor)
    }
}
```

Then:

```swift
let versions = [AppVersion(major: 2, minor: 0), AppVersion(major: 1, minor: 5)]
let sorted = versions.sorted()
```

## Identity vs Equality

Be careful with models:

```swift
struct User: Hashable {
    let id: String
    var name: String
}
```

Should equality include `name`, or only `id`?

For many domain models, full value equality is good.

For identity collections, ID-only equality may be needed.

```swift
struct UserIdentity: Hashable {
    let id: String
}
```

Senior engineers make this explicit.

## Real iOS Use Cases

- `Set` of selected IDs.
- Dictionary keyed by IDs.
- Sorting products by price.
- Testing expected states.
- Diffable data source identifiers.
- SwiftUI `ForEach` identity.

```swift
struct RowID: Hashable {
    let rawValue: String
}
```

Prefer stable IDs for UI identity.

## Senior iOS Engineer Artifact

```text
Artifact: Equality And Identity Contract
Topic: Comparable, Hashable, Equatable
Type: User / Product / Row / Version
Equality meaning: all fields or stable identity?
Hashing fields: must match equality
Ordering meaning: alphabetical, date, priority, semantic version
UI impact: diffing, selection, ForEach, reload behavior
Tests: equal, not equal, same ID changed data, sorted order
```

Senior lens:

- Equality is a product/domain decision, not just a compiler feature.
- Hashing must be consistent with equality.
- Sorting must be stable and meaningful to the user.
- UI identity should not change when display text changes.

## More Coding Examples

### Example 1: Selected IDs

```swift
struct Product: Identifiable {
    let id: String
    let title: String
}

var selectedProductIDs: Set<Product.ID> = []
selectedProductIDs.insert("p1")
```

### Example 2: Sort By Date

```swift
struct Message: Comparable {
    let sentAt: Date

    static func < (lhs: Message, rhs: Message) -> Bool {
        lhs.sentAt < rhs.sentAt
    }
}
```

## Common Mistakes

- Letting synthesized equality include fields that should not define identity.
- Using mutable values as set members in confusing ways.
- Violating hash/equality consistency.
- Sorting by localized strings without considering locale.
- Using array indices as stable IDs in UI.

## Interview Guide

Junior:

`Equatable` supports `==`, `Hashable` supports sets/dictionary keys, and `Comparable` supports sorting.

Mid-level:

Synthesized conformances are convenient, but you must know what fields define equality and ordering.

Senior:

Equality, hashing, and ordering are domain contracts. I define them according to identity, diffing, and user-visible behavior, then test edge cases.

## Practice

1. Make a model equatable.
2. Use a custom ID type as a dictionary key.
3. Sort messages by date.
4. Explain when synthesized equality is wrong.
