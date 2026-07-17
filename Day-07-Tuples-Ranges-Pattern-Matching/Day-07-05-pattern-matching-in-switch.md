# Day 7: Pattern Matching In Switch

## Core Idea

Swift `switch` matches patterns, not just simple values.

```swift
let point = (0, 10)

switch point {
case (0, 0):
    print("Origin")
case (0, _):
    print("Y-axis")
case (_, 0):
    print("X-axis")
default:
    print("Other")
}
```

## Patterns

- Values
- Ranges
- Tuples
- Enums
- Optionals
- Type casts

## Modern Swift 6.x Notes

Swift 6.3 added `@c` support for exposing Swift enums to C in interop scenarios. For normal iOS interviews, focus on Swift enum pattern matching and exhaustiveness.

## Interview Levels

Junior:

Switch checks a value against cases.

Senior:

Swift switch is a pattern matching tool with exhaustiveness checking, making it excellent for enum state and domain modeling.

## Quick Notes

- Switch must be exhaustive
- Pattern order matters
- Supports tuples and ranges
- Great with enums

## Interview Depth

Junior answer: `switch` compares a value with cases.

Mid-level answer: Swift `switch` supports more than simple equality. It can match ranges, tuples, optionals, enums, and conditions with `where`.

Senior answer: Pattern matching is a modeling tool. With enums and associated values, it lets the compiler enforce exhaustive handling of app states. Avoid `default` for app-owned enums when explicit cases would catch future changes.

iOS use case:

```swift
switch state {
case .loading:
    showLoading()
case .loaded(let user):
    show(user)
case .failed(let error):
    show(error)
}
```

Common mistakes:

- Using `default` too early.
- Writing overlapping cases in confusing order.
- Using strings instead of enums.

Practice:

1. Switch over tuple point.
2. Switch over enum state.
3. Explain exhaustiveness.
