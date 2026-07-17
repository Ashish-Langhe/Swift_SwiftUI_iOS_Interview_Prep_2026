# Day 19: Actors

## What Actors Are

An actor is a reference type that protects its mutable state from concurrent access.

```swift
actor TokenStore {
    private var token: String?

    func update(_ token: String) {
        self.token = token
    }

    func currentToken() -> String? {
        token
    }
}
```

Only one piece of actor-isolated code runs on an actor at a time.

## Why Actors Exist

Data races happen when multiple concurrent operations access the same mutable state and at least one access is a write.

Actors solve this by serializing access to protected state.

```swift
let store = TokenStore()
await store.update("abc")
let token = await store.currentToken()
```

Calls from outside the actor usually require `await`.

## Actor Isolation

Actor-isolated state can be accessed directly from inside the actor.

```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

Outside code must cross the actor boundary:

```swift
let counter = Counter()
await counter.increment()
```

## Actors Are Reference Types

```swift
let a = Counter()
let b = a
```

`a` and `b` refer to the same actor instance.

Actor does not mean value type. It means reference type with isolated mutable state.

## Reentrancy

Actors can be reentrant at suspension points.

```swift
actor BankAccount {
    private var balance: Decimal = 100

    func withdraw(_ amount: Decimal) async -> Bool {
        guard balance >= amount else { return false }
        await audit()
        balance -= amount
        return true
    }
}
```

After `await audit()`, another call may have run and changed actor state. Senior engineers avoid assuming state is unchanged across suspension.

Better:

```swift
func withdraw(_ amount: Decimal) async -> Bool {
    guard balance >= amount else { return false }
    balance -= amount
    await audit()
    return true
}
```

Update critical state before suspension when appropriate.

## Real iOS Actor Use Cases

Use actors for:

- Auth token storage
- In-memory caches
- Download coordination
- Image loading deduplication
- File write coordination
- Analytics event buffers
- Rate limiters

```swift
actor ImageCache {
    private var images: [URL: UIImage] = [:]

    func image(for url: URL) -> UIImage? {
        images[url]
    }

    func insert(_ image: UIImage, for url: URL) {
        images[url] = image
    }
}
```

## Actor vs Class

Class:

```swift
final class Counter {
    var value = 0
}
```

Unsafe if accessed from multiple tasks.

Actor:

```swift
actor SafeCounter {
    private var value = 0
}
```

Compiler enforces isolated access.

## Common Mistakes

- Thinking actors make all code inside instantly parallel
- Holding invalid assumptions across `await`
- Using actors for everything, including simple value models
- Updating UI from a non-main actor
- Returning mutable non-Sendable state from actors
- Blocking inside an actor

## Modern Swift 6.x Notes

Swift 6 data-race safety makes actor isolation much more important. The compiler can diagnose unsafe access to shared mutable state and actor-isolated members.

Swift 6.2 approachable concurrency reduces annotation overhead in UI-style code, but actors remain the core tool for protecting shared mutable state outside UI.

## Junior Interview Answer

An actor is like a class that protects its mutable state so only one task accesses it at a time.

## Mid-Level Interview Answer

Actors isolate state. Calls from outside an actor usually require `await`, and the compiler prevents direct unsafe access to actor-isolated properties.

## Senior Interview Answer

Actors are isolation domains for mutable reference state. I use them for shared services and caches, design APIs that avoid leaking mutable state, and watch for reentrancy around suspension points. Actors improve data-race safety, but they do not remove the need for careful ordering and cancellation.

## Practice

1. Convert a class-based cache to an actor.
2. Explain why actor calls require `await`.
3. Find a reentrancy bug around an `await`.
4. Design an actor for auth token refresh coordination.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Actors is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Actors in an app feature:

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

1. Write a minimal example that shows Actors correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Actors, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Actors** is evaluated through this lens: modern Swift concurrency is data-race prevention; senior engineers make ownership and isolation compiler-visible. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a concurrency-safety artifact listing actor-isolated state, Sendable values, global actors, nonisolated APIs, and migration risks
Topic: Actors
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine migrating Swift 5 code to Swift 6, isolating caches, designing UI models, and reviewing shared services. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is silencing warnings with unchecked annotations rather than fixing ownership and transfer boundaries.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- What isolation domain owns the state?
- Is this value Sendable?
- Is @unchecked justified and documented?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Actors**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Actors** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Sendable Snapshot

```swift
struct UserSnapshot: Sendable {
    let id: String
    let name: String
}

actor AuditLog {
    func record(_ user: UserSnapshot) { }
}
```

Sendable values are safer to pass across actor/task boundaries.

### Example 2: Main-Actor ViewModel

```swift
@MainActor
final class AccountViewModel: ObservableObject {
    @Published private(set) var title = ""

    func update(title: String) {
        self.title = title
    }
}
```

UI state should have explicit actor isolation.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Protect shared mutable state

```swift
actor CounterStore {
    private var count = 0

    func increment() {
        count += 1
    }
}
```

Actors make the ownership of shared mutable state explicit.

### Why This Fits Actors

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

