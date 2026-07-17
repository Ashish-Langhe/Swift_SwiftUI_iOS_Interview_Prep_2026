# Day 14: Error Handling In Async Code

## Core Idea

Async throwing functions use `async throws`.

```swift
func fetchUser() async throws -> User {
    try await apiClient.user()
}
```

Call with:

```swift
do {
    let user = try await fetchUser()
    print(user)
} catch {
    print(error)
}
```

## Real iOS Use Cases

- URLSession
- Database operations
- Auth flows
- File loading

## Modern Swift 6.x Notes

Swift 6.2 improved approachable concurrency, async debugging, and task context visibility. Error handling in async code should also respect cancellation.

```swift
try Task.checkCancellation()
```

## Interview Levels

Junior: Use `try await` for async throwing calls.

Senior: Async error handling should distinguish cancellation, retryable failures, user-facing failures, and programmer errors.

## Quick Notes

- `async throws`.
- Call with `try await`.
- Handle cancellation.
- Map errors for UI.

## Interview Depth

Junior answer: Use `try await` when calling an async function that can throw.

Mid-level answer: Async throwing code is handled with `do-catch`, just like synchronous throwing code, but calls include `await`.

Senior answer: Async error handling should distinguish user cancellation, network failure, decoding failure, and business failure. Cancellation is often not an error to show to the user.

iOS use case:

```swift
func loadProfile() async {
    do {
        state = .loading
        state = .loaded(try await service.profile())
    } catch is CancellationError {
        state = .idle
    } catch {
        state = .failed("Could not load profile")
    }
}
```

Common mistakes: ignoring cancellation, updating UI off main actor, retrying non-retryable errors.

Practice: write async throws loader, catch cancellation, map error to state enum.
