# Day 1: Basic Data Types And Literals

## Core Data Types

Swift's most common beginner-level types are:

- `Int`
- `Double`
- `Float`
- `Bool`
- `String`
- `Character`
- `Optional`

## Int

`Int` stores whole numbers.

```swift
let age = 26
let unreadCount = 12
let temperature = -3
```

Use `Int` for counts, indexes, attempts, quantities, and whole-number values.

```swift
var cartItemCount = 0
cartItemCount += 1
```

## Double And Float

`Double` stores decimal numbers with higher precision than `Float`.

```swift
let rating = 4.7
let distanceInKm: Double = 12.5
let animationProgress: Float = 0.75
```

In iOS app code, `Double` is usually the default choice unless an Apple API requires `CGFloat` or `Float`.

For money, prefer `Decimal`:

```swift
let amount: Decimal = 1499.99
```

## Bool

`Bool` stores `true` or `false`.

```swift
let isLoggedIn = true
var isLoading = false
var hasInternetConnection = true
```

Use boolean names that read like yes/no questions.

Good:

```swift
let isPremiumUser = true
let hasCompletedOnboarding = false
let canRetryPayment = true
```

Avoid:

```swift
let premium = true
let onboarding = false
```

## String

`String` stores text.

```swift
let name = "Ananya"
let message = "Welcome, \(name)"
```

String interpolation inserts values into text:

```swift
let score = 95
let result = "Your score is \(score)"
```

Multiline strings:

```swift
let emailBody = """
Hi,

Your interview preparation plan is ready.

Thanks
"""
```

## Character

`Character` stores a single user-perceived character.

```swift
let grade: Character = "A"
let symbol: Character = "?"
```

Swift strings are Unicode-aware, so a character may use more than one underlying Unicode scalar.

## Optional

An optional means a value may be present or absent.

```swift
var selectedUserName: String?
selectedUserName = "Isha"
selectedUserName = nil
```

Use optionals when absence is meaningful:

```swift
struct User {
    let id: Int
    let name: String
    let profileImageURL: URL?
}
```

`profileImageURL` is optional because not every user may have a profile image.

## Literals

A literal is a direct value written in source code.

```swift
let integerLiteral = 100
let floatingLiteral = 99.5
let stringLiteral = "Swift"
let booleanLiteral = true
let arrayLiteral = [1, 2, 3]
let dictionaryLiteral = ["name": "Riya", "city": "Mumbai"]
```

## Numeric Separators

Use underscores to make large numbers readable:

```swift
let population = 1_400_000_000
let fileSize = 10_485_760
```

The underscores do not affect the value.

## Basic Collections Preview

Arrays:

```swift
let topics = ["Variables", "Types", "Operators"]
```

Dictionaries:

```swift
let user = [
    "name": "Rahul",
    "role": "iOS Developer"
]
```

Sets:

```swift
let uniqueSkills: Set<String> = ["Swift", "SwiftUI", "Xcode"]
```

Collections are covered deeply later, but beginners should recognize them early.

## Real iOS Use Cases

### API Model

```swift
struct Product: Decodable {
    let id: Int
    let title: String
    let price: Decimal
    let rating: Double
    let isAvailable: Bool
}
```

### Feature Flag

```swift
let isNewCheckoutEnabled = true
```

### Empty State Message

```swift
let emptyMessage = "No transactions found"
```

### Optional Image URL

```swift
let avatarURL: URL? = URL(string: "https://example.com/avatar.png")
```

## Swift 6.x Relevance

Swift 6.x continues to rely on strong, explicit data modeling. With stricter concurrency checks, simple value types such as `Int`, `String`, `Bool`, and structs containing them are easier to pass across tasks safely when they conform to `Sendable`.

```swift
struct TransactionSummary: Sendable {
    let totalCount: Int
    let totalAmount: Decimal
    let currencyCode: String
}
```

The model is immutable and safe to share as a snapshot.

## Quick Interview Notes

- `Int` is for whole numbers.
- `Double` is the default floating-point type.
- `Float` uses less precision than `Double`.
- `Bool` stores only `true` or `false`.
- `String` stores text.
- `Character` stores a single user-perceived character.
- Optional means a value may be absent.
- Use `Decimal` carefully for money-related values.

## Points To Remember

- Swift is type-safe.
- Choose types that describe the real domain.
- Optional is not an error; it is a model of absence.
- Avoid using `String` for everything.
- Immutable value models are easier to reason about and safer in concurrent code.

## Practice Questions

1. What is the difference between `Double` and `Float`?
2. Why is `Decimal` useful for money?
3. What is an optional?
4. Why should boolean names start with words like `is`, `has`, or `can`?
5. What is a literal?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Basic Data Types And Literals is not only a syntax topic. In production Swift, it affects type safety, mutability, compiler feedback, and readable beginner code. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Basic Data Types And Literals in an app feature:

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

1. Write a minimal example that shows Basic Data Types And Literals correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Basic Data Types And Literals, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Basic Data Types And Literals** is evaluated through this lens: language fundamentals are really about making illegal states hard to write and easy compiler feedback possible. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Basic Data Types And Literals
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

For **Basic Data Types And Literals**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

