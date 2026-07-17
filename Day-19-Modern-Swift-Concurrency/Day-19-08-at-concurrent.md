# Day 19: `@concurrent`

## What `@concurrent` Means

`@concurrent` marks code that should run concurrently rather than remaining serialized on the caller's actor.

In Swift 6.2's approachable concurrency model, this makes concurrent execution explicit.

```swift
@concurrent
func decode(_ data: Data) async throws -> Image {
    try await decoder.decode(data)
}
```

## Why It Exists

If default isolation keeps code on the main actor, you need a clear way to say:

```text
This work should run off the actor because it is expensive or independently concurrent.
```

That is where `@concurrent` fits.

## Real iOS Example

```swift
struct ImagePipeline {
    static func image(from url: URL) async throws -> UIImage {
        let (data, _) = try await URLSession.shared.data(from: url)
        return try await decode(data)
    }

    @concurrent
    static func decode(_ data: Data) async throws -> UIImage {
        guard let image = UIImage(data: data) else {
            throw ImageError.invalidData
        }
        return image
    }
}
```

Network wait and decoding are separated. CPU work does not need to sit on UI isolation.

## What To Put In `@concurrent`

Good candidates:

- Image decoding
- JSON parsing for large payloads
- Compression
- Hashing
- Data transformation
- File processing
- CPU-heavy pure functions

Bad candidates:

- Direct UIKit updates
- Main-actor state mutation
- Unsafely shared mutable class access
- Code that depends on actor-isolated state

## Data Transfer Still Matters

The data passed into concurrent code should be safe to send.

```swift
struct DecodeRequest: Sendable {
    let data: Data
    let scale: CGFloat
}
```

Do not use `@concurrent` to bypass isolation rules.

## Common Mistakes

- Marking UI methods `@concurrent`
- Passing non-sendable mutable references
- Using it as a warning silencer
- Forgetting cancellation in expensive work
- Creating too much parallel CPU work

## Modern Swift 6.2 Notes

`@concurrent` is part of Swift 6.2's approachable concurrency story. It pairs with default actor isolation and caller-context async behavior by making opt-in concurrency visible at the function boundary.

## Junior Interview Answer

`@concurrent` marks async work that should be allowed to run concurrently instead of staying on the caller's actor.

## Mid-Level Interview Answer

I use `@concurrent` for CPU-heavy or independent work like image decoding, while keeping UI state on the main actor.

## Senior Interview Answer

`@concurrent` is an API signal. It says this operation is intentionally concurrent, so its inputs and captured state must be safe to transfer. I use it to separate expensive work from actor-isolated UI flows without weakening data-race safety.

## Practice

1. Mark image decoding as `@concurrent`.
2. Explain why UI updates should not be `@concurrent`.
3. Create a `Sendable` request type for concurrent work.
4. Add cancellation checks to a CPU-heavy concurrent loop.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

@concurrent is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying @concurrent in an app feature:

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

1. Write a minimal example that shows @concurrent correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use @concurrent, but it shows the kind of production shape you should connect this topic to:

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

