# Day 20: Advanced Concurrency And Networking Interview Guide

## One-Minute Interview Answer

For advanced Swift networking, I use URLSession async APIs, validate HTTP responses, decode with clear error modeling, run independent requests in parallel with `async let`, use task groups for dynamic parallel work, model long-lived streams with `AsyncStream`, implement cancellation-aware retry and timeout helpers, and protect shared service state with actors. In Swift 6.x, I also design request/response types to be `Sendable`, isolate UI updates to `MainActor`, and avoid shared mutable state in child tasks.

## Junior Questions

How do you make an async network request?

Use `try await URLSession.shared.data(from:)` or `data(for:)`.

When do you use `async let`?

For a small fixed number of independent async operations.

What is a task group?

A structured way to run a dynamic number of child tasks.

## Mid-Level Questions

How do you validate a network response?

Check `HTTPURLResponse`, status code range, content type if needed, then decode.

How do you avoid stale search results?

Cancel previous tasks, check cancellation before applying results, and update UI on the main actor.

How do you wrap delegate callbacks in async/await?

Use `AsyncStream` or `AsyncThrowingStream` with `yield`, `finish`, and `onTermination`.

## Senior Questions

How do you design retry logic?

Retry only transient failures, use bounded attempts, exponential backoff, preserve cancellation, and avoid blindly retrying non-idempotent operations.

How do you prevent duplicate token refresh?

Use an actor with an in-flight refresh task so concurrent callers await the same refresh.

How do you limit parallel requests?

Use a task group and add only a fixed number of child tasks at a time.

## Architecture Example

```text
ProfileViewModel @MainActor
  -> ProfileService protocol
      -> APIClient
          -> TokenProvider actor
          -> URLSession
```

Responsibilities:

- ViewModel owns UI state.
- Service owns domain operations.
- APIClient owns HTTP mechanics.
- TokenProvider actor owns shared auth state.
- URLSession performs transport.

## Strong Dashboard Loading Answer

```swift
@MainActor
func load() async {
    state = .loading

    do {
        async let profile = profileService.profile()
        async let feed = feedService.feed()
        async let messages = messageService.unreadCount()

        let data = try await DashboardData(
            profile: profile,
            feed: feed,
            unreadCount: messages
        )

        try Task.checkCancellation()
        state = .loaded(data)
    } catch is CancellationError {
        return
    } catch {
        state = .failed(error.localizedDescription)
    }
}
```

Why this is strong:

- Parallel independent requests
- Main-actor UI updates
- Cancellation respected
- Error path explicit
- No shared mutable child-task state

## Strong Networking Client Answer

```swift
struct APIClient: Sendable {
    let baseURL: URL
    let session: URLSession

    func send<Response: Decodable & Sendable>(
        _ request: URLRequest
    ) async throws -> Response {
        let (data, response) = try await session.data(for: request)
        try validate(response)
        return try JSONDecoder().decode(Response.self, from: data)
    }
}
```

Senior notes:

- Public async APIs should consider `Sendable`.
- Validation is separate.
- Decoding errors are not transport errors.
- UI is not updated in the client.

## Common Traps

- Not checking HTTP status codes
- Launching unlimited task group children
- Mutating arrays from child tasks
- Retrying cancellation
- Retrying payments without idempotency
- Forgetting stream termination cleanup
- Using actors for stateless objects unnecessarily
- Doing heavy decode on the main actor

## Latest Swift 6.x Notes

- Swift 6 data-race safety affects networking architecture.
- Swift 6.2 `@concurrent` helps move expensive decode/transform work out of actor isolation.
- Swift 6.2 async debugging makes named tasks useful in complex network flows.
- Swift 6.2 Foundation includes more concurrency-friendly APIs, including typed notification improvements and observable state streams.

## Final Senior-Level Answer

I design networking around structured concurrency and isolation. Fixed independent work uses `async let`; dynamic work uses task groups with limits; streams use `AsyncStream`; retries are bounded, cancellable, and idempotency-aware; shared mutable service state is actor-isolated; and UI updates stay on `MainActor`. Swift 6.x makes these choices more than style because the compiler now participates in data-race safety.

## Practice Prompts

1. Write a generic async API client.
2. Implement timeout using a task group.
3. Add retry with exponential backoff and cancellation.
4. Wrap WebSocket messages in `AsyncThrowingStream`.
5. Build an actor-isolated image loader with request deduplication.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Advanced Concurrency And Networking Interview Guide is not only a syntax topic. In production Swift, it affects async networking, parallelism, task groups, stream bridging, retry, timeout, and actor-isolated services. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Advanced Concurrency And Networking Interview Guide in an app feature:

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

1. Write a minimal example that shows Advanced Concurrency And Networking Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Advanced Concurrency And Networking Interview Guide, but it shows the kind of production shape you should connect this topic to:

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

