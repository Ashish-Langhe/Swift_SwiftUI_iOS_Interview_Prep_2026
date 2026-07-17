# Day 2: If, Else If, And Else Conditionals

## What You Will Learn

- How `if`, `else if`, and `else` work in Swift
- How to write readable conditions
- How conditionals behave with optionals, booleans, and comparisons
- Junior, mid-level, and senior-level interview explanations
- Real iOS use cases
- Common mistakes and best practices

## Core Idea

Conditional statements let your program choose different paths based on a condition.

```swift
let age = 20

if age >= 18 {
    print("Adult")
} else {
    print("Minor")
}
```

The condition must be a `Bool`.

```swift
let isLoggedIn = true

if isLoggedIn {
    print("Show dashboard")
}
```

Swift does not allow non-boolean values as conditions.

```swift
let count = 1

// if count {
//     print("Invalid in Swift")
// }
```

Correct:

```swift
if count > 0 {
    print("Valid")
}
```

## Basic Syntax

```swift
if condition {
    // Runs when condition is true
} else {
    // Runs when condition is false
}
```

Example:

```swift
let score = 82

if score >= 90 {
    print("Excellent")
} else if score >= 75 {
    print("Good")
} else if score >= 50 {
    print("Pass")
} else {
    print("Needs improvement")
}
```

Swift checks conditions from top to bottom. Once a matching branch is found, the remaining branches are skipped.

## Boolean Conditions

```swift
let isPremiumUser = true
let hasActiveSubscription = false

if isPremiumUser && hasActiveSubscription {
    print("Show premium content")
} else {
    print("Show upgrade screen")
}
```

Operators:

- `&&`: both conditions must be true
- `||`: at least one condition must be true
- `!`: reverses a boolean value

```swift
if !hasActiveSubscription {
    print("Subscription required")
}
```

## Optional Binding With `if let`

Use `if let` when a value may be nil.

```swift
var username: String? = "Aarav"

if let username {
    print("Welcome, \(username)")
} else {
    print("Welcome, Guest")
}
```

Older style, still valid:

```swift
if let username = username {
    print("Welcome, \(username)")
}
```

Modern shorthand is common when the unwrapped name should be the same as the optional name.

## Multiple Optional Bindings

```swift
let email: String? = "user@example.com"
let password: String? = "secret"

if let email, let password {
    print("Ready to login with \(email) and password length \(password.count)")
}
```

You can also add boolean checks:

```swift
if let email, email.contains("@") {
    print("Email looks valid")
}
```

## Conditions With Commas

In Swift, commas inside `if` conditions mean each condition must succeed.

```swift
let token: String? = "abc"
let isNetworkAvailable = true

if let token, isNetworkAvailable {
    print("Start request with token \(token)")
}
```

This is similar to logical AND, but it also supports optional binding and pattern matching.

## Nested If Statements

Nested conditions are sometimes needed, but too much nesting makes code harder to read.

```swift
if isLoggedIn {
    if hasActiveSubscription {
        if isNetworkAvailable {
            print("Play video")
        }
    }
}
```

Better:

```swift
if isLoggedIn && hasActiveSubscription && isNetworkAvailable {
    print("Play video")
}
```

Or for validation-heavy functions, use `guard`, which is covered later.

## Real iOS Use Cases

### Login Button State

```swift
let email = "user@example.com"
let password = "password123"

if email.isEmpty || password.isEmpty {
    print("Disable login button")
} else {
    print("Enable login button")
}
```

### Network Error Handling

```swift
let statusCode = 401

if statusCode == 200 {
    print("Success")
} else if statusCode == 401 {
    print("Unauthorized")
} else if statusCode >= 500 {
    print("Server error")
} else {
    print("Unknown response")
}
```

For many fixed cases, `switch` may be clearer.

### SwiftUI Conditional View

```swift
import SwiftUI

struct ProfileHeader: View {
    let name: String?

    var body: some View {
        if let name {
            Text("Welcome, \(name)")
        } else {
            Text("Welcome, Guest")
        }
    }
}
```

SwiftUI view builders support conditional rendering.

## Junior-Level Interview Answer

Question: What is `if else` in Swift?

Answer:

`if else` is used to execute code based on a condition. The condition must return `Bool`. If the condition is true, the `if` block runs. Otherwise, the `else` block runs.

```swift
if age >= 18 {
    print("Adult")
} else {
    print("Minor")
}
```

## Mid-Level Interview Answer

Question: How do you write clean conditional logic in Swift?

Answer:

I keep conditions explicit and readable, avoid unnecessary nesting, use `if let` for optional binding, and use `guard` for early exits when validation fails. If I have many fixed cases or enum states, I prefer `switch` because it is exhaustive and easier to maintain.

## Senior-Level Interview Answer

Question: How do you decide between `if`, `guard`, and `switch`?

Answer:

I use `if` for local branching where both paths are meaningful in the same scope. I use `guard` when a condition must be true for the rest of the function to continue, especially for validation and early exits. I use `switch` for finite states, enums, ranges, and pattern matching because it gives better exhaustiveness and makes impossible states more visible during code review.

Senior engineers usually think about control flow as a readability and correctness tool, not only syntax.

## Common Mistakes

### Comparing Boolean To True

Avoid:

```swift
if isLoggedIn == true {
    print("Logged in")
}
```

Prefer:

```swift
if isLoggedIn {
    print("Logged in")
}
```

### Using Too Many Nested Conditions

Avoid deeply nested logic where possible.

```swift
if let user {
    if user.isActive {
        if user.hasPermission {
            print("Allowed")
        }
    }
}
```

Better:

```swift
if let user, user.isActive, user.hasPermission {
    print("Allowed")
}
```

### Using `if` For Too Many Fixed Cases

```swift
if status == "loading" {
    print("Loading")
} else if status == "success" {
    print("Success")
} else if status == "failure" {
    print("Failure")
}
```

Better with enum and `switch`:

```swift
enum ViewState {
    case loading
    case success
    case failure
}

let state = ViewState.loading

switch state {
case .loading:
    print("Loading")
case .success:
    print("Success")
case .failure:
    print("Failure")
}
```

## Quick Interview Notes

- `if` conditions must be `Bool`.
- Swift does not treat `0`, `1`, empty strings, or nil as booleans.
- Use `if let` to unwrap optionals safely.
- Use commas in conditions to combine optional binding and boolean checks.
- Avoid deep nesting when a flat condition or `guard` is clearer.
- Prefer `switch` for enums and fixed state machines.

## Points To Remember

- Readability matters more than clever one-line conditions.
- Boolean variable names should read naturally, such as `isLoading`, `hasAccess`, and `canRetry`.
- Optional binding creates a non-optional value inside the block.
- `if` is best for small local decisions.
- For validation-heavy functions, `guard` often communicates intent better.

## Practice Questions

1. Why must an `if` condition be a `Bool` in Swift?
2. What is optional binding?
3. What is the difference between `if let name = name` and `if let name`?
4. How can you reduce nested `if` statements?
5. When would you prefer `switch` instead of `if else`?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

If, Else If, And Else Conditionals is not only a syntax topic. In production Swift, it affects branch clarity, exhaustiveness, early exits, and reducing nested logic. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while validating form input, handling API states, and routing user actions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying If, Else If, And Else Conditionals in an app feature:

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

1. Write a minimal example that shows If, Else If, And Else Conditionals correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use If, Else If, And Else Conditionals, but it shows the kind of production shape you should connect this topic to:

```swift
enum CheckoutState {
    case emptyCart
    case needsAddress
    case readyToPay(total: Decimal)
    case processing
    case failed(String)
}

func primaryActionTitle(for state: CheckoutState) -> String {
    switch state {
    case .emptyCart:
        return "Continue Shopping"
    case .needsAddress:
        return "Add Address"
    case .readyToPay:
        return "Pay Now"
    case .processing:
        return "Processing"
    case .failed:
        return "Try Again"
    }
}
```

This is the production mindset for control flow: branch on domain state, keep each case intentional, and let the compiler force you to handle new states.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

