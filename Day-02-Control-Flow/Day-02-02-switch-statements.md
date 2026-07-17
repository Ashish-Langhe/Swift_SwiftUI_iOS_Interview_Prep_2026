# Day 2: Switch Statements

## What You Will Learn

- How `switch` works in Swift
- Why Swift `switch` is more powerful than many other languages
- Exhaustiveness
- Matching values, ranges, tuples, enums, and optionals
- `where` clauses
- Junior to senior-level interview answers

## Core Idea

`switch` compares a value against multiple possible patterns.

```swift
let statusCode = 200

switch statusCode {
case 200:
    print("Success")
case 401:
    print("Unauthorized")
case 500:
    print("Server error")
default:
    print("Other response")
}
```

Swift `switch` must be exhaustive. That means every possible value must be handled.

## No Implicit Fallthrough

In Swift, a `case` does not automatically continue into the next case.

```swift
let grade = "A"

switch grade {
case "A":
    print("Excellent")
case "B":
    print("Good")
default:
    print("Keep trying")
}
```

Only one matching case runs.

If you really need fallthrough, Swift has a `fallthrough` keyword, but it is rarely used in app code.

```swift
switch grade {
case "A":
    print("Excellent")
    fallthrough
case "B":
    print("Passed")
default:
    print("Completed")
}
```

Use `fallthrough` carefully because it can make logic surprising.

## Multiple Values In One Case

```swift
let day = "Saturday"

switch day {
case "Saturday", "Sunday":
    print("Weekend")
default:
    print("Weekday")
}
```

## Range Matching

```swift
let score = 86

switch score {
case 90...100:
    print("Excellent")
case 75..<90:
    print("Good")
case 50..<75:
    print("Pass")
case 0..<50:
    print("Fail")
default:
    print("Invalid score")
}
```

Ranges make score, age, percentage, and status code logic cleaner.

## Tuple Matching

```swift
let point = (x: 0, y: 5)

switch point {
case (0, 0):
    print("Origin")
case (0, _):
    print("On Y-axis")
case (_, 0):
    print("On X-axis")
default:
    print("Somewhere else")
}
```

`_` means ignore this value.

## Value Binding

You can bind values inside a case.

```swift
let response = (statusCode: 404, message: "Not Found")

switch response {
case (200, let message):
    print("Success: \(message)")
case (let code, let message):
    print("Code \(code): \(message)")
}
```

## Where Clauses

Use `where` to add extra conditions to a pattern.

```swift
let point = (x: 3, y: 3)

switch point {
case let (x, y) where x == y:
    print("Diagonal")
case let (x, y) where x > y:
    print("X is greater")
case let (x, y) where y > x:
    print("Y is greater")
default:
    print("Other")
}
```

`where` is powerful, but avoid making cases too complex.

## Enum Switching

`switch` works extremely well with enums.

```swift
enum LoadingState {
    case idle
    case loading
    case loaded
    case failed(String)
}

let state = LoadingState.failed("No internet")

switch state {
case .idle:
    print("Waiting")
case .loading:
    print("Loading")
case .loaded:
    print("Loaded")
case .failed(let message):
    print("Error: \(message)")
}
```

Because all enum cases are handled, no `default` is needed.

## Should You Use Default With Enums?

For app-owned enums, avoid `default` when possible.

```swift
switch state {
case .idle:
    print("Idle")
case .loading:
    print("Loading")
case .loaded:
    print("Loaded")
case .failed:
    print("Failed")
}
```

Why?

If a new enum case is added later, the compiler will force you to update every switch.

Avoid:

```swift
switch state {
case .loaded:
    print("Loaded")
default:
    print("Other")
}
```

This can hide missing behavior when new states are introduced.

For external framework enums, sometimes `@unknown default` is appropriate.

```swift
import Foundation

let style: DateFormatter.Style = .short

switch style {
case .none:
    print("None")
case .short:
    print("Short")
case .medium:
    print("Medium")
case .long:
    print("Long")
case .full:
    print("Full")
@unknown default:
    print("Future unknown style")
}
```

Use `@unknown default` when switching over non-frozen enums from Apple frameworks.

## Optional Switching

```swift
let name: String? = "Naina"

switch name {
case .some(let value):
    print("Name: \(value)")
case .none:
    print("No name")
}
```

More commonly:

```swift
if let name {
    print(name)
}
```

But optional switching is useful when combined with patterns.

## Real iOS Use Cases

### View State

```swift
enum ProfileViewState {
    case loading
    case content(User)
    case empty
    case error(String)
}

switch state {
case .loading:
    showLoadingView()
case .content(let user):
    showProfile(user)
case .empty:
    showEmptyView()
case .error(let message):
    showError(message)
}
```

### HTTP Status Handling

```swift
switch statusCode {
case 200..<300:
    print("Success")
case 400..<500:
    print("Client error")
case 500..<600:
    print("Server error")
default:
    print("Unexpected response")
}
```

### Navigation Action

```swift
enum SettingsAction {
    case openProfile
    case openNotifications
    case logout
}

switch action {
case .openProfile:
    navigateToProfile()
case .openNotifications:
    navigateToNotifications()
case .logout:
    confirmLogout()
}
```

## Junior-Level Interview Answer

Question: What is a `switch` statement in Swift?

Answer:

`switch` checks one value against multiple cases and runs the matching case. Swift switch statements must handle all possible values, either by listing all cases or using `default`.

## Mid-Level Interview Answer

Question: Why is `switch` useful with enums?

Answer:

`switch` is useful with enums because the compiler can check exhaustiveness. If every enum case is handled, no default is needed. If a new case is added later, the compiler shows all places that need updates. This makes state-based code safer.

## Senior-Level Interview Answer

Question: When can `default` be harmful in Swift switches?

Answer:

`default` can be harmful when switching over app-owned enums because it hides future missing cases. If the enum represents domain state, I prefer explicit handling for every case. For non-frozen Apple framework enums, I use `@unknown default` so the app can handle future cases while still getting compiler warnings.

## Quick Interview Notes

- Swift `switch` must be exhaustive.
- Swift does not fall through by default.
- `switch` supports ranges, tuples, value binding, optionals, and `where`.
- Prefer `switch` for enums and state machines.
- Avoid `default` for app-owned enums when exhaustive cases are better.
- Use `@unknown default` for future framework enum cases.

## Points To Remember

- `switch` is a pattern matching tool, not only a replacement for `if else`.
- Exhaustiveness is one of Swift's biggest safety features.
- The order of cases matters when patterns overlap.
- Use `where` for additional constraints.
- Keep each case small and readable.

## Practice Questions

1. What does exhaustive mean in a Swift switch?
2. Does Swift switch fall through by default?
3. When should you avoid `default`?
4. What is `@unknown default`?
5. How can `switch` match ranges and tuples?

