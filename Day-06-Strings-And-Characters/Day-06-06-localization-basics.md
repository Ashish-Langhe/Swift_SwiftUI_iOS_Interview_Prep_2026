# Day 6: Localization Basics

## Core Idea

Localization adapts text for different languages and regions.

Avoid hardcoding user-facing strings in production apps.

```swift
Text("profile.title")
```

With localized resources, the key maps to translated text.

## Why Localization Matters

Different locales affect:

- Text
- Dates
- Numbers
- Currency
- Plurals
- Text direction

## String Interpolation Risk

```swift
"Hello \(name), you have \(count) messages"
```

This may not translate well in all languages.

Prefer localized strings with placeholders and plural rules.

## Real iOS Use Cases

- App labels
- Error messages
- Empty states
- Notifications
- Accessibility text

## Modern Swift Notes

Apple platform localization tooling keeps evolving, and SwiftUI works well with localized string keys. Interviews care less about syntax and more about knowing not to concatenate translated sentences manually.

## Interview Levels

Junior:

Localization means supporting multiple languages.

Senior:

Localization includes language, pluralization, formatting, and cultural expectations. I avoid string concatenation and use localized resources for user-facing text.

## Quick Notes

- Do not hardcode production user-facing text
- Avoid manual sentence concatenation
- Format dates/currency by locale
- Consider pluralization

## Interview Depth

Junior answer: Localization means adapting app text for different languages.

Mid-level answer: Localized apps should not hardcode user-facing strings. They should use localized resources, format dates and currency by locale, and handle pluralization.

Senior answer: Localization is not just translation. Sentence order, plural rules, gender, text direction, and cultural formatting differ across regions. Concatenating localized fragments is a common bug because grammar may require a different order.

iOS use case:

```swift
Text("profile.title")
```

The key should map to localized text in string resources.

Common mistakes:

- Concatenating translated fragments.
- Hardcoding English error messages.
- Ignoring pluralization.
- Forgetting accessibility strings.

Practice:

1. Explain why string concatenation is risky.
2. Identify user-facing strings in a screen.
3. Explain localization vs formatting.
