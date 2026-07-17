# Day 2: Control Flow Interview Guide

## What You Will Learn

- How to answer control-flow questions at junior, mid, and senior levels
- How to compare `if`, `guard`, and `switch`
- How to discuss readability, safety, and maintainability
- Common coding exercises
- Production-level best practices

## Control Flow Summary

Control flow decides which code runs, how often it runs, and when execution exits.

Main tools:

- `if`, `else if`, `else`
- `switch`
- `for-in`
- `while`
- `repeat-while`
- `guard`
- `break`
- `continue`
- Pattern matching
- `where`

## If Vs Guard Vs Switch

| Tool | Best Use |
| --- | --- |
| `if` | Local branching where multiple paths are meaningful |
| `guard` | Required conditions and early exits |
| `switch` | Finite states, enums, patterns, ranges, and exhaustive handling |

## Junior-Level Comparison

`if` checks a condition. `guard` checks a condition that must be true to continue. `switch` checks a value against multiple cases.

```swift
if isLoggedIn {
    print("Home")
}
```

```swift
guard let user else {
    return
}
```

```swift
switch state {
case .loading:
    print("Loading")
case .loaded:
    print("Loaded")
}
```

## Mid-Level Comparison

`if` is best for simple decisions. `guard` is best for validation and optional unwrapping where failure should exit early. `switch` is best when the value has multiple known shapes, especially enums. `switch` helps because Swift checks exhaustiveness.

## Senior-Level Comparison

At senior level, the decision is about modeling and maintainability. If many `if else` checks are testing string states, that probably means the state should be modeled as an enum. If a function has nested validation, `guard` can flatten the happy path. If a domain can be represented as finite cases, `switch` makes future changes safer because the compiler points out missing handling.

## Example: Junior Version

```swift
let status = "success"

if status == "loading" {
    print("Loading")
} else if status == "success" {
    print("Success")
} else {
    print("Failed")
}
```

This works, but string states are fragile.

## Example: Senior Version

```swift
enum RequestState {
    case loading
    case success
    case failed(Error)
}

func render(_ state: RequestState) {
    switch state {
    case .loading:
        print("Loading")
    case .success:
        print("Success")
    case .failed(let error):
        print("Failed: \(error)")
    }
}
```

This is safer because invalid states are not representable as random strings.

## Common Interview Question 1

Question: What is the difference between `if let` and `guard let`?

Junior answer:

`if let` unwraps an optional inside the `if` block. `guard let` unwraps an optional and makes the unwrapped value available after the guard statement.

Senior answer:

I use `if let` when the optional branch itself is the main decision. I use `guard let` when the optional value is a precondition for the rest of the function. `guard` keeps the happy path flat and moves failure handling close to the validation.

## Common Interview Question 2

Question: Why is Swift `switch` considered powerful?

Junior answer:

Swift `switch` can match many cases and must be exhaustive.

Senior answer:

Swift `switch` is a pattern matching construct. It supports values, ranges, tuples, optionals, enum associated values, type casting, and `where` clauses. Its exhaustiveness checking makes state handling safer, especially for app-owned enums.

## Common Interview Question 3

Question: When should you avoid `default` in a switch?

Junior answer:

Avoid `default` when you can handle all enum cases directly.

Senior answer:

For app-owned enums, explicit cases are better because the compiler will catch missing logic when a new case is added. `default` can hide new cases and cause incorrect behavior. For framework enums that may add cases in the future, use `@unknown default`.

## Common Interview Question 4

Question: What is the difference between `break` and `continue`?

Answer:

`break` exits the loop completely. `continue` skips the current iteration and moves to the next iteration.

```swift
for number in 1...5 {
    if number == 3 {
        continue
    }

    print(number)
}
```

This skips `3`.

```swift
for number in 1...5 {
    if number == 3 {
        break
    }

    print(number)
}
```

This stops at `3`.

## Common Interview Question 5

Question: How do you avoid deep nesting?

Answer:

Use `guard` for required preconditions, combine simple boolean checks, extract complex logic into named functions, and model state using enums where appropriate.

```swift
func canSubmit(email: String?, password: String?) -> Bool {
    guard let email, !email.isEmpty else { return false }
    guard let password, password.count >= 8 else { return false }

    return true
}
```

## Production Best Practices

### Prefer Domain Models Over Strings

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

### Keep Branches Small

Avoid putting too much work inside one branch.

```swift
switch action {
case .save:
    save()
case .delete:
    delete()
case .share:
    share()
}
```

Small branches are easier to test and review.

### Extract Complex Conditions

Avoid:

```swift
if user.isActive && user.hasVerifiedEmail && !user.isBlocked && user.role == .admin {
    print("Allow")
}
```

Prefer:

```swift
if user.canAccessAdminPanel {
    print("Allow")
}
```

The computed property can be tested separately.

### Be Careful With Side Effects

Conditions should usually be easy to read and not hide important side effects.

Avoid:

```swift
if service.refreshTokenAndReturnSuccess() {
    print("Continue")
}
```

Prefer:

```swift
let didRefreshToken = service.refreshTokenAndReturnSuccess()

if didRefreshToken {
    print("Continue")
}
```

This makes the side effect visible.

## Coding Exercise 1: Grade Calculator

Problem:

Write a function that returns a grade for a score.

```swift
func grade(for score: Int) -> String {
    switch score {
    case 90...100:
        return "A"
    case 75..<90:
        return "B"
    case 50..<75:
        return "C"
    case 0..<50:
        return "F"
    default:
        return "Invalid"
    }
}
```

## Coding Exercise 2: Login Validation

Problem:

Validate email and password using `guard`.

```swift
enum ValidationError: Error {
    case missingEmail
    case weakPassword
}

func validateLogin(email: String?, password: String?) throws {
    guard let email, email.contains("@") else {
        throw ValidationError.missingEmail
    }

    guard let password, password.count >= 8 else {
        throw ValidationError.weakPassword
    }
}
```

## Coding Exercise 3: Find First Even Number

```swift
func firstEvenNumber(in numbers: [Int]) -> Int? {
    for number in numbers {
        if number.isMultiple(of: 2) {
            return number
        }
    }

    return nil
}
```

Senior alternative:

```swift
func firstEvenNumber(in numbers: [Int]) -> Int? {
    numbers.first { $0.isMultiple(of: 2) }
}
```

Both are valid. The second is concise and expressive.

## Coding Exercise 4: Route Handler

```swift
enum AppRoute {
    case home
    case profile(userId: String)
    case settings
    case transaction(id: String)
}

func handle(route: AppRoute) {
    switch route {
    case .home:
        print("Open home")
    case .profile(let userId):
        print("Open profile \(userId)")
    case .settings:
        print("Open settings")
    case .transaction(let id):
        print("Open transaction \(id)")
    }
}
```

## Quick Interview Notes

- `if` is for conditional branching.
- `guard` is for required conditions and early exits.
- `switch` is best for multiple known cases and pattern matching.
- `for-in` is best for iterating sequences.
- `while` is best when iteration count is unknown.
- `repeat-while` runs at least once.
- `break` exits.
- `continue` skips.
- `where` adds conditions to patterns and loops.
- Avoid stringly typed state.
- Prefer compiler-checked enum states.

## Points To Remember

- Good control flow makes invalid states obvious.
- Senior Swift code often removes nesting through modeling, not only syntax.
- Exhaustive switching is a major safety feature.
- Use `guard` to protect the happy path.
- Use pattern matching for enum-heavy app state.

## Final Revision Questions

1. Explain `if`, `guard`, and `switch` in one sentence each.
2. Why does `guard` require an exit?
3. Why is exhaustive switching useful?
4. When would you use `for case`?
5. What is the risk of using `default` with app-owned enums?
6. How would you refactor nested conditionals?
7. What is the difference between `break` and `continue`?
8. Why should dictionary iteration order not be trusted for business logic?

