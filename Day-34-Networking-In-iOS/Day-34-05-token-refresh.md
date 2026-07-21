# Day 34: Token Refresh

## What Token Refresh Is

Many APIs use:

- Short-lived access token
- Longer-lived refresh token

When access token expires, app uses refresh token to get a new access token.

Flow:

```text
Request -> 401 Unauthorized -> Refresh token -> Retry original request
```

## Auth Session Model

```swift
struct AuthSession: Codable, Equatable {
    let accessToken: String
    let refreshToken: String
    let expiresAt: Date

    var isAccessTokenExpired: Bool {
        Date() >= expiresAt
    }
}
```

Use a clock dependency in testable code instead of direct `Date()`.

## Token Refresh Client

```swift
protocol AuthClient {
    func refresh(refreshToken: String) async throws -> AuthSession
}
```

## Authenticator Actor

Use an actor to serialize refresh and avoid multiple simultaneous refresh calls.

```swift
actor Authenticator {
    private var session: AuthSession?
    private var refreshTask: Task<AuthSession, Error>?

    private let authClient: AuthClient
    private let tokenStore: TokenStoring

    init(authClient: AuthClient, tokenStore: TokenStoring) {
        self.authClient = authClient
        self.tokenStore = tokenStore
    }

    func validSession() async throws -> AuthSession {
        if let refreshTask {
            return try await refreshTask.value
        }

        let current = try await tokenStore.session()

        if !current.isAccessTokenExpired {
            return current
        }

        let task = Task {
            try await authClient.refresh(refreshToken: current.refreshToken)
        }

        refreshTask = task

        do {
            let newSession = try await task.value
            try await tokenStore.save(newSession)
            refreshTask = nil
            return newSession
        } catch {
            refreshTask = nil
            throw error
        }
    }
}
```

This prevents ten API calls from triggering ten refresh requests.

## Retrying Original Request

```swift
struct APIClient {
    let session: URLSession
    let authenticator: Authenticator

    func send(_ request: URLRequest) async throws -> Data {
        var request = request
        let authSession = try await authenticator.validSession()
        request.setValue("Bearer \(authSession.accessToken)", forHTTPHeaderField: "Authorization")

        let (data, response) = try await session.data(for: request)

        if (response as? HTTPURLResponse)?.statusCode == 401 {
            let refreshed = try await authenticator.validSession()
            var retry = request
            retry.setValue("Bearer \(refreshed.accessToken)", forHTTPHeaderField: "Authorization")
            return try await validate(session.data(for: retry))
        }

        return try validate((data, response))
    }
}
```

In production, force-refresh after 401 if `validSession` returns same stale token. The example is simplified.

## Refresh Failure

If refresh fails:

- Clear session.
- Navigate to login.
- Cancel pending authenticated requests.
- Avoid infinite retry loops.
- Preserve intended route if appropriate.

```swift
enum SessionEvent {
    case expired
}
```

Use a session model/router to handle global sign-out.

## Infinite Retry Guard

Never retry forever.

```swift
func send(_ request: URLRequest, retryCount: Int = 0) async throws -> Data {
    guard retryCount <= 1 else {
        throw APIError.unauthorized
    }

    // if 401:
    return try await send(request, retryCount: retryCount + 1)
}
```

## Proactive Refresh

Refresh before expiry:

```swift
var shouldRefreshSoon: Bool {
    expiresAt.timeIntervalSinceNow < 60
}
```

This reduces user-visible failures.

## Senior iOS Engineer Perspective

Ask:

- Who owns session state?
- Is refresh serialized?
- What happens when many requests get 401?
- Is retry bounded?
- Are tokens stored securely?
- What happens if refresh token is revoked?
- How does UI learn session expired?

## Common Mistakes

- Multiple simultaneous refresh calls.
- Infinite retry loops.
- Refresh token in UserDefaults.
- Retrying non-idempotent requests blindly.
- No global session-expired handling.
- ViewModels refreshing tokens themselves.
- Race conditions overwriting newer tokens.

## Interview Notes

Junior:

Token refresh gets a new access token when the old one expires.

Mid-level:

On 401, refresh token, save new session, and retry the original request once.

Senior:

I serialize token refresh with an actor, prevent refresh storms, securely store tokens, bound retries, handle revoked sessions, and coordinate global logout/navigation.

## Practice

1. Build an `Authenticator` actor.
2. Retry a request after 401 once.
3. Handle refresh failure by clearing session.
4. Explain refresh storm prevention.
