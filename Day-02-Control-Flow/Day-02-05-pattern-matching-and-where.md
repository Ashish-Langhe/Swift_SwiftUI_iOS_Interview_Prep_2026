# Day 2: Pattern Matching And Where Clauses

## What You Will Learn

- What pattern matching means in Swift
- How patterns appear in `switch`, `if case`, `guard case`, and `for case`
- How `where` adds conditions
- How pattern matching improves enum-heavy iOS code
- Junior to senior-level interview explanations

## Core Idea

Pattern matching checks whether a value fits a specific shape.

You already use pattern matching when switching over enums:

```swift
enum ResultState {
    case success(String)
    case failure(Error)
}

let state = ResultState.success("Loaded")

switch state {
case .success(let message):
    print(message)
case .failure(let error):
    print(error)
}
```

The case `.success(let message)` matches only success values and extracts the associated message.

## If Case

Use `if case` when you only care about one pattern.

```swift
enum ViewState {
    case loading
    case content(String)
    case error(String)
}

let state = ViewState.content("Profile loaded")

if case .content(let message) = state {
    print(message)
}
```

This is cleaner than a full `switch` when only one case matters.

## Guard Case

Use `guard case` when a specific pattern is required to continue.

```swift
func render(state: ViewState) {
    guard case .content(let message) = state else {
        print("Cannot render content")
        return
    }

    print("Render: \(message)")
}
```

This is similar to `guard let`, but for enum cases and patterns.

## For Case

Use `for case` to iterate only matching values.

```swift
let states: [ViewState] = [
    .loading,
    .content("Home"),
    .error("No internet"),
    .content("Profile")
]

for case .content(let title) in states {
    print(title)
}
```

This prints only content values.

## Optional Pattern Matching

Optionals are enums internally:

```swift
enum Optional<Wrapped> {
    case some(Wrapped)
    case none
}
```

You can pattern match them:

```swift
let name: String? = "Riya"

if case .some(let value) = name {
    print(value)
}
```

Most of the time, use `if let`:

```swift
if let name {
    print(name)
}
```

But optional pattern matching becomes useful in advanced cases.

## Where Clauses In Switch

```swift
let transaction = (amount: 2500, category: "Travel")

switch transaction {
case let (amount, category) where amount > 2000 && category == "Travel":
    print("Large travel expense")
case let (amount, _) where amount > 2000:
    print("Large expense")
default:
    print("Normal expense")
}
```

`where` lets you add boolean rules to a pattern.

## Where In Loops

```swift
let numbers = [1, 2, 3, 4, 5, 6]

for number in numbers where number.isMultiple(of: 2) {
    print(number)
}
```

Use this for simple filtering. For complex filtering, consider a named function or `filter`.

## Tuple Pattern Matching

```swift
let locationPermission = (isGranted: true, isPrecise: false)

switch locationPermission {
case (true, true):
    print("Precise location available")
case (true, false):
    print("Approximate location available")
case (false, _):
    print("Location denied")
}
```

Tuple matching is useful for small combinations of values. If the combination becomes domain-heavy, prefer an enum or struct.

## Pattern Matching With Ranges

```swift
let batteryLevel = 18

switch batteryLevel {
case 0...20:
    print("Low battery")
case 21...80:
    print("Normal battery")
case 81...100:
    print("High battery")
default:
    print("Invalid battery level")
}
```

## Pattern Matching With Type Casting

```swift
let value: Any = "Swift"

switch value {
case let text as String:
    print("String: \(text)")
case let number as Int:
    print("Int: \(number)")
default:
    print("Unknown type")
}
```

Use type-casting switches sparingly. Strong modeling is usually better than passing around `Any`.

## Real iOS Use Cases

### Handling Screen State

```swift
enum ScreenState {
    case idle
    case loading
    case loaded(items: [String])
    case failed(message: String)
}

let screenState = ScreenState.loaded(items: ["A", "B"])

if case .loaded(let items) = screenState, !items.isEmpty {
    print("Show \(items.count) items")
}
```

### Filtering Analytics Events

```swift
enum AnalyticsEvent {
    case screenView(name: String)
    case buttonTap(id: String)
    case purchase(amount: Decimal)
}

let events: [AnalyticsEvent] = [
    .screenView(name: "Home"),
    .buttonTap(id: "pay_now"),
    .purchase(amount: 499)
]

for case .buttonTap(let id) in events {
    print("Tapped: \(id)")
}
```

### Navigation Routes

```swift
enum Route {
    case profile(userId: String)
    case settings
    case transaction(id: String)
}

func handle(route: Route) {
    switch route {
    case .profile(let userId):
        print("Open profile \(userId)")
    case .settings:
        print("Open settings")
    case .transaction(let id):
        print("Open transaction \(id)")
    }
}
```

## Junior-Level Interview Answer

Question: What is pattern matching?

Answer:

Pattern matching checks whether a value matches a specific pattern. In Swift, it is commonly used in `switch` statements, especially with enums and associated values.

## Mid-Level Interview Answer

Question: What is `if case` used for?

Answer:

`if case` is used when we want to check one specific pattern without writing a full switch. It is useful for checking one enum case and extracting its associated value.

## Senior-Level Interview Answer

Question: How does pattern matching help with app architecture?

Answer:

Pattern matching helps model finite state and domain events clearly. When states are represented as enums with associated values, `switch`, `if case`, and `guard case` let the compiler enforce correct handling. This reduces invalid state combinations, removes stringly typed logic, and improves maintainability in view models, reducers, navigation, analytics, and networking layers.

## Quick Interview Notes

- Pattern matching checks a value's shape.
- `switch` is the most common pattern matching tool.
- `if case` is useful for one pattern.
- `guard case` is useful when one pattern is required to continue.
- `for case` filters matching values during iteration.
- `where` adds extra conditions to a pattern.
- Enums with associated values are a major Swift pattern matching use case.

## Points To Remember

- Prefer enums over strings for finite states.
- Avoid overusing tuple matching for complex domain logic.
- Use `where` to keep patterns expressive but not overloaded.
- Pattern matching makes impossible states harder to represent.
- Senior Swift code often uses pattern matching to make state handling explicit.

## Practice Questions

1. What is pattern matching in Swift?
2. What is the difference between `switch` and `if case`?
3. When would you use `guard case`?
4. How does `where` work with pattern matching?
5. Why are enums powerful with pattern matching?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Pattern Matching And Where Clauses is not only a syntax topic. In production Swift, it affects branch clarity, exhaustiveness, early exits, and reducing nested logic. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Pattern Matching And Where Clauses in an app feature:

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

1. Write a minimal example that shows Pattern Matching And Where Clauses correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Pattern Matching And Where Clauses, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Pattern Matching And Where Clauses** is evaluated through this lens: control flow is domain modeling in motion: every branch should represent a real state or decision. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Pattern Matching And Where Clauses
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

For **Pattern Matching And Where Clauses**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

