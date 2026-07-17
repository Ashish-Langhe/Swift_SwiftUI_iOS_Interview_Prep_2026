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
