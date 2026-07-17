# Day 3: Higher-Order Functions Introduction

## Core Idea

A higher-order function is a function that takes another function as input, returns a function, or both.

Swift collections provide common higher-order functions:

- `map`
- `filter`
- `reduce`
- `compactMap`
- `flatMap`
- `forEach`
- `sorted`

## Map

`map` transforms each element.

```swift
let numbers = [1, 2, 3]
let doubled = numbers.map { $0 * 2 }
```

Result:

```swift
[2, 4, 6]
```

## Filter

`filter` keeps elements that match a condition.

```swift
let numbers = [1, 2, 3, 4, 5]
let evenNumbers = numbers.filter { $0.isMultiple(of: 2) }
```

Result:

```swift
[2, 4]
```

## Reduce

`reduce` combines values into one result.

```swift
let prices = [100, 200, 300]
let total = prices.reduce(0) { partialResult, price in
    partialResult + price
}
```

Short form:

```swift
let total = prices.reduce(0, +)
```

## CompactMap

`compactMap` transforms values and removes nil results.

```swift
let strings = ["1", "two", "3"]
let numbers = strings.compactMap { Int($0) }
```

Result:

```swift
[1, 3]
```

## Sorted

```swift
let names = ["Meera", "Aarav", "Kabir"]
let sortedNames = names.sorted { $0 < $1 }
```

Short form:

```swift
let sortedNames = names.sorted()
```

## ForEach

```swift
let topics = ["Variables", "Functions", "Optionals"]

topics.forEach { topic in
    print(topic)
}
```

Use `forEach` for simple side effects. Use a normal `for` loop when you need `break`, `continue`, or early return from the outer scope.

## Real iOS Use Cases

### Mapping API Models To View Models

```swift
struct UserResponse {
    let id: Int
    let name: String
}

struct UserRowViewModel {
    let title: String
}

let responses = [
    UserResponse(id: 1, name: "Aarav"),
    UserResponse(id: 2, name: "Meera")
]

let rows = responses.map { response in
    UserRowViewModel(title: response.name)
}
```

### Filtering Search Results

```swift
let users = ["Aarav", "Meera", "Kabir"]
let query = "aa"

let results = users.filter {
    $0.localizedCaseInsensitiveContains(query)
}
```

### Calculating Total

```swift
let amounts: [Decimal] = [100, 250, 50]
let total = amounts.reduce(0, +)
```

### Parsing IDs

```swift
let rawIds = ["101", "abc", "202"]
let ids = rawIds.compactMap { Int($0) }
```

## Higher-Order Functions Vs Loops

Loop:

```swift
var names: [String] = []

for user in responses {
    names.append(user.name)
}
```

Higher-order function:

```swift
let names = responses.map { $0.name }
```

Use the style that is clearest. Higher-order functions are not automatically better.

## Common Mistakes

### Using Map For Side Effects

Avoid:

```swift
users.map { print($0) }
```

Prefer:

```swift
users.forEach { print($0) }
```

Or:

```swift
for user in users {
    print(user)
}
```

### Making Chains Too Clever

```swift
let result = users
    .filter { $0.isActive }
    .map { $0.name }
    .sorted()
```

This is fine. But if each closure becomes complex, extract named helpers.

## Junior-Level Interview Answer

A higher-order function is a function that takes another function or closure as a parameter, or returns one. Examples are `map`, `filter`, and `reduce`.

## Mid-Level Interview Answer

Higher-order functions make collection transformations concise. `map` transforms, `filter` selects, `reduce` combines, and `compactMap` transforms while removing nil.

## Senior-Level Interview Answer

Higher-order functions are tools for expressing data transformations declaratively. I use them when they improve clarity and keep transformations local. I avoid long chains with complex closures, avoid `map` for side effects, and consider performance when chaining over large collections. In UI code, mapping API models into view models is one of the most common and clean use cases.

## Quick Interview Notes

- `map` transforms.
- `filter` keeps matching values.
- `reduce` combines into one result.
- `compactMap` removes nil after transformation.
- `forEach` performs side effects.
- Use loops when control flow needs `break` or `continue`.

## Practice Questions

1. What is a higher-order function?
2. What is the difference between `map` and `filter`?
3. What does `reduce` do?
4. When should you use `compactMap`?
5. Why should you avoid `map` for side effects?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Higher-Order Functions Introduction is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while building service methods, validation helpers, and reusable transformation functions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Higher-Order Functions Introduction in an app feature:

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

1. Write a minimal example that shows Higher-Order Functions Introduction correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Higher-Order Functions Introduction, but it shows the kind of production shape you should connect this topic to:

```swift
struct PasswordRule {
    let message: String
    let validate: (String) -> Bool
}

func validatePassword(_ password: String, rules: [PasswordRule]) -> [String] {
    rules.compactMap { rule in
        rule.validate(password) ? nil : rule.message
    }
}

let rules = [
    PasswordRule(message: "Use at least 8 characters") { $0.count >= 8 },
    PasswordRule(message: "Add one number") { $0.contains(where: \.isNumber) }
]
```

This connects function design to reuse. The validation function has clear inputs and output, while each rule is injectable and easy to test.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Higher-Order Functions Introduction** is evaluated through this lens: functions are API design units; labels, return types, async/throws, and side effects define how other engineers use the code. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a function contract describing inputs, output, side effects, error behavior, actor isolation, and test examples
Topic: Higher-Order Functions Introduction
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine designing validation, formatting, repository, or service APIs that many features will call. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is functions that do too much, hide mutation, or make the call site read like implementation instead of intent.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Does the call site read naturally?
- Are side effects obvious?
- Can the function be tested without UI or network?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Higher-Order Functions Introduction**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Higher-Order Functions Introduction** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Formatter Function

```swift
func formattedName(first: String, last: String) -> String {
    "\(first) \(last)".trimmingCharacters(in: .whitespacesAndNewlines)
}

let title = formattedName(first: "Ashish", last: "Langhe")
```

A small function keeps formatting consistent across labels, cells, and detail screens.

### Example 2: Async Service Function

```swift
func loadProfileTitle(userID: UUID, service: ProfileService) async throws -> String {
    let profile = try await service.profile(id: userID)
    return formattedName(first: profile.firstName, last: profile.lastName)
}
```

This keeps async loading and formatting separate but composable.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Pass behavior as a value

```swift
let sortByName: (User, User) -> Bool = { $0.name < $1.name }
let sortedUsers = users.sorted(by: sortByName)
```

Function types let you inject behavior without creating a new concrete type.

### Why This Fits Higher-Order Functions Introduction

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

