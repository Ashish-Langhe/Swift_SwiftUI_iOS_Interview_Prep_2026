# Day 34: API Client Architecture

## Why API Client Architecture Matters

Without structure, networking code spreads everywhere:

- ViewModels build requests.
- Views know endpoints.
- Token refresh logic duplicates.
- Errors are inconsistent.
- Tests hit real network.
- DTOs leak into UI.

A strong API client centralizes transport concerns while keeping domain boundaries clean.

## Layered Design

```text
View
ViewModel
Repository / Use Case
API Client
Request Builder / Authenticator
URLSession
```

Each layer has one role.

## Core Types

```swift
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case patch = "PATCH"
    case delete = "DELETE"
}

struct Endpoint {
    let path: String
    let method: HTTPMethod
    let requiresAuth: Bool
}
```

## API Error

```swift
enum APIError: Error, Equatable {
    case invalidURL
    case invalidResponse
    case unauthorized
    case forbidden
    case notFound
    case validation(String)
    case rateLimited
    case server
    case decoding
    case transport(String)
    case unexpectedStatus(Int)
}
```

Normalize errors so feature code does not depend on random `URLError` and status-code handling everywhere.

## API Client

```swift
protocol APIClientProtocol {
    func send<Response: Decodable>(
        _ endpoint: Endpoint,
        body: (any Encodable)?
    ) async throws -> Response
}

struct APIClient: APIClientProtocol {
    let baseURL: URL
    let session: HTTPSession
    let encoder: JSONEncoder
    let decoder: JSONDecoder
    let authenticator: Authenticator?

    func send<Response: Decodable>(
        _ endpoint: Endpoint,
        body: (any Encodable)? = nil
    ) async throws -> Response {
        var request = try makeRequest(endpoint, body: body)

        if endpoint.requiresAuth {
            request = try await authenticator?.authorize(request) ?? request
        }

        let (data, response) = try await session.data(for: request)
        let validData = try validate(data, response)
        return try decoder.decode(Response.self, from: validData)
    }
}
```

Production generic `Encodable` handling needs careful implementation because existential `Encodable` encoding has nuances. Some teams use endpoint-specific body builders or `AnyEncodable`.

## Repository Uses API Client

```swift
struct NetworkProductRepository: ProductRepository {
    let apiClient: APIClientProtocol

    func products() async throws -> [Product] {
        let responses: [ProductResponse] = try await apiClient.send(
            Endpoint(path: "products", method: .get, requiresAuth: true),
            body: nil
        )

        return try responses.map(Product.init(response:))
    }
}
```

The repository maps DTOs to domain models.

## Middleware / Interceptors

Many teams implement request/response interceptors:

- Add headers
- Log requests
- Redact secrets
- Refresh tokens
- Add request IDs
- Retry transient failures

Keep interceptors ordered and testable.

```swift
protocol RequestInterceptor {
    func adapt(_ request: URLRequest) async throws -> URLRequest
}
```

## Logging

Log:

- Method
- Path
- Status code
- Duration
- Request ID

Do not log:

- Authorization tokens
- Passwords
- Payment data
- Sensitive payloads

## Cancellation

`URLSession` async APIs respect task cancellation.

```swift
let task = Task {
    try await apiClient.send(endpoint)
}

task.cancel()
```

Design ViewModels so leaving a screen cancels unnecessary work.

## Testing API Client

```swift
struct StubHTTPSession: HTTPSession {
    let result: Result<(Data, URLResponse), Error>

    func data(for request: URLRequest) async throws -> (Data, URLResponse) {
        try result.get()
    }
}
```

Test:

- Request method/path/headers
- Success decoding
- 401 mapping
- 422 validation mapping
- Network error mapping
- Redacted logging

## Senior iOS Engineer Perspective

Ask:

- What belongs in API client vs repository?
- How are endpoints modeled?
- How is auth injected?
- Is token refresh centralized?
- Is retry centralized but safe?
- Are errors normalized?
- Are logs redacted?
- Can tests assert request construction?

## Common Mistakes

- ViewModels build raw URLRequests.
- API client returns DTOs directly to SwiftUI.
- Token refresh spread across repositories.
- No consistent error mapping.
- Generic API client hides too much type complexity.
- Logs leak credentials.
- Retrying dangerous writes centrally without idempotency.

## Interview Notes

Junior:

API client is a reusable layer for making network requests.

Mid-level:

It builds requests, sends them with URLSession, validates responses, decodes data, and maps errors.

Senior:

I design API clients as transport boundaries, with request building, auth, token refresh, retry, logging, response validation, cancellation, and test seams separated from domain repositories.

## Practice

1. Build an `APIClient.send` method.
2. Add request interceptors.
3. Map API DTOs in a repository.
4. Test 401 and 422 error mapping.
