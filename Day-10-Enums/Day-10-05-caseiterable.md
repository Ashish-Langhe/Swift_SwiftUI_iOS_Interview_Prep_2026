# Day 10: CaseIterable

## Core Idea

`CaseIterable` provides all enum cases.

```swift
enum Filter: CaseIterable {
    case all
    case active
    case completed
}

let filters = Filter.allCases
```

## Real iOS Use Cases

- Segmented controls
- Filter chips
- Settings options
- Picker values

## Interview Levels

Junior:

`CaseIterable` lets you access all enum cases.

Senior:

It is useful for static UI options, but not available automatically for enums with associated values.

## Quick Notes

- Adds `allCases`
- Great for pickers
- Works best with simple enums

## Interview Depth

Junior answer: `CaseIterable` gives access to all enum cases.

Mid-level answer: It is useful for static UI options like filters, tabs, sort modes, and pickers.

Senior answer: `CaseIterable` is excellent when the set is static and case order is meaningful for UI. For associated-value enums, automatic `allCases` is not available because the possible values are not finite in the same simple way.

iOS use case:

```swift
enum SortMode: String, CaseIterable, Identifiable {
    case newest
    case oldest
    case highestAmount

    var id: String { rawValue }
}
```

Common mistakes:

- Expecting automatic allCases for associated values.
- Depending on case order without documenting it.
- Showing raw values directly without display text.

Practice:

1. Build filter enum with `CaseIterable`.
2. Use it in SwiftUI `ForEach`.
3. Explain limitation with associated values.
