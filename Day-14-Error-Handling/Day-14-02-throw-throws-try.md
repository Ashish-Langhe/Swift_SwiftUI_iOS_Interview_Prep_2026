# Day 14: Throw, Throws, And Try

## Core Idea

`throw` emits an error. `throws` marks a function that can throw. `try` calls a throwing function.

```swift
func validate(email: String) throws {
    guard email.contains("@") else {
        throw LoginError.invalidEmail
    }
}

try validate(email: "user@example.com")
```

## Interview Levels

Junior: Use `throws` for functions that can fail and `try` when calling them.

Senior: Throwing APIs are best when failure is exceptional but recoverable and caller should decide handling.

## Quick Notes

- `throw` sends error.
- `throws` marks function.
- `try` calls throwing code.

## Interview Depth

Junior answer: `throws` means a function can fail, `throw` sends an error, and `try` calls a throwing function.

Mid-level answer: Throwing functions make failure explicit at call sites. Callers must handle or propagate errors.

Senior answer: `throws` is API design. Use it for recoverable failure that the caller should decide how to handle. Do not use it for ordinary branching.

iOS use case:

```swift
func validatedEmail(_ text: String) throws -> String {
    guard text.contains("@") else { throw LoginError.invalidEmail }
    return text
}
```

Common mistakes: throwing for non-error control flow, using try! casually, not documenting failure cases.

Practice: write throwing validator, call with try, propagate error.
