# Day 6: String Vs Character

## Core Idea

`String` stores text. `Character` stores one user-perceived character.

```swift
let name: String = "Swift"
let grade: Character = "A"
```

A `String` is a collection of `Character` values.

```swift
for character in name {
    print(character)
}
```

## Important Difference

A Swift `Character` can be more than one Unicode scalar.

```swift
let flag: Character = "🇮🇳"
```

It looks like one character to the user, even though it is internally more complex.

## Real iOS Use Cases

- Display names
- Input validation
- Search text
- Localization
- Formatting labels

## Modern Swift 6.x Notes

Swift 6.2 improved Embedded Swift by including full `String` APIs, making String behavior more consistently available in constrained environments.

## Interview Levels

Junior:

String is text, Character is a single character.

Senior:

Swift strings are Unicode-correct. A character is a user-perceived grapheme cluster, not necessarily one byte or one Unicode scalar.

## Quick Notes

- `String` is text
- `Character` is one user-perceived character
- Unicode makes character handling non-trivial
- Do not assume one character equals one byte

## Interview Depth

Junior answer: `String` is used for text, and `Character` is used for one character.

Mid-level answer: A Swift `String` is a collection of `Character` values, but those characters are Unicode-aware. This means emoji and accented characters are handled as users expect.

Senior answer: Swift's `Character` represents an extended grapheme cluster. This design avoids many internationalization bugs but makes string indexing more explicit. Never assume `Character`, Unicode scalar, byte, and visible glyph all mean the same thing.

iOS use case:

```swift
func initials(from name: String) -> String {
    name
        .split(separator: " ")
        .compactMap(\.first)
        .map(String.init)
        .joined()
}
```

Common mistakes:

- Treating strings as byte arrays.
- Assuming emoji count is simple.
- Using `Character` for values that should be `String`.

Practice:

1. Iterate every character in a username.
2. Explain why `"🇮🇳"` can be one `Character`.
3. Describe `String` vs `Character` in one sentence.
