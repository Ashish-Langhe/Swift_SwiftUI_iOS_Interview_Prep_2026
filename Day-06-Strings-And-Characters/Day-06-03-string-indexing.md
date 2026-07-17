# Day 6: String Indexing

## Core Idea

Swift strings do not use integer indexing.

```swift
let text = "Swift"
let first = text[text.startIndex]
```

This is because characters can have variable storage lengths.

## Moving Indices

```swift
let secondIndex = text.index(after: text.startIndex)
let second = text[secondIndex]
```

Index with offset:

```swift
let index = text.index(text.startIndex, offsetBy: 2)
print(text[index])
```

## Safe Indexing

Avoid offsets without checking length.

```swift
if text.count > 2 {
    let index = text.index(text.startIndex, offsetBy: 2)
    print(text[index])
}
```

## Real iOS Use Cases

- Masking phone numbers
- Extracting initials
- Text validation
- Search highlighting

## Interview Levels

Junior:

Use `startIndex` and `index(after:)` to access string characters.

Senior:

Swift avoids integer string indexing because Unicode makes random character access non-constant and potentially unsafe. Convert to array only when the tradeoff is acceptable.

## Quick Notes

- No `text[0]`
- Use `String.Index`
- Unicode drives this design
- Index carefully

## Interview Depth

Junior answer: Swift strings use `String.Index` instead of integer indexes.

Mid-level answer: You access characters using `startIndex`, `endIndex`, `index(after:)`, or `index(_:offsetBy:)`. This keeps access safe with Unicode text.

Senior answer: Integer indexing would imply constant-time random access, which is not always true for Unicode-correct strings. Swift makes string traversal explicit so developers do not accidentally split grapheme clusters or write ASCII-only logic.

iOS use case:

```swift
func firstInitial(from name: String) -> String? {
    guard let first = name.first else { return nil }
    return String(first)
}
```

Common mistakes:

- Trying `text[0]`.
- Offsetting beyond `endIndex`.
- Converting to `[Character]` for every operation without considering cost.

Practice:

1. Get the first character safely.
2. Get the third character only when it exists.
3. Explain why Swift indexing looks verbose.
