# Day 11: Access Control On Properties

## Core Idea

Access control controls where a property can be read or written.

```swift
struct Counter {
    private(set) var value = 0

    mutating func increment() {
        value += 1
    }
}
```

`private(set)` means public/internal read, private write within the allowed scope.

## Common Levels

- `private`
- `fileprivate`
- `internal`
- `package`
- `public`
- `open`

## Real iOS Use Cases

```swift
final class LoginViewModel {
    private(set) var isLoading = false
}
```

External code can read `isLoading`, but only the view model changes it.

## Modern Swift 6.x Notes

`package` access is useful in modular Swift packages. Access control helps maintain invariants across modules and teams.

## Interview Levels

Junior: Access control limits where properties can be used.

Senior: Access control protects invariants. I expose the smallest surface possible and use `private(set)` when callers should observe state but not mutate it.

## Quick Notes

- Use least needed access.
- `private(set)` is common.
- Access control protects invariants.
- Public mutable state is risky.

## Interview Depth

Junior answer: Access control decides where properties can be read or changed.

Mid-level answer: `private(set)` lets external code read a property while only the owning type can modify it.

Senior answer: Access control is not only hiding code; it protects invariants. A type should expose the smallest mutation surface that still supports its use cases.

iOS use case:

```swift
@MainActor
final class ProfileViewModel {
    private(set) var state: ProfileState = .loading
}
```

Common mistakes: public mutable state, using `private` so aggressively tests become awkward, exposing implementation details.

Practice: refactor public var to private(set), explain invariant, list access levels.
