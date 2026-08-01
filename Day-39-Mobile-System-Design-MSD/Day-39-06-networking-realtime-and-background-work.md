# Day 39: Networking, Realtime, And Background Work

Networking in MSD is not only about calling REST APIs. You need to design request flow, authentication, retries, cancellation, pagination, upload/download, realtime updates, background limitations, and graceful degradation.

## Mobile Networking Stack

```text
View/ViewModel
  -> UseCase
  -> Repository
  -> API Client
  -> Auth Interceptor
  -> URLSession
```

API client responsibilities:

- Build requests.
- Add headers.
- Decode responses.
- Map status codes.
- Apply timeouts.
- Support cancellation.
- Surface typed errors.

Auth layer responsibilities:

- Attach access token.
- Refresh token safely.
- Prevent refresh stampede.
- Retry original request once after refresh.
- Logout on unrecoverable auth failure.

## REST Request Example

```swift
struct Endpoint<Response: Decodable> {
    let path: String
    let method: HTTPMethod
    let query: [URLQueryItem]
    let body: Data?
}

protocol APIClient {
    func send<Response: Decodable>(_ endpoint: Endpoint<Response>) async throws -> Response
}
```

Senior note:

```text
The feature should not know URLSession details. It should depend on repositories or domain clients.
```

## Retry Rules

Retry:

- Network timeout.
- Temporary server errors like 502, 503, 504.
- Idempotent GET requests.
- Writes with idempotency keys.

Do not blindly retry:

- Payment capture.
- Money transfer.
- Login with invalid credentials.
- 400 validation errors.

Backoff example:

```swift
func retrying<T>(
    maxAttempts: Int = 3,
    operation: @escaping () async throws -> T
) async throws -> T {
    var delay: UInt64 = 300_000_000

    for attempt in 1...maxAttempts {
        do {
            return try await operation()
        } catch where attempt < maxAttempts {
            try await Task.sleep(nanoseconds: delay)
            delay *= 2
        }
    }

    return try await operation()
}
```

In production, add jitter and retry only eligible errors.

## Realtime Options

### Push Notifications

Use when:

- Event frequency is low.
- App may be backgrounded.
- User needs notification.

Limitations:

- Not guaranteed delivery.
- Payload size is limited.
- Timing is controlled by OS.

### WebSocket

Use when:

- Active session needs low-latency updates.
- Chat, trading, ride tracking, collaborative updates.

Limitations:

- More battery usage.
- Needs reconnect logic.
- Must handle app lifecycle transitions.

### Polling

Use when:

- Realtime is not critical.
- Backend does not support streaming.
- Simplicity matters.

Limitations:

- Wastes battery/network if too frequent.
- Adds latency.

Senior decision:

```text
For ride tracking, I would use REST for initial trip snapshot, WebSocket while the tracking screen is active, and push notifications for important status changes when the app is backgrounded.
```

## Background Work

iOS background execution is limited.

Use:

- Background URLSession for large uploads/downloads.
- BGTaskScheduler for refresh or processing when the system allows.
- Silent push only as a hint, not a guarantee.
- Local persistence for resumable work.

Do not promise:

- Continuous background execution.
- Guaranteed background sync at exact time.
- Always-live WebSocket in background.

## Upload Design

For media upload:

1. Create upload intent.
2. Upload binary to storage/CDN endpoint.
3. Confirm metadata with backend.
4. Track progress locally.
5. Retry resumable chunks when supported.

```text
iOS -> CreateUpload API -> Upload URL
iOS -> Storage/CDN multipart upload
iOS -> CompleteUpload API
Backend -> Media processing -> Notify client
```

## Download Design

For video or documents:

- Use background download when needed.
- Store metadata separately from file.
- Validate file integrity.
- Respect storage limits.
- Support user deletion.
- Consider DRM for protected media.

## Cancellation

Cancellation is central to modern Swift networking.

Examples:

- Cancel search request when query changes.
- Cancel feed prefetch when cell disappears.
- Cancel image request when reused cell changes.
- Cancel checkout polling after order completes.

```swift
final class SearchViewModel: ObservableObject {
    private var searchTask: Task<Void, Never>?

    @MainActor
    func search(_ query: String) {
        searchTask?.cancel()
        searchTask = Task {
            try? await Task.sleep(nanoseconds: 300_000_000)
            guard !Task.isCancelled else { return }
            // fetch results
        }
    }
}
```

## Interview Notes

- Choose realtime tech based on frequency, latency, battery, and app state.
- Push is not a database.
- Retry policy belongs below feature UI.
- Use idempotency for safe writes.
- Mention cancellation for search, feed, and image loading.
- Be honest about iOS background limits.
