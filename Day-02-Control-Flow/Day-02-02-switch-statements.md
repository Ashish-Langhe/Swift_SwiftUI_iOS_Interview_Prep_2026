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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Switch Statements is not only a syntax topic. In production Swift, it affects branch clarity, exhaustiveness, early exits, and reducing nested logic. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Switch Statements in an app feature:

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

1. Write a minimal example that shows Switch Statements correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Switch Statements, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Switch Statements** is evaluated through this lens: control flow is domain modeling in motion: every branch should represent a real state or decision. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a state-transition table or switch map showing each user/API state and the expected UI behavior
Topic: Switch Statements
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine reviewing checkout, onboarding, login, or permission flows where missing one branch creates a broken user journey. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is deep nesting, duplicated conditions, and default branches that hide unhandled states.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Is the branching exhaustive?
- Can guard clauses remove nesting?
- Does each branch represent a real product state?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Switch Statements**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

