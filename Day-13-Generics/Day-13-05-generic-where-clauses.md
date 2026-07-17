# Day 13: Generic Where Clauses

## Core Idea

`where` clauses express advanced generic constraints.

```swift
func allEqual<S1: Sequence, S2: Sequence>(_ a: S1, _ b: S2) -> Bool
where S1.Element == S2.Element, S1.Element: Equatable {
    Array(a) == Array(b)
}
```

## Real iOS Use Cases

- Reusable collection helpers
- Repository constraints
- Protocol extension specialization

## Interview Levels

Junior: `where` adds extra rules to generics.

Senior: `where` clauses make complex type relationships explicit and keep APIs generic without losing correctness.

## Quick Notes

- Adds constraints after signature.
- Common with associated types.
- Useful in protocol extensions.

## Interview Depth

Junior answer: A `where` clause adds extra generic rules.

Mid-level answer: `where` clauses are helpful when constraints involve associated types or relationships between multiple generic types.

Senior answer: Use `where` to make complex relationships explicit without making the primary signature unreadable.

iOS use case:

```swift
extension Array where Element: Identifiable {
    func ids() -> [Element.ID] {
        map(\.id)
    }
}
```

Common mistakes: writing unreadable constraints, using where when simple `T: Protocol` is enough, hiding important type relationships.

Practice: constrain array element, constrain sequence elements equal, add protocol extension where clause.
