# Day 14: Error Protocol

## Core Idea

Swift errors conform to the `Error` protocol.

```swift
enum LoginError: Error {
    case invalidEmail
    case wrongPassword
}
```

Enums are commonly used because they model finite failure cases.

## Real iOS Use Cases

- Network errors
- Validation errors
- Persistence errors
- Auth errors

## Interview Levels

Junior: `Error` marks a type as throwable.

Senior: Error enums should model recoverable failure in a way callers can handle, log, and present.

## Quick Notes

- `Error` is marker protocol.
- Enums are common.
- Model meaningful failures.

## Interview Depth

Junior answer: `Error` is the protocol Swift error types conform to.

Mid-level answer: Error enums are common because they model known failure cases clearly.

Senior answer: Error types should match recovery behavior. Add associated values when callers need context, but avoid exposing low-level implementation details unnecessarily.

iOS use case:

```swift
enum ProfileError: Error {
    case missingUserId
    case networkUnavailable
    case decodingFailed
}
```

Common mistakes: using generic `Error` everywhere, exposing raw server messages directly, not modeling recoverability.

Practice: create validation error, add associated value, decide user-facing message.
