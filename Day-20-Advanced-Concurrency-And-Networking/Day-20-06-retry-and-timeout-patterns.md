# Day 20: Retry And Timeout Patterns

## Why Retries And Timeouts Matter

Network requests fail for normal reasons:

- Temporary connectivity loss
- Server overload
- Rate limiting
- Captive portals
- Slow responses

Good apps retry selectively and time out clearly.

## Basic Retry

```swift
func retrying<T>(
    attempts: Int,
    operation: () async throws -> T
) async throws -> T {
    var lastError: Error?

    for _ in 0..<attempts {
        do {
            return try await operation()
        } catch is CancellationError {
            throw CancellationError()
        } catch {
            lastError = error
        }
    }

    throw lastError ?? NetworkError.unknown
}
```

## Exponential Backoff

```swift
func retryingWithBackoff<T>(
    attempts: Int,
    operation: () async throws -> T
) async throws -> T {
    var delay: Duration = .milliseconds(300)

    for attempt in 1...attempts {
        do {
            return try await operation()
        } catch is CancellationError {
            throw CancellationError()
        } catch {
            if attempt == attempts { throw error }
            try await Task.sleep(for: delay)
            delay *= 2
        }
    }

    fatalError("Unreachable")
}
```

## Retry Only Safe Operations

Usually safe:

- GET requests
- Idempotent PUT
- Requests with idempotency keys

Be careful:

- Payments
- Orders
- Mutating POST requests
- File uploads

For payments, use server-supported idempotency keys.

## Timeout Pattern

Use a throwing task group race:

```swift
func withTimeout<T>(
    _ duration: Duration,
    operation: @escaping () async throws -> T
) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask {
            try await operation()
        }

        group.addTask {
            try await Task.sleep(for: duration)
            throw TimeoutError()
        }

        let result = try await group.next()!
        group.cancelAll()
        return result
    }
}
```

## Real iOS Example

```swift
let profile = try await withTimeout(.seconds(8)) {
    try await retryingWithBackoff(attempts: 3) {
        try await api.profile()
    }
}
```

## Cancellation

Retries must respect cancellation.

Bad:

```swift
catch {
    try? await Task.sleep(for: .seconds(1))
}
```

Better:

```swift
catch is CancellationError {
    throw CancellationError()
}
```

## Common Mistakes

- Retrying cancellation
- Retrying non-idempotent operations blindly
- No max attempts
- No backoff
- Showing timeout as generic error
- Forgetting to cancel losing timeout task
- Retrying 401 auth errors without refresh strategy

## Modern Swift 6.x Notes

Retry helpers should use `@Sendable` closures if they cross task boundaries and should be cancellation-aware. Public retry APIs need careful sendability and error behavior design.

## Interview Answer

I retry only transient and safe failures, use exponential backoff, preserve cancellation, and use timeout races with task groups. For mutating operations like payments, I require idempotency keys rather than blind retries.

## Practice

1. Write a retry helper that does not retry cancellation.
2. Add exponential backoff.
3. Implement `withTimeout`.
4. Explain why payment retries need idempotency.
