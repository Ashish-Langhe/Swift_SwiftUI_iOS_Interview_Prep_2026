# Day 4: Optional Pattern Matching

## Core Idea

Because optional is conceptually an enum, you can pattern match it.

```swift
let name: String? = "Aarav"

switch name {
case .some(let value):
    print(value)
case .none:
    print("No name")
}
```

Most daily code uses `if let`, but optional pattern matching is useful in advanced control flow.

## If Case With Optionals

```swift
let score: Int? = 90

if case .some(let value) = score {
    print(value)
}
```

Equivalent common style:

```swift
if let score {
    print(score)
}
```

## Optional Pattern Shorthand

Swift supports a pattern shorthand:

```swift
let score: Int? = 90

if case let value? = score {
    print(value)
}
```

`value?` means match `.some(value)`.

## For Case With Optionals

```swift
let values: [Int?] = [1, nil, 3, nil, 5]

for case let value? in values {
    print(value)
}
```

This iterates only non-nil values.

Often, `compactMap` is clearer:

```swift
for value in values.compactMap({ $0 }) {
    print(value)
}
```

## Switch With Optional And Conditions

```swift
let age: Int? = 20

switch age {
case .some(let value) where value >= 18:
    print("Adult")
case .some:
    print("Minor")
case .none:
    print("Unknown age")
}
```

## Real iOS Use Cases

### Optional View State Data

```swift
let selectedUserId: String? = "user-101"

switch selectedUserId {
case .some(let id):
    print("Load user \(id)")
case .none:
    print("Show empty selection")
}
```

### Filtering Optional API Values

```swift
let rawPrices: [Decimal?] = [100, nil, 250]

for case let price? in rawPrices {
    print("Price: \(price)")
}
```

### Combining Enum And Optional

```swift
enum ScreenState {
    case loaded(userId: String?)
    case loading
}

let state = ScreenState.loaded(userId: "abc")

switch state {
case .loaded(let userId?):
    print("Loaded user \(userId)")
case .loaded(nil):
    print("Loaded without user")
case .loading:
    print("Loading")
}
```

## Junior-Level Interview Answer

Optional pattern matching means checking whether an optional is `.some` or `.none` using `switch` or pattern syntax.

## Mid-Level Interview Answer

Optional pattern matching is useful when optionals are part of a larger pattern, such as switching with conditions or iterating through arrays of optional values.

## Senior-Level Interview Answer

Optional pattern matching is powerful but should be used for clarity, not cleverness. In normal code, `if let` is usually clearer. I use optional patterns when they compose naturally with enum states, tuple matching, `where` clauses, or `for case` filtering.

## Quick Interview Notes

- Optional is pattern matchable as `.some` and `.none`.
- `if let` is usually simpler for basic unwrapping.
- `for case let value?` iterates non-nil optional values.
- `where` can add conditions to optional patterns.
- Optional pattern matching is useful with enums and complex state.

## Practice Questions

1. How can you switch over an optional?
2. What does `.some` mean?
3. What does `.none` mean?
4. What does `for case let value?` do?
5. When is optional pattern matching better than `if let`?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Optional Pattern Matching is not only a syntax topic. In production Swift, it affects absence modeling, safe unwrapping, nil propagation, and avoiding crash-prone assumptions. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while decoding API payloads, optional user profile fields, and view state that may not exist yet. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Optional Pattern Matching in an app feature:

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

- Ask whether nil is a valid business state, an incomplete loading state, or a data-quality bug.
- Prefer explicit state enums when multiple optional values must stay consistent.

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

1. Write a minimal example that shows Optional Pattern Matching correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Optional Pattern Matching, but it shows the kind of production shape you should connect this topic to:

```swift
struct ProfileResponse: Decodable {
    let id: String
    let displayName: String?
    let avatarURL: URL?
}

struct ProfileViewState {
    let title: String
    let avatarURL: URL?

    init(response: ProfileResponse) {
        let trimmedName = response.displayName?.trimmingCharacters(in: .whitespacesAndNewlines)
        self.title = trimmedName?.isEmpty == false ? trimmedName! : "Guest"
        self.avatarURL = response.avatarURL
    }
}
```

This shows optional thinking in UI code: not every missing value is an error. Some nil values become fallback UI, while others should become validation or decoding failures.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Optional Pattern Matching** is evaluated through this lens: optionals are absence modeling; senior engineers distinguish valid absence from corrupted data and incomplete loading. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: an optional-decision matrix showing nil-as-valid, nil-as-loading, nil-as-error, and nil-as-not-yet-requested
Topic: Optional Pattern Matching
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine mapping API responses into view state where profile image, display name, token, or cached data may be missing. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is force unwraps, chains that silently hide bugs, and multiple optionals that should be one explicit enum state.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- What does nil mean here?
- Should this be an enum instead?
- Can the UI explain or recover from absence?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Optional Pattern Matching**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

