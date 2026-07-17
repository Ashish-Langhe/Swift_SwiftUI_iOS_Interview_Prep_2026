# Day 6: Substrings

## Core Idea

A `Substring` is a slice of a `String`.

```swift
let text = "Hello Swift"
let spaceIndex = text.firstIndex(of: " ")!
let firstWord = text[..<spaceIndex]
```

`firstWord` is a `Substring`.

## Convert To String

```swift
let word = String(firstWord)
```

Convert when storing long-term.

## Why Substring Exists

Substrings can share storage with the original string for efficiency.

This is good for temporary parsing, but not ideal for long-term storage if the original string is large.

## Real iOS Use Cases

- Parsing names
- Extracting URL path parts
- Search snippets
- Formatting previews

## Interview Levels

Junior:

Substring is part of a string.

Senior:

Substring can share storage with the original string, so convert to `String` when storing beyond a short operation.

## Quick Notes

- Slicing creates `Substring`
- Convert with `String(substring)`
- Good for temporary parsing
- Avoid keeping tiny substrings from huge strings long-term

## Interview Depth

Junior answer: A substring is part of a string.

Mid-level answer: Swift slicing returns `Substring`, not `String`, because it can share storage with the original string for efficiency.

Senior answer: Shared storage is great for short-lived parsing, but if you store a small substring from a huge original string, the original storage may stay alive. Convert to `String` for long-term storage.

iOS use case:

```swift
func firstWord(in text: String) -> String? {
    guard let word = text.split(separator: " ").first else { return nil }
    return String(word)
}
```

Common mistakes:

- Storing `Substring` in models.
- Force-unwrapping indices while slicing.
- Forgetting conversion to `String`.

Practice:

1. Extract first word.
2. Explain why substring can share memory.
3. Convert substring to string.
