# Day 6: Common String Interview Traps

## Trap 1: Assuming Integer Indexing

Wrong:

```swift
// text[0]
```

Swift uses `String.Index`.

## Trap 2: Assuming Count Equals Bytes

```swift
let emoji = "👨‍👩‍👧‍👦"
print(emoji.count) // 1
```

Storage is more complex.

## Trap 3: Keeping Substrings Too Long

Convert to `String` for long-term storage.

## Trap 4: Manual Date And Currency Formatting

Use formatting APIs.

## Trap 5: Concatenating Localized Text

Sentence order differs across languages.

## Interview Levels

Junior:

Swift strings are Unicode-aware.

Senior:

Most string bugs happen when code assumes ASCII-like behavior. Swift intentionally makes string indexing more explicit to preserve correctness.

## Quick Notes

- No integer indexing
- `count` is grapheme clusters
- Use formatters
- Localize properly
- Convert long-lived substrings

## Interview Depth

Junior answer: The most common trap is treating Swift strings like simple arrays of characters.

Mid-level answer: Swift strings are Unicode-aware, so indexing, counting, and slicing require care. You should also avoid manual formatting and hardcoded localized text.

Senior answer: String bugs often appear only in international markets. A senior iOS engineer designs text handling with Unicode, localization, accessibility, formatting, and backend constraints in mind from the start.

iOS use case:

```swift
func canSubmit(username: String) -> Bool {
    let trimmed = username.trimmingCharacters(in: .whitespacesAndNewlines)
    return (3...20).contains(trimmed.count)
}
```

Common mistakes:

- Validating before trimming.
- Counting bytes when UI wants characters.
- Counting characters when server wants bytes.

Practice:

1. Explain why `emoji.count` may surprise people.
2. Explain why `Substring` exists.
3. Explain why localization affects sentence construction.
