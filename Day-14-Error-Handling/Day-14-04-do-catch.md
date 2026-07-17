# Day 14: Do-Catch

## Core Idea

`do-catch` handles thrown errors.

```swift
do {
    try validate(email: email)
    print("Valid")
} catch LoginError.invalidEmail {
    print("Invalid email")
} catch {
    print("Other error: \(error)")
}
```

## Real iOS Use Cases

- Showing error messages
- Retrying network requests
- Logging failures

## Interview Levels

Junior: `do-catch` handles errors from throwing functions.

Senior: Catch specific errors when recovery differs. Avoid swallowing errors silently.

## Quick Notes

- `do` contains throwing code.
- `catch` handles errors.
- Catch specific cases first.

## Interview Depth

Junior answer: `do-catch` handles errors thrown inside the `do` block.

Mid-level answer: Specific catch blocks allow different recovery behavior for different failures.

Senior answer: Error handling should preserve meaning. Catch specific domain errors where recovery differs; log or map unknown errors appropriately. Avoid swallowing errors.

iOS use case:

```swift
do {
    try await authService.login(email: email, password: password)
} catch LoginError.invalidCredentials {
    errorMessage = "Invalid email or password"
} catch {
    errorMessage = "Please try again"
}
```

Common mistakes: empty catch, catching too broadly, showing technical errors directly to users.

Practice: catch enum case, catch unknown error, map error to UI message.
