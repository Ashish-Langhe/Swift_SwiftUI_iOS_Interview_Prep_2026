# Day 19: Global Actors

## What Global Actors Are

A global actor provides one shared actor isolation domain.

The most common global actor is `MainActor`.

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
}
```

All isolated members are protected by the main actor.

## Why Global Actors Exist

Some state belongs to one global execution context.

Examples:

- UI state on `MainActor`
- Persistence coordinator
- Analytics pipeline
- App-wide session state

## Custom Global Actor

```swift
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
}

@DatabaseActor
final class DatabaseService {
    func save(_ user: User) {
        // Serialized through DatabaseActor.
    }
}
```

Now all `@DatabaseActor` code shares one isolation domain.

## Main Actor

```swift
@MainActor
func updateUI() {
    title = "Loaded"
}
```

Use for:

- UIKit updates
- SwiftUI observable state
- UI navigation
- View model state consumed by UI

## Global Actor On Protocols

```swift
@MainActor
protocol ScreenViewModel: ObservableObject {
    func load() async
}
```

Conforming types are expected to satisfy main-actor requirements.

This is powerful but becomes part of the API contract.

## Crossing Isolation

```swift
func backgroundWork() async {
    let value = await service.load()

    await MainActor.run {
        self.title = value.title
    }
}
```

Crossing into a global actor from outside usually requires `await`.

## Real iOS Use Case

```swift
@MainActor
final class SearchViewModel: ObservableObject {
    @Published private(set) var results: [SearchResult] = []

    private let service: SearchService

    func search(_ query: String) async {
        do {
            let results = try await service.search(query)
            self.results = results
        } catch {
            self.results = []
        }
    }
}
```

This keeps UI state serialized.

## Common Mistakes

- Marking heavy services `@MainActor`
- Adding global actor isolation to public protocols casually
- Forgetting that global actor annotations affect callers
- Using custom global actors when a normal actor instance would work
- Treating actor isolation as security

## Modern Swift 6.x Notes

Swift 6.2 default actor isolation can infer main-actor isolation in selected targets, reducing boilerplate for UI-heavy code.

But explicit global actors are still important for:

- Public APIs
- Framework boundaries
- Non-main global resources
- Documentation of isolation behavior

## Junior Interview Answer

A global actor is a shared actor used to isolate code across many types or functions. `MainActor` is the main example for UI work.

## Mid-Level Interview Answer

I mark UI view models and UI-updating methods `@MainActor`. I use custom global actors only when many independent types must share one serialized execution domain.

## Senior Interview Answer

Global actors are API-level isolation tools. They are useful for UI and shared global resources, but they should be applied carefully because isolation annotations affect callers, protocol conformances, testing, and public API evolution.

## Practice

1. Mark a view model as `@MainActor`.
2. Create a custom `DatabaseActor`.
3. Explain when a normal actor is better than a global actor.
4. Identify a public API where `@MainActor` would affect callers.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Global Actors is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Global Actors in an app feature:

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

1. Write a minimal example that shows Global Actors correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Global Actors, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Global Actors** is evaluated through this lens: modern Swift concurrency is data-race prevention; senior engineers make ownership and isolation compiler-visible. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Global Actors
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

For **Global Actors**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Global Actors** to code you might write in a SwiftUI/UIKit feature.

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

### Keep UI state isolated

```swift
@MainActor
func apply(_ state: ViewState) {
    self.state = state
}
```

UI mutations should be isolated to the main actor.

### Why This Fits Global Actors

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

