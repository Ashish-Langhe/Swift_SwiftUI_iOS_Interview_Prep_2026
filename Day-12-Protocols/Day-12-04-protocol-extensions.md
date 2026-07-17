# Day 12: Protocol Extensions

## Core Idea

Protocol extensions provide default behavior.

```swift
protocol Loggable {
    var logName: String { get }
}

extension Loggable {
    func log() {
        print(logName)
    }
}
```

## Real iOS Use Cases

- Shared default behavior
- Convenience helpers
- Reducing duplicate conformer code

## Interview Levels

Junior: Protocol extensions add default methods.

Senior: Protocol extensions are powerful, but be careful with static dispatch differences and hidden default behavior. Keep defaults simple and unsurprising.

## Quick Notes

- Add default implementations.
- Can constrain extensions with `where`.
- Helps protocol-oriented programming.

## Interview Depth

Junior answer: Protocol extensions add shared methods or default implementations.

Mid-level answer: They reduce duplicate code across conforming types.

Senior answer: Default implementations should be unsurprising. Understand whether a method is a protocol requirement or only an extension method, because dispatch behavior can differ.

iOS use case:

```swift
extension AnalyticsTracking {
    func trackScreen(_ name: String) {
        track("screen_view:\(name)")
    }
}
```

Common mistakes: hiding important behavior in defaults, making extensions too broad, relying on confusing dispatch.

Practice: add default method, add constrained extension, explain dispatch caution.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Protocol Extensions is not only a syntax topic. In production Swift, it affects capability modeling, abstraction boundaries, associated types, existentials, and testability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Protocol Extensions in an app feature:

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

1. Write a minimal example that shows Protocol Extensions correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Protocol Extensions, but it shows the kind of production shape you should connect this topic to:

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

