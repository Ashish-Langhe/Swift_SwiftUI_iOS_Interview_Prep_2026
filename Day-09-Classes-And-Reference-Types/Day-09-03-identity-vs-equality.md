# Day 9: Identity Vs Equality

## Core Idea

Identity asks whether two references point to the same object.

Equality asks whether two values are considered equal.

```swift
let a = User()
let b = a

print(a === b) // true
```

`===` checks identity for class instances.

## Equality

```swift
struct User: Equatable {
    let id: String
}

User(id: "1") == User(id: "1")
```

## Interview Levels

Junior:

`==` checks equality. `===` checks same class object.

Senior:

Identity is about object instance; equality is domain-defined equivalence. Choose carefully for caching, diffing, and UI updates.

## Quick Notes

- `==` equality
- `===` identity
- Identity only for class references
- Equatable defines value equality

## Interview Depth

Junior answer: `==` checks whether values are equal. `===` checks whether two class references point to the same object.

Mid-level answer: Equality is domain-defined. Identity is object instance sameness.

Senior answer: This matters in diffing, caching, navigation stacks, and object lifecycle debugging. Two users may be equal by ID but still be different object instances.

iOS use case:

```swift
if oldViewController === newViewController {
    print("Same screen instance")
}
```

Common mistakes:

- Using `===` with structs.
- Assuming equality means same object.
- Implementing `Equatable` with too many fields when identity should be ID-based.

Practice:

1. Compare two class references.
2. Implement `Equatable` by ID.
3. Explain identity vs equality in UI diffing.
