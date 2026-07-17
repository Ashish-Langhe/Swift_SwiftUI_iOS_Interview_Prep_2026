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

