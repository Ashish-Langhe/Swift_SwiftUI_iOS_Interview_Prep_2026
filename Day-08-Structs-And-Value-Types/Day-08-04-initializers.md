# Day 8: Initializers

## Core Idea

Initializers create valid instances.

```swift
struct User {
    let id: String
    let name: String

    init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}
```

Structs often get a memberwise initializer automatically.

## Validation

```swift
struct Email {
    let value: String

    init?(_ value: String) {
        guard value.contains("@") else { return nil }
        self.value = value
    }
}
```

## Interview Levels

Junior:

Initializers set up values.

Senior:

Initializers should establish invariants. Failable initializers are useful when invalid input should not create an instance.

## Quick Notes

- `init` creates instances
- Structs get memberwise init
- Use failable init for invalid input
- Initialization should create valid state

## Interview Depth

Junior answer: An initializer creates an instance and sets its properties.

Mid-level answer: Structs get a memberwise initializer automatically, but custom initializers are useful for validation, transformation, or defaults.

Senior answer: Initializers should establish invariants. If an instance cannot be valid with bad input, use a failable or throwing initializer instead of creating a broken object.

iOS use case:

```swift
struct EmailAddress {
    let value: String

    init?(_ value: String) {
        guard value.contains("@") else { return nil }
        self.value = value
    }
}
```

Common mistakes:

- Allowing invalid model state.
- Doing too much work in init.
- Force-unwrapping inside init.

Practice:

1. Create failable initializer.
2. Add default parameter in init.
3. Explain invariant.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Initializers is not only a syntax topic. In production Swift, it affects value semantics, invariants, copying behavior, and predictable state changes. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Initializers in an app feature:

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

1. Write a minimal example that shows Initializers correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Initializers, but it shows the kind of production shape you should connect this topic to:

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

