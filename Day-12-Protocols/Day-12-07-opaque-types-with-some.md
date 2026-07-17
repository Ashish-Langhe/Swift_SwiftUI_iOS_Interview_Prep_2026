# Day 12: Opaque Types With Some

## Core Idea

`some Protocol` hides a concrete return type while preserving it for the compiler.

```swift
func makeTitle() -> some CustomStringConvertible {
    "Hello"
}
```

SwiftUI uses this heavily:

```swift
var body: some View {
    Text("Hello")
}
```

## Any Vs Some

`any` means any conforming type at runtime.

`some` means one specific hidden concrete type chosen by the implementation.

## Interview Levels

Junior: `some` returns a value conforming to a protocol.

Senior: `some` preserves static type information and optimization while hiding implementation details. It is great for return positions, especially SwiftUI views.

## Quick Notes

- Opaque type.
- Concrete type hidden from caller.
- Compiler still knows exact type.
- Common in SwiftUI.

## Interview Depth

Junior answer: `some Protocol` returns a value that conforms to a protocol.

Mid-level answer: The implementation chooses one concrete type, but callers do not need to know the exact type.

Senior answer: `some` hides implementation while preserving static type information and optimization. It is especially useful in return positions, such as SwiftUI `some View`.

iOS use case:

```swift
var body: some View {
    Text("Profile")
}
```

Common mistakes: thinking `some` means any type can be returned from different branches, confusing with `any`, using where stored existential is needed.

Practice: explain `some View`, compare any/some, identify branch return issue.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Opaque Types With Some is not only a syntax topic. In production Swift, it affects capability modeling, abstraction boundaries, associated types, existentials, and testability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Opaque Types With Some in an app feature:

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

1. Write a minimal example that shows Opaque Types With Some correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Opaque Types With Some, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Opaque Types With Some** is evaluated through this lens: protocols are boundary tools; senior engineers introduce them to express capability and dependency direction, not decoration. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a protocol-boundary artifact showing concrete owner, protocol consumer, test double, and public/internal visibility
Topic: Opaque Types With Some
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine designing repositories, services, adapters, mockable dependencies, and module contracts. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is protocols with no alternate implementation or public protocols that become hard to evolve.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Who consumes the abstraction?
- Is there a real substitution point?
- Will adding a requirement break clients?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Opaque Types With Some**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

