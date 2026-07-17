# Day 6: Unicode And Grapheme Clusters

## Core Idea

Swift strings are Unicode-aware. A displayed character may be made of multiple Unicode scalars.

```swift
let cafe = "café"
print(cafe.count)
```

Swift counts user-perceived characters, not bytes.

## Grapheme Cluster

A grapheme cluster is what a user sees as one character.

```swift
let emoji = "👨‍👩‍👧‍👦"
print(emoji.count) // 1
```

Internally this is made from multiple scalars.

## Why This Matters

Do not assume:

- character count equals byte count
- string index can be an integer
- emoji length is simple

## Real iOS Use Cases

- Username length limits
- SMS character counts
- Text field validation
- Truncation
- Localization

## Interview Levels

Junior:

Swift supports Unicode text.

Senior:

Swift `String.count` counts extended grapheme clusters, which is correct for users but may differ from storage size or server byte limits.

## Quick Notes

- Unicode is global text representation
- Grapheme cluster is user-perceived character
- Emoji can be one character but many scalars
- Be careful with backend byte limits

## Interview Depth

Junior answer: Unicode lets Swift support text from many languages and emoji.

Mid-level answer: Swift counts user-perceived characters, called extended grapheme clusters. A single displayed character may be made from multiple Unicode scalars.

Senior answer: This affects validation and storage. A UI character limit should often use `String.count`, while a backend byte limit should measure encoded bytes. A good iOS engineer knows these are different.

iOS use case:

```swift
func isDisplayNameValid(_ name: String) -> Bool {
    (2...30).contains(name.count)
}
```

Common mistakes:

- Using `count` as byte length.
- Breaking emoji by slicing incorrectly.
- Assuming all languages behave like English.

Practice:

1. Explain grapheme cluster.
2. Explain why server byte limit differs from UI character limit.
3. Test a name with accents or emoji.
