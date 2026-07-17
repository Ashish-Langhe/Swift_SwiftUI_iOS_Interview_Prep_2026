# Day 3: Inout Parameters

## Core Idea

Function parameters are constants by default. You cannot modify them directly.

```swift
func increment(_ value: Int) {
    // value += 1
    // Error
}
```

Use `inout` when a function needs to modify the caller's variable.

```swift
func increment(_ value: inout Int) {
    value += 1
}

var count = 10
increment(&count)
print(count)
```

The `&` means the argument is passed for modification.

## How Inout Works Conceptually

Think of `inout` as:

1. Copy the value into the function.
2. Modify it inside the function.
3. Write the modified value back to the caller.

Swift enforces rules to keep this safe.

## Real iOS Use Cases

### Swapping Values

```swift
func swapValues<T>(_ first: inout T, _ second: inout T) {
    let temporary = first
    first = second
    second = temporary
}

var a = 10
var b = 20
swapValues(&a, &b)
```

Swift already has `swap(&a, &b)`, but this demonstrates the idea.

### Updating Draft State

```swift
struct ProfileDraft {
    var name: String
    var city: String
}

func normalize(_ draft: inout ProfileDraft) {
    draft.name = draft.name.trimmingCharacters(in: .whitespaces)
    draft.city = draft.city.trimmingCharacters(in: .whitespaces)
}

var draft = ProfileDraft(name: " Riya ", city: " Pune ")
normalize(&draft)
```

## Inout Vs Return New Value

Instead of:

```swift
func trim(_ text: inout String) {
    text = text.trimmingCharacters(in: .whitespaces)
}
```

Often prefer:

```swift
func trimmed(_ text: String) -> String {
    text.trimmingCharacters(in: .whitespaces)
}

let cleanText = trimmed(" Swift ")
```

Returning a new value is easier to reason about and test.

## Inout Restrictions

You can only pass variables, not constants or literals.

```swift
var value = 5
increment(&value)

let fixedValue = 5
// increment(&fixedValue)
// Error

// increment(&10)
// Error
```

## Inout And Side Effects

`inout` creates side effects because it modifies external state.

That is not always bad, but it should be intentional and obvious.

```swift
func applyDiscount(to price: inout Decimal) {
    price *= 0.9
}
```

The caller sees `&price`, which clearly signals mutation.

## Junior-Level Interview Answer

`inout` lets a function change the original variable passed by the caller. The caller uses `&` when passing the variable.

## Mid-Level Interview Answer

Function parameters are constants by default. `inout` allows controlled mutation of the caller's variable. It is useful for operations like swapping values or updating a mutable draft, but should not be overused.

## Senior-Level Interview Answer

`inout` is explicit shared mutation at the function boundary. I use it when mutation is the clearest API, such as low-level algorithms, performance-sensitive transformations, or update-style APIs. For most business logic, I prefer returning a new value because it is easier to test, compose, reason about, and use safely in concurrent contexts.

## Quick Interview Notes

- Parameters are constants by default.
- `inout` allows modifying caller variables.
- Callers pass arguments with `&`.
- You cannot pass constants or literals to `inout`.
- Prefer return values unless mutation is the clearest design.

## Practice Questions

1. Why do we need `inout`?
2. What does `&` mean at the call site?
3. Can you pass a `let` constant to an `inout` parameter?
4. When is returning a new value better than using `inout`?
5. Why should `inout` be used carefully?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Inout Parameters is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Inout Parameters in an app feature:

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

1. Write a minimal example that shows Inout Parameters correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Inout Parameters, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Inout Parameters** is evaluated through this lens: functions are API design units; labels, return types, async/throws, and side effects define how other engineers use the code. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Inout Parameters
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

For **Inout Parameters**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

