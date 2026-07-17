# Day 10: Enums Interview Guide

## One-Minute Interview Answer

Enums model a finite set of states. Swift enums are powerful because they support associated values, raw values, methods, computed properties, protocol conformance, recursive cases, `CaseIterable`, and pattern matching. I use enums to replace stringly typed state, model screen states, define routes and actions, and make invalid state combinations impossible. In SwiftUI, enum-driven UI works well because `switch` rendering is exhaustive.

## Modern Swift 6.x Notes

Swift 6.3 added `@c` support that can expose Swift enums to C for interop scenarios. For iOS interviews, the bigger everyday point is still exhaustive switching and enum-driven state modeling. Swift 6 concurrency also benefits from immutable enum state snapshots.

## Junior Questions

What is an enum?

A type with a fixed set of cases.

What are associated values?

Extra data attached to a specific case.

## Senior Questions

Why are enums powerful in app architecture?

They make finite states explicit and prevent invalid combinations.

Raw value or associated value?

Raw values are fixed constants; associated values carry dynamic case-specific data.

## Common Traps

- Using strings for states
- Adding `default` and hiding future enum cases
- Using many booleans instead of one enum state
- Confusing raw values and associated values

## Topic-By-Topic Deep Dive

### Basic Enums

```swift
enum AuthState {
    case loggedOut
    case loggingIn
    case loggedIn
}
```

Enums replace fragile strings and make state finite.

### Associated Values

```swift
enum AuthState {
    case loggedOut
    case loggedIn(User)
    case failed(String)
}
```

Each case carries only the data it needs.

### Raw Values

```swift
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
}
```

Use raw values for persistence, API constants, and simple interop.

### Recursive Enums

```swift
indirect enum MenuItem {
    case item(String)
    case group(String, [MenuItem])
}
```

Great for trees, menus, expressions, and nested structures.

### CaseIterable

```swift
enum Filter: CaseIterable {
    case all
    case favorites
}
```

Useful for static UI choices.

### Identifiable Enums

```swift
enum Sheet: Identifiable {
    case settings
    case profile(String)

    var id: String {
        switch self {
        case .settings: return "settings"
        case .profile(let id): return "profile-\(id)"
        }
    }
}
```

Great for SwiftUI sheets and routes.

### State Modeling In iOS

Avoid:

```swift
var isLoading = false
var user: User?
var error: String?
```

Prefer:

```swift
enum ProfileState {
    case loading
    case loaded(User)
    case failed(String)
}
```

### Enum-Driven UI

```swift
switch state {
case .loading:
    ProgressView()
case .loaded(let user):
    ProfileView(user: user)
case .failed(let message):
    Text(message)
}
```

Senior answer:

Enums make UI rendering exhaustive and prevent invalid combinations.

## Production Decision Table

| Need | Enum Feature |
| --- | --- |
| Fixed set of choices | Basic enum |
| State with data | Associated values |
| API/persistence value | Raw values |
| Tree-like structure | Recursive enum |
| All static choices | `CaseIterable` |
| SwiftUI identity | `Identifiable` |
| Screen state | Enum state machine |

## Final Revision

- Basic enum: fixed cases
- Associated values: case-specific data
- Raw values: fixed backing values
- Recursive enums need `indirect`
- `CaseIterable` gives `allCases`
- `Identifiable` helps SwiftUI
- Enums are ideal for state modeling

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Enums Interview Guide is not only a syntax topic. In production Swift, it affects finite state modeling, associated data, exhaustive switching, and invalid state prevention. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while loading states, screen routes, API errors, and enum-driven UI. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Enums Interview Guide in an app feature:

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

1. Write a minimal example that shows Enums Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Enums Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
enum Loadable<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(String)
}

func render(_ state: Loadable<[Product]>) -> String {
    switch state {
    case .idle: return "Idle"
    case .loading: return "Loading"
    case .loaded(let products): return "Products: \(products.count)"
    case .failed(let message): return message
    }
}
```

Enums prevent invalid combinations like `isLoading == true` while also having an error and stale data unless you intentionally model that state.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Enums Interview Guide** is evaluated through this lens: enums are invalid-state prevention; senior engineers use them to model finite domain states explicitly. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a state-machine artifact listing cases, associated values, allowed transitions, and UI output for each case
Topic: Enums Interview Guide
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine modeling loading, routing, permissions, API errors, empty states, and feature modes. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is separate booleans that allow impossible combinations or raw values leaking into domain logic.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Are all states representable?
- Are impossible states excluded?
- Does switch exhaustiveness help future changes?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Enums Interview Guide**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

