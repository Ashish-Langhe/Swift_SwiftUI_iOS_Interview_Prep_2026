# Day 34: Retry And Backoff

## Why Retry Matters

Networks fail temporarily:

- Connection drops
- DNS hiccups
- Server overloaded
- Timeout
- Rate limiting
- Cellular transition

Retry can improve reliability, but unsafe retry can duplicate actions.

## Retryable vs Non-Retryable

Usually retryable:

- Timeout
- Network connection lost
- 503 Service unavailable
- 429 Rate limited
- Some 5xx

Usually not retryable:

- 400 Bad request
- 401 Unauthorized without refresh flow
- 403 Forbidden
- 404 Not found
- 422 Validation error

Dangerous:

- Payment
- Order creation
- Booking
- Bank transfer

Retry dangerous writes only with idempotency support.

## Retry Policy

```swift
struct RetryPolicy {
    let maxAttempts: Int
    let baseDelay: Duration
    let maxDelay: Duration

    func delay(for attempt: Int) -> Duration {
        let multiplier = 1 << max(0, attempt - 1)
        let seconds = min(
            Double(baseDelay.components.seconds) * Double(multiplier),
            Double(maxDelay.components.seconds)
        )
        return .seconds(seconds)
    }

    func shouldRetry(error: Error, attempt: Int) -> Bool {
        attempt < maxAttempts && error.isTransientNetworkError
    }
}
```

## Exponential Backoff

```text
Attempt 1: immediate
Attempt 2: wait 1s
Attempt 3: wait 2s
Attempt 4: wait 4s
```

Backoff avoids hammering a struggling server.

## Jitter

Add randomness so many clients do not retry at the same time.

```swift
func jitteredDelay(_ delay: Duration) -> Duration {
    let milliseconds = Int.random(in: 0...500)
    return delay + .milliseconds(milliseconds)
}
```

## Retry Helper

```swift
func retrying<T>(
    policy: RetryPolicy,
    operation: () async throws -> T
) async throws -> T {
    var attempt = 0

    while true {
        do {
            return try await operation()
        } catch {
            attempt += 1

            guard policy.shouldRetry(error: error, attempt: attempt) else {
                throw error
            }

            try await Task.sleep(for: policy.delay(for: attempt))
        }
    }
}
```

## Respect Cancellation

```swift
try Task.checkCancellation()
try await Task.sleep(for: delay)
try Task.checkCancellation()
```

Do not keep retrying after user leaves the screen.

## Retry-After Header

For `429` or `503`, backend may send:

```text
Retry-After: 30
```

Respect it when present.

```swift
if let retryAfter = http.value(forHTTPHeaderField: "Retry-After"),
   let seconds = Double(retryAfter) {
    try await Task.sleep(for: .seconds(seconds))
}
```

## User Experience

Retry silently for quick transient read failures.

Show UI for:

- Long retry
- Repeated failure
- User-triggered retry
- Offline mode
- Payment/write failure

## Senior iOS Engineer Perspective

Ask:

- Is operation idempotent?
- Which errors are retryable?
- Is backoff bounded?
- Is jitter needed?
- Is cancellation respected?
- Does server provide Retry-After?
- What does the user see?

## Common Mistakes

- Retrying all errors.
- Retrying payments without idempotency.
- Infinite retry loops.
- No cancellation.
- No backoff/jitter.
- Ignoring Retry-After.
- Showing repeated alert spam.

## Interview Notes

Junior:

Retry tries a failed request again.

Mid-level:

Use exponential backoff and retry only transient errors.

Senior:

I design retry around idempotency, bounded attempts, backoff, jitter, Retry-After, cancellation, and user-visible recovery.

## Practice

1. Build a retry helper.
2. Add backoff and jitter.
3. Decide if `POST /orders` can be retried.
4. Handle `Retry-After`.
