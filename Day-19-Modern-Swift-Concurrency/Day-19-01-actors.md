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

