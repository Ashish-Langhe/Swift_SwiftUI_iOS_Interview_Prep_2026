# Day 3: Function Types

## Core Idea

Functions have types. A function type describes its parameter types and return type.

```swift
func add(_ a: Int, _ b: Int) -> Int {
    a + b
}

let operation: (Int, Int) -> Int = add
let result = operation(2, 3)
```

The type `(Int, Int) -> Int` means:

- Takes two `Int` values
- Returns an `Int`

## Function As A Value

You can store functions in variables or constants.

```swift
func multiply(_ a: Int, _ b: Int) -> Int {
    a * b
}

let mathOperation = multiply
print(mathOperation(4, 5))
```

## Function As A Parameter

You can pass a function into another function.

```swift
func perform(_ operation: (Int, Int) -> Int, a: Int, b: Int) -> Int {
    operation(a, b)
}

let result = perform(add, a: 10, b: 20)
```

This is a foundation for callbacks and higher-order functions.

## Function As A Return Value

```swift
func makeMultiplier(by factor: Int) -> (Int) -> Int {
    func multiplier(value: Int) -> Int {
        value * factor
    }

    return multiplier
}

let double = makeMultiplier(by: 2)
print(double(10))
```

## Optional Function Types

```swift
var completion: (() -> Void)?

completion = {
    print("Completed")
}

completion?()
```

This is common for callbacks.

## Typealias For Function Types

Function types can become hard to read. Use `typealias`.

```swift
typealias CompletionHandler = (Result<Data, Error>) -> Void

func loadData(completion: CompletionHandler) {
    completion(.success(Data()))
}
```

## Real iOS Use Cases

### Completion Handler

```swift
func fetchUser(completion: @escaping (String) -> Void) {
    completion("Aarav")
}
```

`@escaping` is needed when the closure may be called after the function returns.

### Button Action Concept

```swift
let onTap: () -> Void = {
    print("Button tapped")
}

onTap()
```

### Dependency Injection

```swift
struct LoginValidator {
    let isValidEmail: (String) -> Bool
}

let validator = LoginValidator { email in
    email.contains("@")
}
```

## Junior-Level Interview Answer

A function type defines what parameters a function accepts and what it returns.

```swift
(Int, Int) -> Int
```

## Mid-Level Interview Answer

Functions are first-class values in Swift. We can store them in variables, pass them as parameters, and return them from other functions. This enables callbacks, dependency injection, and flexible behavior.

## Senior-Level Interview Answer

Function types let behavior become data. This is powerful for composition, dependency injection, testability, and asynchronous APIs. I use type aliases when function signatures become complex. I also pay attention to escaping closures, capture lists, actor isolation, and retain cycles when function values are stored or called later.

## Quick Interview Notes

- Function types use `(Parameters) -> Return`.
- Functions can be stored in variables.
- Functions can be passed as arguments.
- Functions can be returned.
- Use `typealias` for readability.
- Stored callbacks often involve escaping closures.

## Practice Questions

1. What is the type of a function that takes `String` and returns `Bool`?
2. Can functions be stored in variables?
3. Why use a function as a parameter?
4. What is a callback?
5. When is `typealias` useful?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Function Types is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Function Types in an app feature:

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

1. Write a minimal example that shows Function Types correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Function Types, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Function Types** is evaluated through this lens: functions are API design units; labels, return types, async/throws, and side effects define how other engineers use the code. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Function Types
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

For **Function Types**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

