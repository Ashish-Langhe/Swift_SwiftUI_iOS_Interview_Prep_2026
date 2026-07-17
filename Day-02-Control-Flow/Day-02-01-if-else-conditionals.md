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

