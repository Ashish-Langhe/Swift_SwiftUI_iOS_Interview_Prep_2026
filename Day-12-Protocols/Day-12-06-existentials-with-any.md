# Day 12: Existentials With Any

## Core Idea

`any Protocol` means a value can hold any concrete type conforming to the protocol.

```swift
protocol AnalyticsTracking {
    func track(_ event: String)
}

let tracker: any AnalyticsTracking = ConsoleTracker()
```

## Why `any` Matters

It makes existential type erasure explicit.

## Real iOS Use Cases

- Heterogeneous arrays
- Runtime-selected services
- Dependency storage

## Modern Swift 6.x Notes

Swift 6.2 notes mention Embedded Swift support for `any` types for class-constrained protocols. Modern Swift encourages writing `any` explicitly when using existential values.

## Interview Levels

Junior: `any` stores any conforming type.

Senior: `any` has dynamic dispatch and hides the concrete type. It is flexible but may lose associated type information and optimization opportunities.

## Quick Notes

- `any Protocol` is existential.
- Hides concrete type.
- Useful for storage.
- Not the same as generics.

## Interview Depth

Junior answer: `any Protocol` can store any value that conforms to the protocol.

Mid-level answer: `any` makes existential usage explicit. It hides the concrete type behind a protocol value.

Senior answer: Existentials are flexible but trade away some static type information. Use them for storage and runtime polymorphism; use generics when you want compile-time type preservation.

iOS use case:

```swift
let tracker: any AnalyticsTracking = ConsoleAnalyticsTracker()
```

Common mistakes: confusing `any` and `some`, using `any` when generic would be clearer, losing associated type relationships.

Practice: store protocol value, compare with generic function, explain existential.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Existentials With Any is not only a syntax topic. In production Swift, it affects capability modeling, abstraction boundaries, associated types, existentials, and testability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while service contracts, dependency injection, adapters, and mocks. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Existentials With Any in an app feature:

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

- Expose protocols around behavior, not around every concrete type by habit.
- For public protocols, remember that adding a new requirement can break conforming clients.

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

1. Write a minimal example that shows Existentials With Any correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Existentials With Any, but it shows the kind of production shape you should connect this topic to:

```swift
protocol UserLoading {
    func user(id: User.ID) async throws -> User
}

struct ProfileFeature {
    let loader: UserLoading

    func title(for id: User.ID) async throws -> String {
        try await loader.user(id: id).name
    }
}
```

Protocols should describe a capability the feature needs. This keeps the feature testable without exposing the concrete network client.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

