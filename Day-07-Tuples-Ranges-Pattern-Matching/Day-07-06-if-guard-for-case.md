# Day 7: If Case, Guard Case, And For Case

## Core Idea

Use `if case`, `guard case`, and `for case` when matching one pattern.

```swift
if case .success(let value) = result {
    print(value)
}
```

## Guard Case

```swift
guard case .loaded(let user) = state else {
    return
}
```

## For Case

```swift
for case let value? in optionalValues {
    print(value)
}
```

## Real iOS Use Cases

- Extracting loaded state
- Filtering analytics events
- Handling optional arrays
- Reducer/state-machine logic

## Interview Levels

Junior:

These are ways to match patterns outside a full switch.

Senior:

They are useful when one case matters, but overuse can become cryptic. Use full switch when multiple states need explicit handling.

## Quick Notes

- `if case`: one optional pattern branch
- `guard case`: required pattern
- `for case`: filter matching values
- Prefer clarity over cleverness

## Interview Depth

Junior answer: `if case`, `guard case`, and `for case` match patterns without writing a full switch.

Mid-level answer: Use `if case` when one case matters, `guard case` when a case is required to continue, and `for case` when iterating only matching values.

Senior answer: These constructs are expressive but can become cryptic. Use them when they remove noise. Use a full switch when multiple states deserve visible handling.

iOS use case:

```swift
guard case .loaded(let profile) = state else {
    return
}

render(profile)
```

Common mistakes:

- Overusing shorthand in team code.
- Hiding important state handling.
- Forgetting that `guard case` must exit on failure.

Practice:

1. Extract loaded value with `if case`.
2. Require success state with `guard case`.
3. Iterate non-nil values with `for case`.
