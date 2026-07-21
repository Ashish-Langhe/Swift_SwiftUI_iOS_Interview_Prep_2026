# Day 34: Authentication Headers

## What Authentication Headers Are

Most authenticated APIs require credentials in HTTP headers.

Common header:

```text
Authorization: Bearer <access-token>
```

Other common headers:

```text
Content-Type: application/json
Accept: application/json
X-Request-ID: <uuid>
Idempotency-Key: <uuid>
```

## Bearer Token Request

```swift
struct AuthenticatedRequestBuilder {
    let baseURL: URL
    let tokenStore: TokenStore

    func request(path: String) async throws -> URLRequest {
        var request = URLRequest(url: baseURL.appending(path: path))
        request.httpMethod = "GET"
        request.setValue("application/json", forHTTPHeaderField: "Accept")

        let token = try await tokenStore.accessToken()
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

        return request
    }
}
```

## Token Storage

Access tokens and refresh tokens are sensitive.

Use Keychain for persisted tokens.

```swift
protocol TokenStoring {
    func accessToken() async throws -> String
    func refreshToken() async throws -> String
    func save(_ session: AuthSession) async throws
    func clear() async throws
}
```

Do not store tokens in UserDefaults.

## Header Injection Layer

Keep header logic centralized.

```swift
protocol RequestAuthorizing {
    func authorize(_ request: URLRequest) async throws -> URLRequest
}

struct BearerTokenAuthorizer: RequestAuthorizing {
    let tokenStore: TokenStoring

    func authorize(_ request: URLRequest) async throws -> URLRequest {
        var request = request
        let token = try await tokenStore.accessToken()
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        return request
    }
}
```

API client:

```swift
var request = try requestBuilder.request(endpoint)
request = try await authorizer.authorize(request)
```

## Authentication Challenges

Some servers issue authentication challenges. `URLSessionDelegate` can respond to them.

```swift
final class SessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        completionHandler(.performDefaultHandling, nil)
    }
}
```

Use delegate-based handling for server trust, client certificates, or advanced auth challenge cases.

## Logging Safety

Never log tokens.

Bad:

```swift
print(request.allHTTPHeaderFields)
```

Better:

```swift
func redactedHeaders(_ headers: [String: String]) -> [String: String] {
    headers.mapValues { _ in "<redacted>" }
}
```

## Request ID

Add a request ID for tracing.

```swift
request.setValue(UUID().uuidString, forHTTPHeaderField: "X-Request-ID")
```

This helps backend/mobile teams debug production incidents.

## Idempotency Header

For write operations that may be retried:

```swift
request.setValue(UUID().uuidString, forHTTPHeaderField: "Idempotency-Key")
```

Use for:

- Payments
- Order creation
- Booking
- Bank transfer

Only when backend supports it.

## Senior iOS Engineer Perspective

Ask:

- Where are tokens stored?
- Who injects auth headers?
- Are tokens redacted in logs?
- Are request IDs present?
- Is idempotency used for dangerous writes?
- Are auth challenges handled?
- What happens on 401?

## Common Mistakes

- Tokens in UserDefaults.
- Header logic duplicated across endpoints.
- Logging Authorization headers.
- No request correlation ID.
- Retrying payments without idempotency.
- Treating 401 like normal failure.
- Vendor auth logic leaked into ViewModels.

## Interview Notes

Junior:

Authentication headers send credentials with API requests.

Mid-level:

Use Bearer tokens, store secrets in Keychain, and centralize header injection.

Senior:

I design auth headers around secure token storage, centralized authorization, redacted logging, request tracing, idempotency, auth challenge handling, and 401/session behavior.

## Practice

1. Add Bearer token authorization to a request.
2. Redact headers for logging.
3. Add request ID and idempotency key.
4. Explain why tokens should be in Keychain.
