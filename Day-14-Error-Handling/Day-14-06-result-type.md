# Day 14: Result Type

## Core Idea

`Result` stores success or failure.

```swift
let result: Result<User, LoginError> = .success(user)
```

Switch over it:

```swift
switch result {
case .success(let user):
    print(user)
case .failure(let error):
    print(error)
}
```

## Real iOS Use Cases

- Completion handlers
- Storing operation outcomes
- Bridging callback APIs

## Interview Levels

Junior: `Result` has success and failure.

Senior: `Result` is useful when the outcome must be passed around as a value. For direct control flow, `async throws` or throwing functions are often cleaner.

## Quick Notes

- `Result<Success, Failure>`.
- Failure conforms to `Error`.
- Useful for callbacks.

## Interview Depth

Junior answer: `Result` stores either success or failure.

Mid-level answer: It is common in completion handlers and when an operation outcome must be stored or passed around.

Senior answer: With async/await, many APIs are cleaner as `async throws`. Use `Result` when the result itself is data, especially in callbacks, streams, queues, or state machines.

iOS use case:

```swift
typealias Completion = (Result<User, LoginError>) -> Void
```

Common mistakes: nesting `Result` unnecessarily, using `Result` and throwing together awkwardly, ignoring failure associated data.

Practice: switch on Result, convert throwing call to Result, compare async throws.
