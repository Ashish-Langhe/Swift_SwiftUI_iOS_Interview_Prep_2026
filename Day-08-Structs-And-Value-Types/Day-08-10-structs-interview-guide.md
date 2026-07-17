# Day 8: Structs And Value Types Interview Guide

## One-Minute Interview Answer

Structs are value types in Swift. They can have stored properties, computed properties, initializers, methods, and mutating methods. Swift prefers structs because value semantics make state predictable. When a struct is copied, changes to one value do not affect the other. Swift collections use copy-on-write to keep value semantics efficient. I use structs for models, view models, configuration, and state snapshots, and classes when identity or shared mutable reference behavior is required.

## Modern Swift 6.x Notes

Swift 6's concurrency safety direction makes immutable structs and `Sendable` models increasingly important. Value snapshots are easier to pass across tasks than shared mutable class instances.

## Junior Questions

What is a struct?

A type that groups data and behavior.

What is `mutating`?

A keyword required when a struct method changes stored properties.

## Senior Questions

Why prefer structs?

Predictable value semantics, safer state, easier concurrency reasoning, and great fit for SwiftUI.

When use classes?

When identity, inheritance, shared mutable state, or Objective-C interoperability is needed.

## Common Traps

- Forgetting `mutating`
- Using classes for simple models
- Assuming copy-on-write changes behavior
- Putting UI formatting everywhere in domain models

## Topic-By-Topic Deep Dive

### Struct Declaration

```swift
struct UserProfile {
    let id: String
    var displayName: String
}
```

Senior note:

Structs are usually the default for data because their value semantics avoid hidden sharing.

### Stored Properties

Stored properties define state.

```swift
let id: String
var displayName: String
```

Make properties `let` unless domain behavior requires mutation.

### Computed Properties

```swift
var initials: String {
    displayName.split(separator: " ").compactMap(\.first).map(String.init).joined()
}
```

Use for cheap derived values. Avoid hiding expensive work.

### Initializers

Initializers should create valid values.

```swift
init?(email: String) {
    guard email.contains("@") else { return nil }
    self.email = email
}
```

Senior note:

Use failable initializers or throwing initializers to protect invariants.

### Methods

Methods express behavior owned by the value.

```swift
func isOwned(by userId: String) -> Bool {
    ownerId == userId
}
```

### Mutating Methods

```swift
mutating func markCompleted() {
    isCompleted = true
}
```

Mutation is explicit and requires a `var` instance.

### Value Semantics

```swift
var draft = profile
draft.displayName = "New Name"
```

Changing `draft` does not change `profile`.

### Copy-On-Write

Swift collections share storage until mutation. You get logical copying without paying full copy cost immediately.

### Why Swift Prefers Structs

Structs work beautifully with SwiftUI because the UI can render state snapshots.

Senior answer:

Prefer structs for data and classes for identity.

## Production Decision Table

| Need | Use |
| --- | --- |
| Simple model | Struct |
| Shared identity | Class |
| Local state update | Mutating method |
| Derived display value | Computed property |
| Validated construction | Failable/throwing init |
| Large collection value behavior | Copy-on-write |

## Final Revision

- Structs are value types
- Stored properties hold data
- Computed properties derive data
- Initializers create valid instances
- `mutating` changes value state
- Copy-on-write optimizes collections
- Swift prefers structs for predictable state

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Structs And Value Types Interview Guide is not only a syntax topic. In production Swift, it affects value semantics, invariants, copying behavior, and predictable state changes. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while view state, request models, domain entities, and SwiftUI state values. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Structs And Value Types Interview Guide in an app feature:

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

1. Write a minimal example that shows Structs And Value Types Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Structs And Value Types Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
struct SearchState: Equatable {
    var query: String = ""
    var results: [SearchResult] = []
    var isLoading = false

    mutating func startSearch(query: String) {
        self.query = query
        self.isLoading = true
        self.results = []
    }
}
```

Value types are ideal for UI state because each mutation creates a clear before/after model that is easy to compare, test, and reason about.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

