# Day 10: State Modeling In iOS

## Core Idea

Enums are excellent for screen state.

```swift
enum ProfileState {
    case loading
    case loaded(User)
    case empty
    case failed(String)
}
```

This avoids invalid combinations like loading and error at the same time.

## Real iOS Use Cases

- Network loading state
- Navigation routes
- Form validation state
- Authentication state

## Modern Swift 6.x Notes

Enum-based state pairs well with SwiftUI and concurrency because UI can render immutable snapshots of finite state.

## Interview Levels

Junior:

Enums can represent different app states.

Senior:

Enums prevent invalid states and make rendering exhaustive through switch. This is one of the strongest Swift architecture patterns.

## Quick Notes

- Avoid optional-state explosion
- Make impossible states impossible
- Switch rendering is exhaustive

## Interview Depth

Junior answer: Enums can represent different screen states.

Mid-level answer: A screen often has loading, loaded, empty, and failed states. An enum makes those states explicit.

Senior answer: Enum state modeling prevents invalid combinations. It also makes UI rendering exhaustive and easier to test. This is much better than separate booleans and optionals that can conflict.

iOS use case:

```swift
enum SearchState {
    case idle
    case searching
    case results([SearchResult])
    case noResults
    case failed(String)
}
```

Common mistakes:

- Using `isLoading`, `data?`, and `error?` together.
- Adding vague `.unknown` cases.
- Ignoring empty state.

Practice:

1. Refactor optional-heavy state to enum.
2. Switch render every state.
3. Explain impossible states.
