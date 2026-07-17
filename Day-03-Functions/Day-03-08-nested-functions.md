# Day 3: Nested Functions

## Core Idea

A nested function is a function declared inside another function.

```swift
func makeGreeting(name: String) -> String {
    func prefix() -> String {
        "Hello"
    }

    return "\(prefix()), \(name)"
}
```

The nested function is available only inside the outer function.

## Why Use Nested Functions?

Use nested functions when helper logic is needed only inside one function.

```swift
func validatePassword(_ password: String) -> Bool {
    func hasMinimumLength() -> Bool {
        password.count >= 8
    }

    func hasNumber() -> Bool {
        password.contains { $0.isNumber }
    }

    return hasMinimumLength() && hasNumber()
}
```

The helpers are not visible outside `validatePassword`.

## Capturing Values

Nested functions can use values from the outer function.

```swift
func makeCounter(start: Int) -> () -> Int {
    var count = start

    func next() -> Int {
        count += 1
        return count
    }

    return next
}

let counter = makeCounter(start: 0)
print(counter())
print(counter())
```

The nested function captures `count`.

## Real iOS Use Cases

### Local Validation Helpers

```swift
func canSubmit(email: String, password: String) -> Bool {
    func isEmailValid() -> Bool {
        email.contains("@")
    }

    func isPasswordValid() -> Bool {
        password.count >= 8
    }

    return isEmailValid() && isPasswordValid()
}
```

### Local Formatting Helper

```swift
func profileSummary(name: String, city: String?) -> String {
    func cityText() -> String {
        city ?? "Unknown city"
    }

    return "\(name), \(cityText())"
}
```

### Recursive Helper

```swift
func countdown(from number: Int) {
    func printNumber(_ value: Int) {
        guard value > 0 else { return }
        print(value)
        printNumber(value - 1)
    }

    printNumber(number)
}
```

## When Not To Use Nested Functions

Avoid nested functions when the helper:

- Is useful elsewhere
- Needs independent tests
- Makes the outer function too long
- Captures too much state

In those cases, extract a private method or standalone function.

## Junior-Level Interview Answer

A nested function is a function written inside another function. It can be used only inside the outer function.

## Mid-Level Interview Answer

Nested functions are useful for local helper logic. They keep implementation details hidden and prevent helper functions from polluting the wider type or file.

## Senior-Level Interview Answer

Nested functions are a scoping tool. They are useful when helper behavior belongs exclusively to one algorithm and benefits from capturing local values. I avoid them when they reduce testability, hide too much complexity, or capture mutable state in a way that makes behavior harder to reason about.

## Quick Interview Notes

- Nested functions are declared inside functions.
- They are scoped to the outer function.
- They can capture outer values.
- They are useful for local helper logic.
- Extract them if reuse or testing becomes important.

## Practice Questions

1. What is a nested function?
2. Can a nested function access outer function values?
3. Why use nested functions?
4. When should you avoid nested functions?
5. How are nested functions related to closures?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Nested Functions is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Nested Functions in an app feature:

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

1. Write a minimal example that shows Nested Functions correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Nested Functions, but it shows the kind of production shape you should connect this topic to:

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

