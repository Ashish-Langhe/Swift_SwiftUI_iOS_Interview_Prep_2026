# Day 11: Property Observers

## Core Idea

Property observers run when a stored property changes.

```swift
var score = 0 {
    willSet {
        print("New score will be \(newValue)")
    }
    didSet {
        print("Old score was \(oldValue)")
    }
}
```

## Real iOS Use Cases

- Refresh UI after state changes
- Validate local changes
- Trigger lightweight side effects

```swift
var searchText = "" {
    didSet {
        print("Search changed to \(searchText)")
    }
}
```

## Interview Levels

Junior: `willSet` runs before change, `didSet` runs after change.

Senior: Observers are convenient, but hidden side effects can make state flow hard to debug. In SwiftUI/Observation-style code, prefer explicit state flow and avoid doing heavy work in observers.

## Quick Notes

- `willSet` sees `newValue`.
- `didSet` sees `oldValue`.
- Not called during initial assignment.
- Keep observers lightweight.

## Interview Depth

Junior answer: Property observers run before or after a property changes.

Mid-level answer: Use `willSet` to inspect the incoming value and `didSet` to react after assignment.

Senior answer: Observers can hide side effects. Keep them small and local. Avoid heavy operations, network calls, navigation, or complex validation in observers.

iOS use case:

```swift
var searchText = "" {
    didSet {
        guard searchText != oldValue else { return }
        updateSearchResults()
    }
}
```

Common mistakes: recursive updates, expensive work, assuming observers run during initialization.

Practice: use `didSet`, compare `oldValue`, explain observer risks.
