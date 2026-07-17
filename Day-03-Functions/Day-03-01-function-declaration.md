# Day 3: Function Declaration

## What You Will Learn

- What a function is
- How to declare and call functions
- How function names communicate intent
- How functions improve reuse, readability, and testability
- Junior to senior-level interview explanations

## Core Idea

A function is a named block of code that performs a task.

```swift
func greet() {
    print("Hello, Swift")
}

greet()
```

Function syntax:

```swift
func functionName() {
    // Code
}
```

## Why Functions Matter

Functions help you:

- Avoid duplicate code
- Give a name to a behavior
- Split large logic into smaller pieces
- Test behavior independently
- Improve readability

Without a function:

```swift
let price = 100.0
let tax = price * 0.18
let finalPrice = price + tax
```

With a function:

```swift
func finalPrice(for price: Double) -> Double {
    let tax = price * 0.18
    return price + tax
}
```

## Calling A Function

```swift
func showWelcomeMessage() {
    print("Welcome back")
}

showWelcomeMessage()
```

The code inside the function runs only when the function is called.

## Function Naming

Swift function names should read naturally at the call site.

Good:

```swift
func loadUserProfile() { }
func validateEmail() { }
func calculateTotalPrice() { }
```

Weak:

```swift
func doThing() { }
func process() { }
func data() { }
```

Clear names are especially important in interviews and production code.

## Functions Inside Types

Functions inside structs, classes, actors, and enums are usually called methods.

```swift
struct Cart {
    var items: [String]

    func itemCount() -> Int {
        items.count
    }
}

let cart = Cart(items: ["Book", "Pen"])
print(cart.itemCount())
```

## Void Functions

A function that does not return a value returns `Void`.

```swift
func logOut() {
    print("User logged out")
}
```

This is equivalent to:

```swift
func logOut() -> Void {
    print("User logged out")
}
```

Most developers omit `-> Void`.

## Single-Expression Functions

When a function has one expression, Swift can infer the return.

```swift
func square(_ number: Int) -> Int {
    number * number
}
```

This is valid because `number * number` is the only expression.

For beginners, writing `return` is sometimes clearer:

```swift
func square(_ number: Int) -> Int {
    return number * number
}
```

## Real iOS Use Cases

### ViewModel Action

```swift
final class LoginViewModel {
    func loginButtonTapped() {
        print("Validate input and start login")
    }
}
```

### Formatting Display Text

```swift
func displayName(firstName: String, lastName: String) -> String {
    "\(firstName) \(lastName)"
}
```

### Analytics Event

```swift
func trackScreenView(name: String) {
    print("Screen viewed: \(name)")
}
```

## Junior-Level Interview Answer

Question: What is a function in Swift?

Answer:

A function is a reusable block of code. We declare it using the `func` keyword and call it by using its name.

## Mid-Level Interview Answer

Question: Why do we use functions?

Answer:

Functions reduce duplication, make code more readable, and separate responsibilities. They also make logic easier to test because a function can take input and return output.

## Senior-Level Interview Answer

Question: What makes a function well-designed?

Answer:

A well-designed function has a clear purpose, a meaningful name, minimal side effects, focused inputs, and predictable output. In production Swift, I try to keep functions small enough to reason about, but not so fragmented that the flow becomes hard to follow. I also separate pure calculation functions from functions that perform side effects like network calls, navigation, persistence, or analytics.

## Quick Interview Notes

- Functions are declared with `func`.
- A function runs only when called.
- Functions improve reuse and testability.
- Functions inside types are called methods.
- `Void` means no return value.
- Single-expression functions can omit `return`.

## Points To Remember

- Name functions by what they do.
- Keep functions focused.
- Avoid hidden side effects in utility functions.
- Prefer functions that are easy to test.
- In iOS code, functions often represent user actions, formatting, validation, networking, and state updates.

## Practice Questions

1. What keyword declares a function?
2. What is the difference between declaring and calling a function?
3. What does `Void` mean?
4. What is a method?
5. What makes a function easy to test?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Function Declaration is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Function Declaration in an app feature:

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

1. Write a minimal example that shows Function Declaration correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Function Declaration, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Function Declaration** is evaluated through this lens: functions are API design units; labels, return types, async/throws, and side effects define how other engineers use the code. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Function Declaration
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

For **Function Declaration**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Function Declaration** to code you might write in a SwiftUI/UIKit feature.

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

### Why This Fits Function Declaration

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

