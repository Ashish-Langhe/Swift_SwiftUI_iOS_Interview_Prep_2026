# Day 10: Basic Enums

## Core Idea

An enum defines a finite set of cases.

```swift
enum Direction {
    case north
    case south
    case east
    case west
}
```

## Real iOS Use Cases

- View states
- Routes
- Actions
- Permissions
- Filter modes

## Interview Levels

Junior:

An enum stores one of several possible cases.

Senior:

Enums model finite domain states and make invalid states harder to represent.

## Quick Notes

- Finite set of cases
- Great with switch
- Safer than strings
- Can have methods and properties

## Interview Depth

Junior answer: An enum is a type with a fixed set of possible cases.

Mid-level answer: Enums are safer than strings for states because the compiler checks valid cases.

Senior answer: Enums make impossible states harder to represent. They are a core Swift modeling tool for app routes, view states, actions, permissions, and modes.

iOS use case:

```swift
enum Tab {
    case home
    case search
    case settings
}
```

Common mistakes:

- Using strings for finite states.
- Adding meaningless default cases.
- Not switching exhaustively.

Practice:

1. Model login state with enum.
2. Switch over all cases.
3. Explain enum vs string state.
