# Day 6: Strings And Characters Interview Guide

## One-Minute Interview Answer

Swift `String` is a Unicode-correct collection of `Character` values. A `Character` represents a user-perceived grapheme cluster, so one character may be made of multiple Unicode scalars. Because of this, Swift uses `String.Index` instead of integer indexing. Substrings can share storage with the original string, so I convert them to `String` when storing. For user-facing text, I use localization and formatting APIs instead of manual concatenation.

## Modern Swift 6.x Notes

Swift 6.2 expanded Embedded Swift with full `String` APIs. Swift 6.3 also allows using module selectors like `Swift::` for disambiguating APIs, including String processing library APIs.

## Junior Questions

What is `String`?

Text.

What is `Character`?

One user-visible character.

## Senior Questions

Why no integer indexing?

Unicode characters have variable length, so integer offsets can be expensive and unsafe.

Why is localization more than translation?

It includes pluralization, formatting, region, currency, date, and layout expectations.

## Common Traps

- Treating strings like arrays of bytes
- Assuming emoji length is simple
- Manual localization
- Keeping substrings long-term
- Using `String.count` for backend byte limits

## Topic-By-Topic Deep Dive

### String Vs Character

```swift
let word: String = "Swift"
let letter: Character = "S"
```

Senior note:

`Character` means user-perceived character, not byte. This matters for emoji, accents, and non-English text.

### Unicode And Grapheme Clusters

```swift
let family = "👨‍👩‍👧‍👦"
print(family.count) // 1
```

Senior answer:

Swift optimizes for correctness from the user's point of view. Backend storage limits may use bytes, so UI validation and server validation may need different measurements.

### String Indexing

```swift
let text = "café"
let first = text[text.startIndex]
```

Swift does not allow `text[0]` because character boundaries are not fixed-width.

### Substrings

```swift
let prefix = text.prefix(2)
let stored = String(prefix)
```

Senior note:

`Substring` can share storage with the original string. Convert to `String` when storing long-term.

### Formatting

```swift
let amount = 1234.5
let display = amount.formatted(.currency(code: "INR"))
```

Senior note:

Avoid manual date and currency formatting. User-facing formatting should respect locale.

### Localization

Localization affects more than translation:

- Word order
- Plurals
- Currency
- Dates
- Text direction
- Cultural expectations

Senior note:

Do not concatenate localized sentence fragments. Use localized templates and plural rules.

### Interview Traps

Best senior answer:

Swift string complexity is intentional. The language makes Unicode correctness explicit instead of giving fast but incorrect ASCII-style shortcuts.

## Production Decision Table

| Need | Tool |
| --- | --- |
| Simple display | String interpolation |
| Currency/date/number | `.formatted()` / formatter |
| Character iteration | `for character in string` |
| Safe indexing | `String.Index` |
| Long-term slice storage | Convert `Substring` to `String` |
| Multi-language UI | Localization resources |

## Final Revision

- `String` is Unicode-aware
- `Character` is a grapheme cluster
- Use `String.Index`
- Convert `Substring` for storage
- Use `.formatted()` and localization
