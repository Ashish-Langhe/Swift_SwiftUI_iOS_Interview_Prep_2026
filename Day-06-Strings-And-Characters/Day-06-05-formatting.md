# Day 6: Formatting

## Core Idea

Formatting converts values into user-readable text.

```swift
let name = "Aarav"
let message = "Hello, \(name)"
```

String interpolation is simple formatting.

## Number Formatting

```swift
let amount = 1234.5
let text = amount.formatted(.currency(code: "INR"))
```

## Date Formatting

```swift
let date = Date()
let display = date.formatted(date: .abbreviated, time: .shortened)
```

## Real iOS Use Cases

- Price labels
- Dates
- Percentages
- File sizes
- User-facing messages

## Interview Levels

Junior:

Use string interpolation for simple formatting.

Senior:

For user-facing values, prefer formatters or `.formatted()` APIs over manual string building, especially for dates, currency, locale, and pluralization.

## Quick Notes

- Interpolation is simple
- Use `.formatted()` for user-facing values
- Currency and dates depend on locale
- Avoid manual formatting for complex display

## Interview Depth

Junior answer: Formatting means converting values into readable text.

Mid-level answer: String interpolation is fine for simple debug or display text, but dates, numbers, percentages, and currency should use formatting APIs.

Senior answer: Formatting is localization-sensitive. Manually building `"Rs. \(amount)"` or `"MM/dd/yyyy"` can be wrong for many users. Prefer `FormatStyle`, `DateFormatter`, `NumberFormatter`, or localized resources depending on the deployment target and use case.

iOS use case:

```swift
let priceText = amount.formatted(.currency(code: "INR"))
let dateText = Date().formatted(date: .abbreviated, time: .shortened)
```

Common mistakes:

- Manual currency strings.
- Hardcoded date formats for user-facing UI.
- Formatting in many views instead of a view model/helper.

Practice:

1. Format currency.
2. Format date for display.
3. Explain interpolation vs formatter.
