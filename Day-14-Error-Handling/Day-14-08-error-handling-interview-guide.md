# Day 14: Error Handling Interview Guide

## One-Minute Interview Answer

Swift error handling uses types conforming to `Error`, throwing functions marked with `throws`, and calls marked with `try`. `do-catch` handles errors, `try?` converts errors to nil, and `try!` crashes if an error occurs. `Result` stores success or failure as a value. Modern Swift also supports typed throws syntax, and async code uses `async throws` with `try await`.

## Modern Swift 6.x Notes

Typed throws are documented as `throws(MyError)`. Swift 6.2 improved async debugging and approachable concurrency, which matters when handling errors in async tasks. Swift Testing also gained exit testing and richer attachments in Swift 6.2/6.3, useful for testing failure paths.

## Common Traps

- Using `try?` and losing important error details.
- Using `try!` with dynamic data.
- Catching all errors and doing nothing.
- Showing raw technical errors to users.
- Ignoring cancellation in async work.

## Topic-By-Topic Deep Dive

### `Error` Protocol

Swift errors are values that conform to `Error`.

```swift
enum PaymentError: Error {
    case cardDeclined
    case insufficientFunds
    case networkUnavailable
}
```

Senior answer:

Use error cases that match recovery decisions. If the UI handles two failures the same way, they may not need separate public cases. If logging/debugging needs more detail, use associated values.

### `throw`, `throws`, `try`

```swift
func charge(amount: Decimal) throws {
    guard amount > 0 else {
        throw PaymentError.insufficientFunds
    }
}
```

`throws` is part of the function contract. `try` makes failure visible at the call site.

### `try?` And `try!`

`try?` is good when only success/failure matters:

```swift
let image = try? loadCachedImage()
```

`try!` is only appropriate when failure means programmer error or impossible invariant failure:

```swift
let regex = try! Regex("[0-9]+")
```

### `do-catch`

```swift
do {
    try await viewModel.login()
} catch AuthError.invalidCredentials {
    message = "Invalid email or password"
} catch {
    message = "Something went wrong"
}
```

Senior rule:

Catch specifically when recovery differs. Avoid one giant catch that loses meaning.

### Typed Error Modeling

Typed throws:

```swift
func parseToken(_ raw: String) throws(AuthError) -> Token {
    guard !raw.isEmpty else {
        throw .missingToken
    }
    return Token(rawValue: raw)
}
```

Senior answer:

Typed throws are useful when the failure domain is stable and meaningful. Untyped `throws` remains practical when composing many possible error sources.

### `Result`

Use `Result` when success/failure must be stored or passed as a value.

```swift
let result: Result<User, AuthError>
```

With async/await, direct `async throws` APIs are often cleaner than callback `Result`.

### Async Error Handling

```swift
func load() async {
    do {
        state = .loading
        let user = try await service.fetchUser()
        state = .loaded(user)
    } catch is CancellationError {
        state = .idle
    } catch {
        state = .failed("Could not load user")
    }
}
```

Senior answer:

Cancellation is not the same as failure. Treat it separately when user intent or task lifecycle matters.

## Production Decision Table

| Situation | Best Tool |
| --- | --- |
| Recoverable failure | `throws` |
| Failure as stored value | `Result` |
| Ignore details safely | `try?` |
| Programmer invariant | rare `try!` |
| Stable failure domain | typed throws / error enum |
| Async operation failure | `async throws` |

## Final Revision

- `Error` marks error types.
- `throw` emits.
- `throws` declares.
- `try` calls.
- `do-catch` handles.
- `Result` stores outcome.
- `async throws` combines concurrency and failure.
