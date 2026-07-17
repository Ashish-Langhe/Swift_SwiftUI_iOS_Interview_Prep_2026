# Day 18: Async Sequences

## What An Async Sequence Is

An async sequence produces values over time.

Normal sequence:

```swift
for item in items {
    print(item)
}
```

Async sequence:

```swift
for await value in stream {
    print(value)
}
```

Use async sequences when values arrive gradually instead of all at once.

## Beginner Mental Model

An array gives you values now.

An async sequence gives you values later, one by one.

Examples:

- Notifications
- Lines from a file
- WebSocket messages
- Location updates
- Progress updates
- State observation
- Async streams

## `for await`

```swift
for await message in messages {
    print(message)
}
```

If the sequence can throw:

```swift
do {
    for try await line in fileLines {
        print(line)
    }
} catch {
    print(error)
}
```

## Async Sequence And Cancellation

A `for await` loop usually runs until:

- The sequence ends
- The task is cancelled
- The loop breaks
- An error is thrown

```swift
for await event in events {
    if Task.isCancelled { break }
    handle(event)
}
```

## Real iOS Example: Notification Stream

Modern Foundation continues to improve typed async-friendly notification APIs, but the mental model is the same: observe values over time.

```swift
for await notification in NotificationCenter.default.notifications(
    named: UIApplication.didBecomeActiveNotification
) {
    await refresh()
}
```

This is easier to reason about than manually adding and removing observers.

## Real iOS Example: Progress

```swift
func observeUploadProgress(_ progress: AsyncStream<Double>) async {
    for await value in progress {
        self.progress = value
    }
}
```

Each progress value updates UI.

## Creating A Simple AsyncStream

```swift
func countdown(from start: Int) -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for value in stride(from: start, through: 0, by: -1) {
                continuation.yield(value)
                try? await Task.sleep(for: .seconds(1))
            }
            continuation.finish()
        }
    }
}
```

Usage:

```swift
for await value in countdown(from: 3) {
    print(value)
}
```

## Swift 6.2 Observations

Swift 6.2 introduced `Observations`, an async sequence for streaming transactional changes from observable state.

Conceptually:

```swift
for await value in Observations({ model.status }) {
    print(value)
}
```

This matters because modern Swift increasingly models long-lived change streams as async sequences.

## When To Use Async Sequence

Use async sequence when:

- There can be multiple values
- Values arrive over time
- The consumer should use async/await style
- Cancellation should stop observation
- You want structured control instead of callback registration

Do not use it for a single value. Use an async function instead.

## Common Mistakes

- Using async sequence for one-time network calls
- Forgetting that `for await` can run forever
- Updating UI outside the main actor
- Not finishing custom streams
- Retaining self forever in stream producers
- Ignoring cancellation cleanup

## Junior Interview Answer

An async sequence is like a sequence whose values arrive asynchronously. You consume it with `for await`.

## Mid-Level Interview Answer

I use async sequences for streams of values like notifications, progress, WebSocket messages, and state changes. The consuming task can be cancelled, which stops observation.

## Senior Interview Answer

AsyncSequence is Swift's abstraction for asynchronous streams. It composes with structured concurrency, cancellation, and actor isolation. I use it to replace callback-based observers when the domain naturally produces many values over time, and I design custom streams carefully so they finish and clean up resources.

## Points To Remember

- Single async result: async function.
- Multiple async values: async sequence.
- Consume with `for await`.
- Throwing streams use `for try await`.
- Long-lived streams need cancellation and cleanup.

## Practice

1. Write a countdown `AsyncStream`.
2. Consume notifications using `for await`.
3. Explain why a WebSocket fits async sequence better than a plain async function.
4. Add cancellation cleanup to a stream producer.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Async Sequences is not only a syntax topic. In production Swift, it affects suspension, task lifetime, structured work, cancellation, priority, streams, and UI isolation. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while screen loading, search debounce, progress streams, and main-actor view models. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Async Sequences in an app feature:

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

1. Write a minimal example that shows Async Sequences correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Async Sequences, but it shows the kind of production shape you should connect this topic to:

```swift
@MainActor
final class SearchViewModel: ObservableObject {
    @Published private(set) var results: [SearchResult] = []
    private var searchTask: Task<Void, Never>?

    func search(_ query: String) {
        searchTask?.cancel()
        searchTask = Task {
            do {
                try await Task.sleep(for: .milliseconds(300))
                let results = try await service.search(query)
                try Task.checkCancellation()
                self.results = results
            } catch is CancellationError {
                return
            } catch {
                self.results = []
            }
        }
    }
}
```

Concurrency basics become real in screens like search: debounce, cancel stale work, await the service, and update UI state on the main actor.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Async Sequences** is evaluated through this lens: basic concurrency is lifecycle design; senior engineers pair every async operation with ownership, cancellation, and UI isolation. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a task-lifecycle artifact showing task owner, cancellation trigger, actor context, priority, and result application point
Topic: Async Sequences
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine building search, refresh, progress, infinite scroll, async loading, and main-actor view models. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is orphaned tasks, stale UI updates, accidental main-actor heavy work, and swallowed cancellation.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Who cancels this task?
- Where does it resume?
- Can stale results update the UI?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Async Sequences**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Async Sequences** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Lifecycle-Aware SwiftUI Task

```swift
struct ProductsView: View {
    @StateObject private var viewModel = ProductsViewModel()

    var body: some View {
        List(viewModel.products) { product in
            Text(product.title)
        }
        .task {
            await viewModel.load()
        }
    }
}
```

`.task` ties async loading to the view lifecycle.

### Example 2: Cooperative Cancellation

```swift
func process(items: [Item]) async throws {
    for item in items {
        try Task.checkCancellation()
        try await process(item)
    }
}
```

Long-running async work should check cancellation at safe points.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Consume values over time

```swift
for await progress in uploadProgressStream {
    self.progress = progress
}
```

Async sequences are ideal when a feature receives multiple values over time.

### Why This Fits Async Sequences

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

