# Day 3: Parameters And Return Values

## Core Idea

Parameters are inputs to a function. Return values are outputs from a function.

```swift
func greet(name: String) -> String {
    "Hello, \(name)"
}

let message = greet(name: "Aarav")
```

Here:

- `name` is a parameter
- `"Aarav"` is an argument
- `String` after `->` is the return type
- `message` receives the returned value

## Function With Parameters

```swift
func printUser(name: String, age: Int) {
    print("\(name) is \(age) years old")
}

printUser(name: "Meera", age: 28)
```

Parameters let the same function work with different values.

## Function With Return Value

```swift
func add(_ first: Int, _ second: Int) -> Int {
    first + second
}

let result = add(10, 20)
```

The return type must match the returned value.

## Multiple Return Paths

```swift
func passStatus(score: Int) -> String {
    if score >= 50 {
        return "Pass"
    } else {
        return "Fail"
    }
}
```

Every possible path must return a `String`.

## Returning Optionals

Use optional returns when a function may not produce a value.

```swift
func firstCharacter(in text: String) -> Character? {
    text.first
}

let character = firstCharacter(in: "Swift")
```

This is better than returning a fake placeholder.

## Returning Tuples

Use tuples for small, related return values.

```swift
func minMax(numbers: [Int]) -> (min: Int, max: Int)? {
    guard let first = numbers.first else {
        return nil
    }

    var minValue = first
    var maxValue = first

    for number in numbers {
        if number < minValue { minValue = number }
        if number > maxValue { maxValue = number }
    }

    return (minValue, maxValue)
}
```

Use a struct when the return value is part of your domain and will grow over time.

## Returning Result

For success/failure APIs, `Result` can be useful.

```swift
enum LoginError: Error {
    case invalidEmail
}

func validate(email: String) -> Result<String, LoginError> {
    guard email.contains("@") else {
        return .failure(.invalidEmail)
    }

    return .success(email)
}
```

Throwing functions are often more idiomatic for error propagation, but `Result` is useful for storing or passing outcomes.

## Real iOS Use Cases

### Validation

```swift
func isValidEmail(_ email: String) -> Bool {
    email.contains("@") && email.contains(".")
}
```

### Formatting Currency

```swift
func formattedPrice(amount: Decimal, currencyCode: String) -> String {
    "\(currencyCode) \(amount)"
}
```

### Building URL

```swift
func userURL(baseURL: URL, userId: String) -> URL {
    baseURL.appendingPathComponent("users").appendingPathComponent(userId)
}
```

## Junior-Level Interview Answer

Parameters are values passed into a function. Return values are values sent back from a function.

```swift
func double(_ number: Int) -> Int {
    number * 2
}
```

## Mid-Level Interview Answer

Parameters define what a function needs to do its job. Return values define what the caller receives. I use optional return types when a value may not exist, tuples for small grouped results, and custom structs for meaningful domain results.

## Senior-Level Interview Answer

Function signatures are API design. The parameter list should reveal dependencies and the return type should model the outcome accurately. Returning `nil`, throwing an error, returning `Result`, or returning a domain object are different design choices. I choose based on whether failure is expected, recoverable, and meaningful to the caller.

## Quick Interview Notes

- Parameters are inputs.
- Arguments are actual values passed during the call.
- Return values are outputs.
- `-> Type` declares the return type.
- Optional returns represent possible absence.
- Tuples are fine for small related values.
- Domain-heavy returns should usually be structs or enums.

## Practice Questions

1. What is the difference between a parameter and an argument?
2. How do you declare a return type?
3. When should a function return an optional?
4. When is a tuple return useful?
5. What makes a function signature well-designed?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Parameters And Return Values is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Parameters And Return Values in an app feature:

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

1. Write a minimal example that shows Parameters And Return Values correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Parameters And Return Values, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Parameters And Return Values** is evaluated through this lens: functions are API design units; labels, return types, async/throws, and side effects define how other engineers use the code. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Parameters And Return Values
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

For **Parameters And Return Values**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Parameters And Return Values** to code you might write in a SwiftUI/UIKit feature.

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

### Why This Fits Parameters And Return Values

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

