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
