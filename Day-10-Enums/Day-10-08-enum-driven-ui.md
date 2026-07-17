# Day 10: Enum-Driven UI

## Core Idea

Enum-driven UI renders different views for different states.

```swift
switch state {
case .loading:
    ProgressView()
case .loaded(let user):
    ProfileContent(user: user)
case .failed(let message):
    Text(message)
}
```

## Real iOS Use Cases

- SwiftUI screens
- UIKit coordinators
- Sheet routing
- Alert routing
- Empty/loading/error states

## Interview Levels

Junior:

Use switch to show UI for enum cases.

Senior:

Enum-driven UI makes UI state explicit, testable, and exhaustive. It avoids scattered booleans like `isLoading`, `error`, and optional data competing with each other.

## Quick Notes

- Great for SwiftUI
- Exhaustive rendering
- Avoids invalid UI state
- Works well with reducers/view models

## Interview Depth

Junior answer: Enum-driven UI means showing different UI for different enum cases.

Mid-level answer: SwiftUI works naturally with enum state because `switch` can render loading, content, error, and empty screens.

Senior answer: Enum-driven UI centralizes state. It reduces scattered conditional logic and makes UI behavior easier to test. It also pairs well with reducer-style architectures and unidirectional data flow.

iOS use case:

```swift
@ViewBuilder
func content(for state: ProfileState) -> some View {
    switch state {
    case .loading:
        ProgressView()
    case .loaded(let user):
        Text(user.name)
    case .failed(let message):
        Text(message)
    }
}
```

Common mistakes:

- Rendering only happy path.
- Using default and hiding new states.
- Mixing enum state with separate conflicting booleans.

Practice:

1. Render enum state in SwiftUI.
2. Add new enum case and update switch.
3. Explain why exhaustive UI is safer.
