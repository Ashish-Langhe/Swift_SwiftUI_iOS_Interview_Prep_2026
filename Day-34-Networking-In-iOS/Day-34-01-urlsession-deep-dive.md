# Day 34: URLSession Deep Dive

## What URLSession Is

`URLSession` is Foundation's main networking API. It coordinates related network transfer tasks.

Use it for:

- REST API calls
- Downloads
- Uploads
- Background transfers
- Streaming bytes
- Authentication challenges
- WebSocket tasks

Apple describes URLSession as coordinating a group of related network data transfer tasks.

## URLSession Task Types

Main task types:

```text
Data task:
Small request/response data, common for REST APIs.

Upload task:
Uploads data or files, supports background upload behavior.

Download task:
Downloads to a temporary file URL, supports background downloads.

WebSocket task:
Bidirectional WebSocket messaging.
```

For most app API clients, `data(for:)` is the everyday tool.

## Basic Async Request

```swift
struct ProductAPI {
    let session: URLSession
    let baseURL: URL

    func products() async throws -> [ProductResponse] {
        let url = baseURL.appending(path: "products")
        let (data, response) = try await session.data(from: url)

        guard let httpResponse = response as? HTTPURLResponse,
              200..<300 ~= httpResponse.statusCode else {
            throw APIError.invalidResponse
        }

        return try JSONDecoder().decode([ProductResponse].self, from: data)
    }
}
```

## URLRequest

Use `URLRequest` when you need:

- HTTP method
- Headers
- Body
- Cache policy
- Timeout

```swift
var request = URLRequest(url: baseURL.appending(path: "orders"))
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
request.httpBody = try JSONEncoder().encode(CreateOrderRequest(cartID: cartID))

let (data, response) = try await session.data(for: request)
```

`URLRequest` represents request metadata. `URLSession` sends it.

## URLSessionConfiguration

```swift
let configuration = URLSessionConfiguration.default
configuration.timeoutIntervalForRequest = 30
configuration.timeoutIntervalForResource = 60
configuration.waitsForConnectivity = true
configuration.allowsExpensiveNetworkAccess = true
configuration.allowsConstrainedNetworkAccess = true

let session = URLSession(configuration: configuration)
```

Important options:

- `timeoutIntervalForRequest`
- `timeoutIntervalForResource`
- `waitsForConnectivity`
- `allowsCellularAccess`
- `allowsExpensiveNetworkAccess`
- `allowsConstrainedNetworkAccess`
- `httpAdditionalHeaders`
- `urlCache`

## waitsForConnectivity

`waitsForConnectivity` controls whether tasks wait for connectivity or fail immediately while establishing a connection.

```swift
configuration.waitsForConnectivity = true
```

Use it when:

- Temporary connectivity loss is expected.
- User action can wait.
- You want URLSession to wait before failing initial connection.

Do not use it as your whole offline strategy. It does not solve server errors, dropped connections after establishment, or stale data needs.

## Delegates

Delegates support advanced behavior:

- Authentication challenges
- Redirect handling
- Progress events
- Background transfer events
- Custom caching decisions

```swift
final class NetworkSessionDelegate: NSObject, URLSessionTaskDelegate {
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didCompleteWithError error: Error?
    ) {
        // observe completion
    }
}
```

Important memory note: URLSession strongly retains its delegate until invalidated. Invalidate sessions when they are no longer needed.

```swift
session.invalidateAndCancel()
```

## URLSession Shared vs Custom

`URLSession.shared` is fine for quick/simple calls.

Prefer custom sessions when you need:

- Custom timeouts
- Auth challenge delegate
- Background configuration
- Cache policy
- Connectivity behavior
- Testing boundary
- Request logging

## Testing URLSession Code

Avoid hard-coding `URLSession.shared` deep in APIs.

```swift
protocol HTTPSession {
    func data(for request: URLRequest) async throws -> (Data, URLResponse)
}

extension URLSession: HTTPSession {}
```

API client:

```swift
struct APIClient {
    let session: HTTPSession
}
```

Test:

```swift
struct StubSession: HTTPSession {
    let data: Data
    let response: URLResponse

    func data(for request: URLRequest) async throws -> (Data, URLResponse) {
        (data, response)
    }
}
```

## Senior iOS Engineer Perspective

Senior URLSession design asks:

- Is this request short, upload, download, background, or stream?
- Do I need custom configuration?
- Is connectivity behavior deliberate?
- Are delegates retained/invalidation handled?
- Are responses and errors normalized?
- Can this be tested without real network?
- Does this API client leak transport details?

## Common Mistakes

- Using `URLSession.shared` everywhere.
- Not checking HTTP status codes.
- Decoding before validating response.
- No timeout strategy.
- No cancellation handling.
- No test seam.
- Delegate session leaked by not invalidating.
- Treating `waitsForConnectivity` as reachability.

## Interview Notes

Junior:

`URLSession` performs network requests, uploads, and downloads.

Mid-level:

Use `URLRequest` for methods/headers/body, validate `HTTPURLResponse`, and decode data.

Senior:

I design URLSession usage around task type, configuration, delegates, cancellation, connectivity, testability, and API-client boundaries.

## Practice

1. Build a custom `URLSessionConfiguration`.
2. Implement a simple `HTTPSession` protocol.
3. Validate HTTP status before decoding.
4. Explain when to use data, upload, download, or background sessions.
