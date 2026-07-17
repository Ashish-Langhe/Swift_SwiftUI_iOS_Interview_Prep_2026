# Day 1: Operators, Expressions, And Basic Output

## Core Idea

Operators are symbols or keywords that perform operations on values.

```swift
let total = 10 + 5
let isAdult = age >= 18
let canAccess = isLoggedIn && isPremium
```

An expression is a piece of code that produces a value.

```swift
10 + 5
age >= 18
"Hello, \(name)"
```

## Arithmetic Operators

```swift
let a = 10
let b = 3

let sum = a + b        // 13
let difference = a - b // 7
let product = a * b    // 30
let quotient = a / b   // 3
let remainder = a % b  // 1
```

Integer division removes the decimal part:

```swift
let result = 10 / 3 // 3
```

Use `Double` for decimal division:

```swift
let result = 10.0 / 3.0 // 3.333...
```

## Assignment Operators

```swift
var count = 0
count += 1
count -= 1
count *= 2
count /= 2
```

Swift does not support `count++` or `count--`.

Use:

```swift
count += 1
```

## Comparison Operators

```swift
let isEqual = a == b
let isNotEqual = a != b
let isGreater = a > b
let isLess = a < b
let isGreaterOrEqual = a >= b
let isLessOrEqual = a <= b
```

Comparison operators return `Bool`.

## Logical Operators

```swift
let isLoggedIn = true
let hasSubscription = false

let canWatchPremiumVideo = isLoggedIn && hasSubscription
let canSeePreview = isLoggedIn || hasSubscription
let shouldShowLogin = !isLoggedIn
```

Meanings:

- `&&`: AND
- `||`: OR
- `!`: NOT

## Nil-Coalescing Operator

The nil-coalescing operator provides a fallback for an optional.

```swift
let displayName: String? = nil
let visibleName = displayName ?? "Guest"
```

If `displayName` has a value, `visibleName` uses it. Otherwise, it uses `"Guest"`.

## Range Operators

Closed range:

```swift
for number in 1...5 {
    print(number)
}
```

Half-open range:

```swift
let names = ["A", "B", "C"]

for index in 0..<names.count {
    print(names[index])
}
```

Prefer direct collection iteration when possible:

```swift
for name in names {
    print(name)
}
```

## Ternary Conditional Operator

```swift
let age = 20
let message = age >= 18 ? "Adult" : "Minor"
```

Use it for simple choices. Avoid nesting ternary operators because readability drops quickly.

## String Interpolation

```swift
let name = "Sana"
let completed = 4

let message = "\(name) completed \(completed) lessons."
```

This is common in UI labels, logs, and debug messages.

## Basic Output

Use `print` for simple output:

```swift
print("Hello, Swift")
```

Print multiple values:

```swift
let user = "Dev"
let score = 90

print(user, score)
```

Use interpolation for clearer output:

```swift
print("User: \(user), Score: \(score)")
```

In real iOS apps, prefer logging tools for serious diagnostics, but `print` is fine for beginner examples and quick debugging.

## Comments

Single-line comment:

```swift
// This is a comment
```

Multiline comment:

```swift
/*
 This is a multiline comment.
 It can span multiple lines.
 */
```

Documentation comment:

```swift
/// Returns the final price after applying tax.
func finalPrice(amount: Double, taxRate: Double) -> Double {
    amount + (amount * taxRate)
}
```

Documentation comments are useful for public APIs, utilities, and reusable components.

## Real iOS Use Cases

### Badge Count

```swift
var unreadMessages = 9
unreadMessages += 1
```

### Access Check

```swift
let canOpenSettings = isLoggedIn && !isGuest
```

### Fallback Name

```swift
let title = user.profileName ?? user.email
```

### Price Label

```swift
let quantity = 2
let price = 499.0
let total = Double(quantity) * price

let label = "Total: INR \(total)"
```

## Swift 6.x Relevance

The operators themselves are beginner-level, but Swift 6.x makes expression clarity more important in concurrent code. For example, avoid hiding mutable shared state changes inside complex expressions.

Less clear:

```swift
var count = 0
let message = "Count: \(count += 1)"
```

This style is not valid Swift as written, and even if similar patterns are possible through function calls, they are hard to reason about.

Clearer:

```swift
count += 1
let message = "Count: \(count)"
```

Simple, readable steps make concurrency and debugging easier.

## Quick Interview Notes

- Arithmetic operators include `+`, `-`, `*`, `/`, `%`.
- `==` checks equality; `=` assigns a value.
- `&&`, `||`, and `!` are logical operators.
- `??` gives a default value for an optional.
- `...` creates a closed range; `..<` creates a half-open range.
- Swift removed `++` and `--`; use `+= 1` and `-= 1`.
- `print` is useful for learning but not a complete logging strategy.

## Points To Remember

- Operators should make code shorter, not less readable.
- Be careful with integer division.
- Prefer direct collection iteration over index-based loops when possible.
- Use documentation comments for reusable functions.
- Avoid overly clever expressions in production code.

## Practice Questions

1. What is the difference between `=` and `==`?
2. What does the `%` operator do?
3. What is the difference between `...` and `..<`?
4. Why does `10 / 3` return `3`?
5. What does the nil-coalescing operator do?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Operators, Expressions, And Basic Output is not only a syntax topic. In production Swift, it affects type safety, mutability, compiler feedback, and readable beginner code. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Operators, Expressions, And Basic Output in an app feature:

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

1. Write a minimal example that shows Operators, Expressions, And Basic Output correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Operators, Expressions, And Basic Output, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Operators, Expressions, And Basic Output** is evaluated through this lens: language fundamentals are really about making illegal states hard to write and easy compiler feedback possible. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Operators, Expressions, And Basic Output
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

For **Operators, Expressions, And Basic Output**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

