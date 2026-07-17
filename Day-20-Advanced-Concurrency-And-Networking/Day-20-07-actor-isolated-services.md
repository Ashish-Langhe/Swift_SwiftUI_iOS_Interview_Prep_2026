# Day 20: Actor-Isolated Services

## What Actor-Isolated Services Are

An actor-isolated service protects shared mutable service state.

```swift
actor AuthService {
    private var token: String?

    func authenticatedRequest(_ request: URLRequest) async throws -> URLRequest {
        var request = request
        request.setValue(try await validToken(), forHTTPHeaderField: "Authorization")
        return request
    }
}
```

## Why Services Need Isolation

Networking services often share:

- Auth tokens
- Refresh tasks
- Caches
- Rate limit state
- Request deduplication maps
- Upload progress
- WebSocket connection state

Actors protect that state.

## Token Refresh Coalescing

```swift
actor TokenProvider {
    private var token: String?
    private var refreshTask: Task<String, Error>?

    func validToken() async throws -> String {
        if let token { return token }

        if let refreshTask {
            return try await refreshTask.value
        }

        let task = Task {
            try await requestTokenFromServer()
        }

        refreshTask = task

        do {
            let token = try await task.value
            self.token = token
            refreshTask = nil
            return token
        } catch {
            refreshTask = nil
            throw error
        }
    }
}
```

This prevents ten simultaneous requests from causing ten token refresh calls.

## Image Loader Deduplication

```swift
actor ImageLoader {
    private var cache: [URL: UIImage] = [:]
    private var inFlight: [URL: Task<UIImage, Error>] = [:]

    func image(from url: URL) async throws -> UIImage {
        if let image = cache[url] { return image }
        if let task = inFlight[url] { return try await task.value }

        let task = Task { try await downloadImage(url) }
        inFlight[url] = task

        do {
            let image = try await task.value
            cache[url] = image
            inFlight[url] = nil
            return image
        } catch {
            inFlight[url] = nil
            throw error
        }
    }
}
```

## Reentrancy Warning

Actor methods can suspend.

State may change while awaiting.

Pattern:

- Check state.
- Store in-flight task before await.
- Await.
- Clean up state.

This is why examples store `refreshTask` before awaiting it.

## Public Service API

```swift
protocol UserServicing: Sendable {
    func user(id: User.ID) async throws -> User
}
```

Actors can conform to async service protocols.

```swift
actor UserService: UserServicing {
    func user(id: User.ID) async throws -> User {
        try await client.send(UserEndpoint(id: id))
    }
}
```

## Common Mistakes

- Using a mutable singleton service without isolation
- Starting duplicate refresh requests
- Forgetting actor reentrancy
- Returning mutable internal state
- Doing UI updates inside service actors
- Creating actor bottlenecks for stateless work

## Modern Swift 6.x Notes

Actor-isolated services are a natural Swift 6 solution for strict concurrency warnings. Swift 6.2's approachable concurrency makes UI isolation easier, but shared service state still belongs behind actors or safe synchronization.

## Interview Answer

I use actors for services with shared mutable state, such as token refresh, caches, rate limits, and request deduplication. I design actor APIs to return values, avoid exposing mutable internals, and handle reentrancy by storing in-flight operations before suspension.

## Practice

1. Build a token provider actor.
2. Add request deduplication to an image loader actor.
3. Explain actor reentrancy in refresh logic.
4. Decide whether a stateless API client needs to be an actor.
