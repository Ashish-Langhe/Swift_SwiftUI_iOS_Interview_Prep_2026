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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

String Indexing is not only a syntax topic. In production Swift, it affects Unicode correctness, localization, formatting, and user-visible text safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while display names, search fields, localized labels, and formatted dates or currency. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying String Indexing in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Prefer code that communicates intent at the call site, not only code that compiles.
- When a feature grows, revisit whether the type still owns the right responsibilities.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows String Indexing correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use String Indexing, but it shows the kind of production shape you should connect this topic to:

```swift
let name = "Jose\u{301}"
let normalized = name.precomposedStringWithCanonicalMapping

let greeting = String(localized: "profile.greeting \(normalized)")
let price = 19.99.formatted(.currency(code: "USD"))
```

String work is user-facing work. Unicode, formatting, and localization decide whether the app feels correct for real people, not just for ASCII test data.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **String Indexing** is evaluated through this lens: strings are user-facing data; senior engineers treat Unicode, formatting, and localization as correctness issues. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a text-handling checklist covering grapheme clusters, locale-aware formatting, search normalization, and localization keys
Topic: String Indexing
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine reviewing profile names, currency, dates, search, accessibility labels, and server-provided display text. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is ASCII assumptions, integer indexing, hard-coded text, and formatting that only works in one locale.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Does this work for composed characters?
- Is formatting locale-aware?
- Is user-visible text localizable?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **String Indexing**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **String Indexing** to code you might write in a SwiftUI/UIKit feature.

### Example 1: User-Visible Formatting

```swift
let amount = 1299.5
let displayAmount = amount.formatted(.currency(code: "USD"))
print(displayAmount)
```

Formatting should be delegated to Swift/Foundation instead of manually building user-visible strings.

### Example 2: Safe Character Count

```swift
let displayName = "Ana 👩‍💻"
let characterCount = displayName.count
print("Visible characters: \(characterCount)")
```

Swift `String.count` counts user-perceived characters, which matters for names, bios, and validation.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Small realistic usage

```swift
struct ExampleState: Equatable {
    var title: String
    var isEnabled: Bool
}

let state = ExampleState(title: "Continue", isEnabled: true)
print(state.title)
```

When studying the topic, rewrite this generic shape into the exact model your screen needs.

### Why This Fits String Indexing

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

