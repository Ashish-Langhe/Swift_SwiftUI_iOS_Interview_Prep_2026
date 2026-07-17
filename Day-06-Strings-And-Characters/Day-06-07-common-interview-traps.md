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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Common String Interview Traps is not only a syntax topic. In production Swift, it affects Unicode correctness, localization, formatting, and user-visible text safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Common String Interview Traps in an app feature:

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

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.

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

1. Write a minimal example that shows Common String Interview Traps correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Common String Interview Traps, but it shows the kind of production shape you should connect this topic to:

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

