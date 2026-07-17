# Day 14: Try? And Try!

## Core Idea

`try?` converts a throwing result into optional.

```swift
let user = try? loadUser()
```

`try!` crashes if an error is thrown.

```swift
let config = try! loadKnownValidConfig()
```

## Interview Levels

Junior: `try?` returns nil on error. `try!` crashes on error.

Senior: `try?` is useful when failure details do not matter. `try!` should be rare and only used when failure means programmer error.

## Quick Notes

- `try?` loses error details.
- `try!` can crash.
- Prefer `do-catch` when error matters.

## Interview Depth

Junior answer: `try?` returns nil if an error happens. `try!` crashes if an error happens.

Mid-level answer: Use `try?` when the exact error does not matter, such as optional cache loading. Avoid `try!` except for guaranteed invariants.

Senior answer: `try?` is lossy error handling. It is fine for fallback flows but bad for diagnostics, analytics, and user-facing failures. `try!` is an assertion, not normal error handling.

iOS use case:

```swift
let cachedUser = try? cache.loadUser()
```

Common mistakes: losing important error context, using `try!` with network or file input, silently ignoring failures.

Practice: convert throw to optional, replace try! with do-catch, explain when try? is acceptable.
