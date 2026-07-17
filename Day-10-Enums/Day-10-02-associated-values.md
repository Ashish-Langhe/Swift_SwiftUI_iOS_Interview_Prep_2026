# Day 10: Associated Values

## Core Idea

Associated values attach data to enum cases.

```swift
enum LoadingState {
    case loading
    case loaded([String])
    case failed(String)
}
```

## Pattern Matching

```swift
switch state {
case .loaded(let items):
    print(items)
case .failed(let message):
    print(message)
case .loading:
    print("Loading")
}
```

## Interview Levels

Junior:

Associated values store extra data with enum cases.

Senior:

Associated values let each state carry exactly the data it needs, avoiding many optional properties.

## Quick Notes

- Case-specific data
- Excellent for state machines
- Use switch to extract values

## Interview Depth

Junior answer: Associated values store extra data inside enum cases.

Mid-level answer: Each enum case can carry the exact data it needs, such as loaded data or an error message.

Senior answer: Associated values prevent optional-state explosion. Instead of storing `data?`, `error?`, and `isLoading`, one enum can represent valid states precisely.

iOS use case:

```swift
enum ScreenState {
    case loading
    case loaded([Transaction])
    case failed(message: String)
}
```

Common mistakes:

- Confusing associated values with raw values.
- Adding associated values when a struct would be clearer.
- Using `default` and ignoring associated data.

Practice:

1. Model network state.
2. Extract associated value in switch.
3. Explain why it reduces optionals.
