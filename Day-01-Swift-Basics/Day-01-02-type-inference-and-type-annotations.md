# Day 1: Type Inference And Type Annotations

## Core Idea

Swift is a strongly typed language. Every value has a type. You can either let the compiler infer the type or write the type explicitly.

```swift
let city = "Pune"      // Inferred as String
let year = 2026        // Inferred as Int
let progress = 0.75    // Inferred as Double
let isActive = true    // Inferred as Bool
```

Type annotation means writing the type yourself:

```swift
let city: String = "Pune"
let year: Int = 2026
let progress: Double = 0.75
let isActive: Bool = true
```

## Why Type Inference Matters

Type inference keeps Swift concise without making it dynamically typed.

```swift
let userName = "Meera"
```

The compiler knows `userName` is a `String`. You do not need to write:

```swift
let userName: String = "Meera"
```

Both are valid. The first is more common when the type is obvious.

## When To Use Type Annotation

### 1. Declaring Without Initial Value

```swift
var selectedEmail: String
selectedEmail = "user@example.com"
```

The compiler needs the type because there is no initial value.

### 2. Choosing A Specific Numeric Type

```swift
let count: Int = 10
let price: Double = 99.99
let preciseAmount: Decimal = 199.99
```

For financial values, `Decimal` is often safer than `Double`.

### 3. Improving Readability

```swift
let completionHandler: (Result<String, Error>) -> Void
```

The explicit type helps the reader understand what kind of closure is expected.

### 4. Working With Protocols

```swift
protocol PaymentService {
    func pay(amount: Decimal)
}

let service: PaymentService = StripePaymentService()
```

The annotation hides the concrete type and exposes only the protocol contract.

## Type Safety

Swift does not silently mix unrelated types.

```swift
let age = 30
let name = "Ravi"

// let result = age + name
// Error: Binary operator '+' cannot be applied to operands of type 'Int' and 'String'
```

You must convert intentionally:

```swift
let result = "\(name) is \(age) years old"
```

## Numeric Inference

Integer literals are inferred as `Int` by default:

```swift
let quantity = 5 // Int
```

Floating-point literals are inferred as `Double` by default:

```swift
let discount = 0.10 // Double
```

You can ask for `Float`:

```swift
let animationDuration: Float = 0.35
```

In most app code, `Double` is preferred unless an API specifically requires `Float`.

## Type Conversion

Swift does not automatically convert between numeric types.

```swift
let quantity = 3
let price = 49.99

// let total = quantity * price
// Error: Cannot convert value of type 'Int' to expected argument type 'Double'
```

Correct:

```swift
let total = Double(quantity) * price
```

This explicit conversion avoids hidden precision or overflow bugs.

## Type Aliases

You can create a name for an existing type:

```swift
typealias UserId = String
typealias Completion = (Result<Data, Error>) -> Void

let id: UserId = "user-101"
```

Type aliases improve readability but do not create a new distinct type.

```swift
let userId: UserId = "u-1"
let plainString: String = userId // Allowed
```

## Real iOS Use Cases

### URL Construction

```swift
let baseURL: URL = URL(string: "https://api.example.com")!
```

Explicit `URL` makes the intended type clear.

### View Model State

```swift
final class LoginViewModel {
    var email: String = ""
    var password: String = ""
    var errorMessage: String?
}
```

`String?` clearly communicates that an error may or may not exist.

### API Response

```swift
struct UserResponse: Decodable {
    let id: Int
    let name: String
    let isVerified: Bool
}
```

The model's type annotations document the API contract.

## Swift 6.x Relevance

Modern Swift uses types not just for values, but also for safety across concurrency boundaries.

For example, `Sendable` tells Swift that a value can safely cross concurrency domains:

```swift
struct UserSnapshot: Sendable {
    let id: String
    let name: String
}
```

This matters in Swift 6 because strict concurrency checking catches more unsafe sharing.

## Quick Interview Notes

- Swift is statically and strongly typed.
- Type inference means the compiler determines the type from context.
- Type annotation means the developer writes the type explicitly.
- Integer literals default to `Int`.
- Floating-point literals default to `Double`.
- Swift does not perform implicit numeric conversions.
- Use explicit types when they improve clarity, when no initial value exists, or when a specific API contract matters.

## Points To Remember

- Type inference reduces noise, not type safety.
- Explicit conversions are intentional in Swift.
- Protocol-typed variables can hide implementation details.
- Type aliases improve readability but do not create new types.
- Swift 6 concurrency features rely heavily on precise type information.

## Practice Questions

1. What is type inference?
2. Why does Swift avoid implicit conversion from `Int` to `Double`?
3. When should you prefer explicit type annotations?
4. What is the default type of `10` and `10.0`?
5. Does `typealias UserId = String` create a new type?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Type Inference And Type Annotations is not only a syntax topic. In production Swift, it affects type safety, mutability, compiler feedback, and readable beginner code. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while building a small profile form, parsing values, and keeping invalid states out of your model. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Type Inference And Type Annotations in an app feature:

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

1. Write a minimal example that shows Type Inference And Type Annotations correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Type Inference And Type Annotations, but it shows the kind of production shape you should connect this topic to:

```swift
struct SignupDraft {
    var email: String
    var password: String
    let createdAt: Date

    var isReadyToSubmit: Bool {
        !email.isEmpty && password.count >= 8
    }
}

var draft = SignupDraft(email: "", password: "", createdAt: Date())
draft.email = "user@example.com"
draft.password = "12345678"

if draft.isReadyToSubmit {
    print("Submit enabled")
}
```

This example connects basics to real form state. `var` marks values the user edits, `let` protects creation metadata, and the computed property gives the UI a readable decision point.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Type Inference And Type Annotations** is evaluated through this lens: language fundamentals are really about making illegal states hard to write and easy compiler feedback possible. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a small value model for a screen, with intentional `let`/`var`, readable names, and no hidden mutable global state
Topic: Type Inference And Type Annotations
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine reviewing a signup form and deciding which values are user-editable, derived, or fixed after creation. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is treating beginner syntax as harmless and allowing loose mutation or vague names that later spread through the app.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Is each mutable value truly mutable?
- Would a clearer type or name remove a comment?
- Can the compiler catch the mistake before runtime?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Type Inference And Type Annotations**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

