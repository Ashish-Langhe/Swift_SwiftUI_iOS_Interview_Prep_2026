# Day 13: Generic Functions

## Core Idea

Generic functions work with placeholder types.

```swift
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}
```

## Real iOS Use Cases

- Reusable decoding helpers
- Generic cache helpers
- Common collection utilities

```swift
func decode<T: Decodable>(_ type: T.Type, from data: Data) throws -> T {
    try JSONDecoder().decode(T.self, from: data)
}
```

## Interview Levels

Junior: Generics let one function work with many types.

Senior: Generics preserve type safety and static dispatch while avoiding duplicate code.

## Quick Notes

- Placeholder type like `T`.
- Type-safe reuse.
- Often constrained with protocols.

## Interview Depth

Junior answer: A generic function works with more than one type.

Mid-level answer: Generics avoid duplicate functions while keeping type safety.

Senior answer: Generic functions are compile-time abstraction. They are often better than `Any` because the compiler preserves input and output type relationships.

iOS use case:

```swift
func loadJSON<T: Decodable>(_ type: T.Type, from data: Data) throws -> T {
    try JSONDecoder().decode(T.self, from: data)
}
```

Common mistakes: using `Any`, overgeneric APIs, constraints that are too broad.

Practice: generic first element, generic decode, add constraint.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Generic Functions is not only a syntax topic. In production Swift, it affects type-safe reuse, constraints, abstraction cost, and preserving static information. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while network clients, repositories, reusable UI models, and type-erased dependencies. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Generic Functions in an app feature:

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

- Use generics when the caller benefits from preserving concrete type information.
- Reach for type erasure only when storage or heterogeneous collections require it.

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

1. Write a minimal example that shows Generic Functions correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Generic Functions, but it shows the kind of production shape you should connect this topic to:

```swift
struct Endpoint<Response: Decodable> {
    let path: String
    let decode: (Data) throws -> Response
}

func send<Response>(_ endpoint: Endpoint<Response>) async throws -> Response {
    let data = try await loadData(path: endpoint.path)
    return try endpoint.decode(data)
}
```

Generics let the compiler preserve the response type from the endpoint to the call site, removing casts and reducing runtime mistakes.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Generic Functions** is evaluated through this lens: generics preserve type information; senior engineers use them when static guarantees are worth the added abstraction. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a generic API contract showing type parameter, constraints, caller benefit, and where type erasure begins
Topic: Generic Functions
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine building typed network endpoints, reusable repositories, containers, result models, and strongly typed UI config. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is generic complexity that hides intent or premature type erasure that loses compiler guarantees.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- What concrete type does the caller keep?
- Are constraints minimal?
- Would a protocol existential be simpler?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Generic Functions**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Generic Functions** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Generic Response Wrapper

```swift
struct APIResponse<Value: Decodable>: Decodable {
    let value: Value
    let requestID: String
}
```

Generics preserve the decoded payload type while sharing wrapper logic.

### Example 2: Reusable Cache

```swift
final class Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]

    subscript(key: Key) -> Value? {
        get { storage[key] }
        set { storage[key] = newValue }
    }
}
```

Generic types are useful when the behavior is the same but the stored types vary.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Small realistic usage

```swift
struct ExampleState: Equatable {
    var title: String
    var isEnabled: Bool
}

let state = ExampleState(title: "Continue", isEnabled: true)
print(state.title)
```

When studying the topic, rewrite this generic shape into the exact model your screen needs.

### Why This Fits Generic Functions

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

