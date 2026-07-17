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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Substrings is not only a syntax topic. In production Swift, it affects Unicode correctness, localization, formatting, and user-visible text safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Substrings in an app feature:

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

1. Write a minimal example that shows Substrings correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Substrings, but it shows the kind of production shape you should connect this topic to:

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

