# Day 20: AsyncStream

## What AsyncStream Is

`AsyncStream` lets you wrap callback-style or event-based APIs as an async sequence.

```swift
func values() -> AsyncStream<Int> {
    AsyncStream { continuation in
        continuation.yield(1)
        continuation.yield(2)
        continuation.finish()
    }
}
```

Consume:

```swift
for await value in values() {
    print(value)
}
```

## When To Use AsyncStream

Use it for:

- Delegate callbacks
- Timer events
- Progress updates
- WebSocket messages
- Location updates
- Legacy observer APIs

Do not use it for a single network result.

## Wrapping A Delegate

```swift
final class LocationStream: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private var continuation: AsyncStream<CLLocation>.Continuation?

    func locations() -> AsyncStream<CLLocation> {
        AsyncStream { continuation in
            self.continuation = continuation
            manager.delegate = self
            manager.startUpdatingLocation()

            continuation.onTermination = { [weak self] _ in
                self?.manager.stopUpdatingLocation()
                self?.continuation = nil
            }
        }
    }

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        locations.forEach { continuation?.yield($0) }
    }
}
```

## AsyncThrowingStream

Use `AsyncThrowingStream` when the stream can fail.

```swift
func messages() -> AsyncThrowingStream<Message, Error> {
    AsyncThrowingStream { continuation in
        socket.onMessage = { continuation.yield($0) }
        socket.onError = { continuation.finish(throwing: $0) }
        socket.onClose = { continuation.finish() }
    }
}
```

## Cleanup Is Critical

Always handle termination:

```swift
continuation.onTermination = { _ in
    token.invalidate()
}
```

Without cleanup, streams can leak observers, delegates, sockets, or tasks.

## Backpressure And Buffering

Streams can produce values faster than consumers read them.

Use buffering policies when needed:

```swift
AsyncStream(bufferingPolicy: .bufferingNewest(1)) { continuation in
    // Yield latest value only.
}
```

This is useful for UI progress or location where latest value matters most.

## Common Mistakes

- Not calling `finish()`
- Not cleaning up in `onTermination`
- Capturing self strongly forever
- Using unbounded buffering for high-frequency events
- Updating UI from stream producer instead of consumer
- Ignoring cancellation

## Modern Swift 6.x Notes

Async streams fit Swift's structured concurrency model, but producer code must still be memory-safe and cancellation-aware. With Swift 6 strict checking, captured values and continuation usage should be designed carefully.

Swift 6.2's `Observations` shows the same direction: model state changes as async sequences.

## Interview Answer

`AsyncStream` bridges callback/event APIs into Swift's async sequence world. I use it for repeated values over time, add termination cleanup, choose buffering carefully, and consume it from a cancellable task.

## Practice

1. Wrap a timer as `AsyncStream<Date>`.
2. Wrap a delegate callback as `AsyncStream`.
3. Add `onTermination` cleanup.
4. Explain bufferingNewest for search/progress updates.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

AsyncStream is not only a syntax topic. In production Swift, it affects async networking, parallelism, task groups, stream bridging, retry, timeout, and actor-isolated services. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while API clients, dashboards, image loaders, WebSocket streams, and token refresh coordination. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying AsyncStream in an app feature:

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

1. Write a minimal example that shows AsyncStream correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use AsyncStream, but it shows the kind of production shape you should connect this topic to:

```swift
actor ImagePipeline {
    private var inFlight: [URL: Task<UIImage, Error>] = [:]

    func image(for url: URL) async throws -> UIImage {
        if let task = inFlight[url] {
            return try await task.value
        }

        let task = Task { try await downloadAndDecode(url) }
        inFlight[url] = task
        defer { inFlight[url] = nil }
        return try await task.value
    }
}
```

Advanced networking is not only making requests. It is also deduplicating work, respecting cancellation, protecting shared state, and keeping UI updates outside the transport layer.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

