# Day 20: URLSession Async APIs

## Why URLSession Async APIs Matter

Networking is one of the most common uses of Swift concurrency.

```swift
let (data, response) = try await URLSession.shared.data(from: url)
```

This replaces nested completion handlers with readable async code.

## Basic GET Request

```swift
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://example.com/users")!
    let (data, response) = try await URLSession.shared.data(from: url)

    guard let http = response as? HTTPURLResponse,
          (200..<300).contains(http.statusCode) else {
        throw NetworkError.badStatus
    }

    return try JSONDecoder().decode([User].self, from: data)
}
```

## Request-Based API

```swift
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
request.httpBody = try JSONEncoder().encode(body)

let (data, response) = try await URLSession.shared.data(for: request)
```

Use `data(for:)` when you need headers, method, body, auth, or cache policy.

## Typed Endpoint Pattern

```swift
protocol Endpoint {
    associatedtype Response: Decodable

    var path: String { get }
    var method: String { get }
}

struct APIClient {
    let baseURL: URL

    func send<E: Endpoint>(_ endpoint: E) async throws -> E.Response {
        var request = URLRequest(url: baseURL.appending(path: endpoint.path))
        request.httpMethod = endpoint.method

        let (data, response) = try await URLSession.shared.data(for: request)
        try validate(response)
        return try JSONDecoder().decode(E.Response.self, from: data)
    }
}
```

## Error Modeling

```swift
enum NetworkError: Error {
    case badURL
    case badStatus(Int)
    case decoding(Error)
    case transport(Error)
}
```

Good network layers separate:

- URL construction errors
- Transport errors
- HTTP status errors
- Decoding errors
- Domain/API errors
- Cancellation

## Cancellation

URLSession async APIs integrate with task cancellation.

```swift
let task = Task {
    try await client.send(endpoint)
}

task.cancel()
```

Still handle `CancellationError` separately in UI code.

## Main Actor Boundary

Do networking outside UI state when possible.

```swift
@MainActor
func load() async {
    state = .loading
    do {
        let users = try await service.users()
        state = .loaded(users)
    } catch {
        state = .failed(error.localizedDescription)
    }
}
```

The service does network work. The view model updates UI state.

## Common Mistakes

- Not validating HTTP status
- Treating every error as decoding failure
- Updating UI in network service
- Ignoring cancellation
- Decoding huge payloads on the main actor
- Using force unwrap for URLs in production

## Modern Swift 6.x Notes

With Swift 6 data-race safety, networking APIs should consider `Sendable` request/response models, actor-isolated shared state, and main-actor UI boundaries.

Swift 6.2's `@concurrent` can be useful for expensive decode/transform work when default actor isolation would otherwise keep code serialized.

## Interview Answer

I use URLSession async APIs with `data(for:)`, validate status codes, decode into `Decodable` models, separate cancellation from real errors, and keep UI updates on the main actor. For reusable clients, I model endpoints generically and avoid leaking backend DTOs into UI.

## Practice

1. Write a generic `APIClient.send`.
2. Add HTTP status validation.
3. Handle cancellation separately.
4. Move UI state updates out of the network layer.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

URLSession Async APIs is not only a syntax topic. In production Swift, it affects async networking, parallelism, task groups, stream bridging, retry, timeout, and actor-isolated services. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying URLSession Async APIs in an app feature:

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

1. Write a minimal example that shows URLSession Async APIs correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use URLSession Async APIs, but it shows the kind of production shape you should connect this topic to:

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

