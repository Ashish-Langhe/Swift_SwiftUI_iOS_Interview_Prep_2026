# Day 1: Swift Basics Interview Guide

## What You Will Learn

- How to answer Swift basics questions from junior to senior level
- How to explain `let`, `var`, type inference, type safety, literals, and operators
- How to connect beginner syntax to production iOS code
- How Swift 6.x changes affect basic Swift interview answers
- Common coding exercises and revision questions

## Day 1 Summary

Day 1 covers the foundation of Swift:

- Variables and constants
- Type inference and type annotations
- Basic data types
- Literals
- Operators
- String interpolation
- Comments
- Basic output
- Swift 6.1 to 6.3 beginner-aware changes

The goal is not only to know syntax. The real interview goal is to explain why Swift is safe, expressive, and strongly typed.

## Core Interview Theme

Swift basics are built around four ideas:

- Safety
- Clarity
- Type correctness
- Intentional mutability

When answering interviews, do not only say what the syntax does. Explain the reason behind the syntax.

Example:

```swift
let username = "Aarav"
var loginAttemptCount = 0
```

Junior answer:

`let` is constant and `var` is variable.

Senior answer:

I use `let` by default because it communicates immutability and prevents accidental reassignment. I use `var` only when the value genuinely changes over time, such as UI state, counters, form input, or cached mutable data.

## Question 1: What Is The Difference Between Let And Var?

### Junior-Level Answer

`let` creates a constant and `var` creates a variable. A constant cannot be changed after assigning a value. A variable can be changed.

```swift
let appName = "Interview Prep"
var score = 0

score += 10
```

### Mid-Level Answer

`let` should be preferred by default because it makes code safer and easier to understand. `var` should be used when a value needs to change, such as loading state, counters, user input, or SwiftUI state.

```swift
let baseURL = URL(string: "https://api.example.com")!
var isLoading = false
```

### Senior-Level Answer

The choice between `let` and `var` is about controlling mutability. Immutability reduces accidental state changes, improves reasoning, and helps when reviewing concurrent code. In Swift 6, where data-race safety is more important, unnecessary mutable shared state becomes a bigger design smell. For shared mutable state, I prefer isolation tools such as actors or main actor-bound view models.

```swift
actor DownloadCounter {
    private var count = 0

    func increment() {
        count += 1
    }

    func currentValue() -> Int {
        count
    }
}
```

## Question 2: What Happens If A Class Instance Is Declared With Let?

### Junior-Level Answer

If a class object is declared with `let`, the reference cannot point to a new object.

```swift
final class User {
    var name: String

    init(name: String) {
        self.name = name
    }
}

let user = User(name: "Meera")
user.name = "Meera Sharma"
```

This is allowed because the object property is mutable.

### Mid-Level Answer

`let` prevents reassignment of the reference, not necessarily mutation inside the referenced object. Classes are reference types. Structs are value types, so a `let` struct cannot have its mutable stored properties changed.

```swift
struct Profile {
    var name: String
}

let profile = Profile(name: "Kabir")
// profile.name = "Kabir Singh"
// Error
```

### Senior-Level Answer

This distinction is part of Swift's value semantics vs reference semantics story. With classes, `let` freezes the reference binding. With structs, `let` freezes the value. In production code, this affects API design, thread safety, copy behavior, and predictability. For models, I usually prefer immutable structs unless identity, inheritance, shared mutable state, or Objective-C interoperability makes a class appropriate.

## Question 3: What Is Type Inference?

### Junior-Level Answer

Type inference means Swift can understand the type from the assigned value.

```swift
let name = "Riya"   // String
let age = 25        // Int
let rating = 4.5    // Double
```

### Mid-Level Answer

Type inference reduces boilerplate but Swift remains strongly typed. Once Swift infers a type, that value cannot be used as a different unrelated type.

```swift
let age = 25
// age = "Twenty five"
// Error
```

### Senior-Level Answer

Type inference is a compiler feature that improves expressiveness while preserving static type safety. I use inference when the type is obvious and explicit annotations when they clarify intent, define an API boundary, select a specific numeric type, or make protocol-based abstraction visible.

```swift
let amount: Decimal = 199.99
let completion: (Result<Data, Error>) -> Void
```

## Question 4: Is Swift Dynamically Typed?

Answer:

No. Swift is statically typed and strongly typed.

Statically typed means types are checked at compile time.

Strongly typed means Swift does not freely mix incompatible types.

```swift
let count = 5
let text = "items"

// let result = count + text
// Error
```

Correct:

```swift
let result = "\(count) items"
```

Senior note:

Swift gives a clean syntax through type inference, but it is not dynamically typed like JavaScript or Python.

## Question 5: Why Does Swift Not Automatically Convert Int To Double?

### Junior-Level Answer

Swift requires explicit conversion between numeric types.

```swift
let quantity = 3
let price = 99.99

let total = Double(quantity) * price
```

### Senior-Level Answer

Swift avoids implicit numeric conversions to prevent hidden precision loss, overflow issues, and unclear behavior. Explicit conversion makes the programmer's intent visible. This matters in financial, analytics, animation, graphics, and API code where numeric meaning is important.

## Question 6: What Are Basic Swift Data Types?

Common types:

- `Int`
- `Double`
- `Float`
- `Bool`
- `String`
- `Character`
- `Optional`

Example:

```swift
let id: Int = 101
let name: String = "Anika"
let rating: Double = 4.8
let isPremium: Bool = true
let middleName: String? = nil
```

Senior explanation:

Choosing the right type is part of modeling. Do not use `String` for everything. Use booleans for true/false state, `Decimal` for money when precision matters, enums for finite states, and optionals when absence is meaningful.

## Question 7: What Is An Optional?

### Junior-Level Answer

An optional means a value may or may not exist.

```swift
var profileImageURL: URL?
```

It can contain a URL or `nil`.

### Mid-Level Answer

Optionals are Swift's safe way to represent missing values. Instead of using null unsafely, Swift forces us to unwrap optionals before using the wrapped value.

```swift
if let profileImageURL {
    print(profileImageURL)
}
```

### Senior-Level Answer

Optionals are a modeling tool. A property should be optional only when absence is valid in the domain. Overusing optionals creates unnecessary unwrapping and unclear states. Underusing optionals can force placeholder values such as empty strings, which often hide real absence.

## Question 8: What Is A Literal?

Answer:

A literal is a value written directly in code.

```swift
let count = 10
let message = "Hello"
let isEnabled = true
let numbers = [1, 2, 3]
```

Senior note:

Literals are convenient, but production code should avoid scattering magic values everywhere. Repeated values should often become constants, configuration, enums, or localized strings.

```swift
let maximumLoginAttempts = 3
```

## Question 9: What Is String Interpolation?

Answer:

String interpolation inserts values into a string using `\(value)`.

```swift
let name = "Dev"
let score = 95

let message = "\(name) scored \(score)"
```

Senior note:

String interpolation is useful for display and debugging, but user-facing strings should consider localization, formatting, currency, dates, and pluralization.

## Question 10: What Is The Difference Between = And ==?

Answer:

`=` assigns a value.

```swift
var count = 10
```

`==` compares two values.

```swift
if count == 10 {
    print("Ten")
}
```

Common interview trap:

Do not confuse assignment with equality checking.

## Question 11: What Are Logical Operators?

Answer:

Logical operators combine or modify boolean values.

```swift
let isLoggedIn = true
let hasSubscription = false

let canWatch = isLoggedIn && hasSubscription
let canPreview = isLoggedIn || hasSubscription
let shouldShowLogin = !isLoggedIn
```

Meanings:

- `&&`: AND
- `||`: OR
- `!`: NOT

Senior note:

When logical expressions become too long, extract them into named properties or methods.

```swift
if user.canAccessPremiumContent {
    print("Show premium content")
}
```

## Question 12: What Is The Nil-Coalescing Operator?

Answer:

`??` provides a default value when an optional is nil.

```swift
let displayName: String? = nil
let title = displayName ?? "Guest"
```

Senior note:

Use nil-coalescing when the fallback is truly valid. Do not hide missing critical data with a default that changes business meaning.

## Question 13: What Is The Difference Between Double, Float, And Decimal?

Answer:

`Double` is a 64-bit floating-point type and is the default for decimal literals.

`Float` is a 32-bit floating-point type with less precision.

`Decimal` is useful when decimal precision matters, especially for money-like values.

```swift
let rating: Double = 4.8
let progress: Float = 0.75
let price: Decimal = 1499.99
```

Senior note:

Use the type required by the API and domain. For UI animation, Apple APIs may use `Double`, `Float`, or `CGFloat`. For financial calculations, avoid casual use of binary floating-point types when exact decimal behavior matters.

## Question 14: What Swift 6.x Changes Should Beginners Know?

### Junior-Level Answer

Swift 6.x focuses a lot on safer concurrency and better tooling.

### Mid-Level Answer

Swift 6.1 improved areas like trailing comma support, task group inference, `nonisolated` on types and extensions, and package traits. Swift 6.2 made concurrency more approachable and added features like `@concurrent`, `InlineArray`, `Span`, and testing improvements. Swift 6.3 added official Swift SDK support for Android and improved Embedded Swift and DocC.

### Senior-Level Answer

The important direction is that Swift is becoming safer and more cross-platform. For normal iOS app work, the most relevant theme is concurrency safety: reduce unnecessary shared mutable state, understand actor isolation, learn `Sendable`, and model data as immutable value snapshots where possible. For interview accuracy, I would not claim Swift 6.4 features unless an official Swift.org release confirms them.

## Common Coding Exercise 1: Create A User Summary

Problem:

Create a function that accepts name, age, and premium status, then returns a readable summary.

```swift
func userSummary(name: String, age: Int, isPremium: Bool) -> String {
    let accountType = isPremium ? "Premium" : "Free"
    return "\(name), age \(age), account: \(accountType)"
}
```

Interview discussion:

- Uses explicit parameter types
- Uses a constant for derived value
- Uses ternary for simple conditional choice
- Uses string interpolation

## Common Coding Exercise 2: Calculate Total Price

```swift
func totalPrice(quantity: Int, unitPrice: Decimal) -> Decimal {
    Decimal(quantity) * unitPrice
}
```

Interview discussion:

- Shows explicit numeric conversion
- Uses `Decimal` for price
- Keeps values immutable

## Common Coding Exercise 3: Safely Build Display Name

```swift
func displayName(firstName: String?, lastName: String?) -> String {
    let first = firstName ?? ""
    let last = lastName ?? ""
    let fullName = "\(first) \(last)".trimmingCharacters(in: .whitespaces)

    return fullName.isEmpty ? "Guest" : fullName
}
```

Senior discussion:

This is acceptable for display fallback, but for identity-sensitive flows, returning `"Guest"` may hide missing data. The right behavior depends on the domain.

## Common Coding Exercise 4: Choose Let Or Var

Question:

Which declarations should be `let` and which should be `var`?

```swift
let apiEndpoint = "https://api.example.com/users"
var retryCount = 0
let maximumRetryCount = 3
var isLoading = false
```

Explanation:

- `apiEndpoint` is constant configuration.
- `retryCount` changes.
- `maximumRetryCount` is a fixed rule.
- `isLoading` changes based on request state.

## Common Mistakes In Interviews

### Saying Let Makes An Object Fully Immutable

Wrong:

`let` makes a class object fully immutable.

Correct:

`let` prevents reassignment of the reference. Class properties may still change if they are declared with `var`.

### Saying Swift Is Dynamically Typed

Wrong:

Swift is dynamically typed because it can infer types.

Correct:

Swift is statically typed. Type inference is compile-time convenience.

### Using String For Every State

Avoid:

```swift
let state = "loading"
```

Prefer:

```swift
enum State {
    case loading
    case loaded
    case failed
}
```

### Ignoring Optionals

Avoid force unwraps in interview examples unless you explain why they are safe.

```swift
let url = URL(string: "https://example.com")!
```

Better for general examples:

```swift
if let url = URL(string: "https://example.com") {
    print(url)
}
```

## Junior-Level Rapid Revision

- `let` means constant.
- `var` means variable.
- Swift is type-safe.
- Type inference lets Swift detect types automatically.
- `String` stores text.
- `Int` stores whole numbers.
- `Double` stores decimal numbers.
- `Bool` stores true or false.
- Optional means value or nil.
- `print()` displays output.

## Mid-Level Rapid Revision

- Prefer `let` by default.
- Use explicit type annotations when they improve clarity.
- Swift does not do implicit numeric conversion.
- Use `Decimal` carefully for money.
- Use optionals only when absence is meaningful.
- Use string interpolation for readable strings.
- Extract magic values into named constants.
- Avoid force unwraps in normal app code.

## Senior-Level Rapid Revision

- Mutability is a design decision.
- Type choices are domain modeling decisions.
- Immutability helps readability and concurrency safety.
- `let` behaves differently with value types and reference types.
- Type inference should not hide important API contracts.
- Defaults should not hide business-critical missing data.
- Swift 6.x interview answers should mention concurrency safety carefully.
- Do not invent unverified Swift version features.

## Final Interview Questions

1. Explain `let` vs `var` with value types and reference types.
2. Why does Swift prefer constants?
3. Is Swift statically typed or dynamically typed?
4. What is type inference?
5. When should you write explicit type annotations?
6. Why does Swift require explicit numeric conversion?
7. What is an optional?
8. When should a property be optional?
9. What is string interpolation?
10. What is the difference between `=` and `==`?
11. What does `??` do?
12. Why might `Decimal` be better than `Double` for money?
13. How does Swift 6.x make mutability more important?
14. What beginner Swift 6.1 to 6.3 changes can you mention in interviews?
15. Why should you avoid claiming Swift 6.4 features without official confirmation?

## One-Minute Perfect Answer

Swift basics are centered on safety and clarity. I use `let` for constants and `var` only when a value must change. Swift is statically and strongly typed, but type inference keeps syntax concise. I choose types based on domain meaning, use optionals to represent real absence, and avoid force unwraps unless there is a guaranteed invariant. Swift operators and string interpolation make code expressive, but I keep business logic readable by naming important constants and extracting complex conditions. In modern Swift 6.x, these basics matter even more because immutability, precise types, and clear state modeling help with concurrency safety and maintainable iOS code.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Swift Basics Interview Guide is not only a syntax topic. In production Swift, it affects type safety, mutability, compiler feedback, and readable beginner code. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Swift Basics Interview Guide in an app feature:

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

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.

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

1. Write a minimal example that shows Swift Basics Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Swift Basics Interview Guide, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Swift Basics Interview Guide** is evaluated through this lens: language fundamentals are really about making illegal states hard to write and easy compiler feedback possible. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Swift Basics Interview Guide
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

For **Swift Basics Interview Guide**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Swift Basics Interview Guide** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Editable Profile Draft

```swift
struct ProfileDraft {
    var firstName: String
    var lastName: String
    let userID: UUID

    var displayName: String {
        "\(firstName) \(lastName)".trimmingCharacters(in: .whitespaces)
    }
}

var draft = ProfileDraft(firstName: "Ashish", lastName: "Langhe", userID: UUID())
draft.firstName = "Aashish"
print(draft.displayName)
```

This is approachable because it uses basic variables and constants in a real app shape: editable user input plus a fixed identity.

### Example 2: Simple Validation Flag

```swift
let minimumPasswordLength = 8
var password = "swift2026"

let isPasswordValid = password.count >= minimumPasswordLength
print(isPasswordValid ? "Enable Continue" : "Disable Continue")
```

Small constants remove magic numbers and make UI decisions easier to read.

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

### Why This Fits Swift Basics Interview Guide

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

