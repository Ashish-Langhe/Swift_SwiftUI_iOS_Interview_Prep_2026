# Day 19: Modern Swift Concurrency Interview Guide

## One-Minute Interview Answer

Modern Swift concurrency is about data-race safety. Actors protect mutable state, global actors like `MainActor` isolate UI, `Sendable` controls values crossing concurrency boundaries, and strict concurrency checks catch unsafe access at compile time. Swift 6 made data-race safety a major language-mode feature, while Swift 6.2 made concurrency more approachable with default actor isolation, caller-context async behavior, explicit `@concurrent`, named tasks, and better debugging.

## Junior Questions

What is an actor?

An actor is a reference type that protects mutable state by serializing access.

What is `MainActor`?

The global actor used for UI-related work.

What is `Sendable`?

A protocol indicating a type can be safely passed across concurrency boundaries.

## Mid-Level Questions

When do you use an actor?

For shared mutable state such as caches, token stores, download coordinators, and analytics buffers.

When do you use `@MainActor`?

For UI state and UI-updating methods.

What is strict concurrency?

Compiler checking for potential data races, unsafe actor access, and non-sendable sharing.

## Senior Questions

What is actor reentrancy?

An actor may process other work while a method is suspended at `await`. Do not assume actor state remains unchanged across suspension points.

How do you migrate to Swift 6 concurrency?

Classify state ownership, mark UI state `@MainActor`, convert shared mutable services to actors or safe synchronization, make transferred values `Sendable`, and avoid unsafe escape hatches unless justified.

What is the value of Swift 6.2 approachable concurrency?

It reduces boilerplate for common UI-style code while making intentional concurrent execution explicit with `@concurrent`.

## Scenario: Token Refresh

Strong design:

```swift
actor AuthTokenStore {
    private var token: String?
    private var refreshTask: Task<String, Error>?

    func validToken() async throws -> String {
        if let token { return token }

        if let refreshTask {
            return try await refreshTask.value
        }

        let task = Task { try await requestNewToken() }
        refreshTask = task

        do {
            let token = try await task.value
            self.token = token
            refreshTask = nil
            return token
        } catch {
            refreshTask = nil
            throw error
        }
    }
}
```

Why this is senior:

- Shared mutable token state is actor-isolated.
- Duplicate refreshes are coalesced.
- Errors reset state.
- Callers use async safely.

## Scenario: UI ViewModel

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published private(set) var state: State = .idle

    func load() async {
        state = .loading
        do {
            let profile = try await service.profile()
            state = .loaded(profile)
        } catch is CancellationError {
            return
        } catch {
            state = .failed(error.localizedDescription)
        }
    }
}
```

Strong points:

- UI state is main-actor isolated.
- Errors and cancellation are separated.
- External mutation is controlled.

## Common Traps

- "Actor means no race conditions" is incomplete.
- `@MainActor` everywhere can hide performance problems.
- `@unchecked Sendable` is not a real fix by itself.
- `Task.detached` should not be a default migration tool.
- Public protocol isolation affects all conformers.
- `private` is not concurrency isolation.

## Latest Swift 6.x Notes

- Swift 6 language mode strengthens data-race safety.
- Swift 6.2 adds default actor isolation settings.
- Swift 6.2 introduces explicit `@concurrent` for work that should run concurrently.
- Swift 6.2 improves async debugging and task context.
- `Sendable` and actor isolation remain core concepts.

## Interview Closing Answer

I design Swift concurrency around ownership. UI state belongs to the main actor, shared mutable services belong behind actors or synchronization, data crossing boundaries should be immutable and `Sendable`, and long-running flows should respect cancellation. Swift 6.x gives compiler support for these choices, but the architecture still needs clear boundaries.

## Practice Prompts

1. Explain actor reentrancy with a bank account example.
2. Migrate a mutable singleton cache to an actor.
3. Decide whether a model should be `Sendable`.
4. Explain Swift 6.2 approachable concurrency in two minutes.
5. Design a public async API with actor and sendability in mind.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Modern Swift Concurrency Interview Guide is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while token stores, caches, UI models, Swift 6 migration, and shared services. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Modern Swift Concurrency Interview Guide in an app feature:

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
- Always separate task lifetime from object lifetime; a task can outlive the screen that started it.
- State crossing a concurrency boundary should be immutable, actor-isolated, or clearly Sendable.

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

1. Write a minimal example that shows Modern Swift Concurrency Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Modern Swift Concurrency Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
actor UserCache {
    private var storage: [User.ID: User] = [:]

    func user(id: User.ID) -> User? {
        storage[id]
    }

    func insert(_ user: User) {
        storage[user.id] = user
    }
}
```

Modern Swift concurrency asks who owns mutable state. If many tasks need the same cache, actor isolation is clearer than hoping callers use it safely.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

