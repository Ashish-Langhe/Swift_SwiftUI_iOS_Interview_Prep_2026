# Day 5: Iteration Patterns

## Core Idea

Iteration means going through collection values.

```swift
for item in items {
    print(item)
}
```

## Common Patterns

Array values:

```swift
for name in names {
    print(name)
}
```

Index and value:

```swift
for (index, name) in names.enumerated() {
    print("\(index): \(name)")
}
```

Dictionary:

```swift
for (id, user) in usersById {
    print(id, user)
}
```

Set:

```swift
for tag in tags.sorted() {
    print(tag)
}
```

## Where Clause

```swift
for user in users where user.isActive {
    print(user.name)
}
```

## Real iOS Use Cases

- Rendering rows
- Validating fields
- Building view models
- Grouping API data

```swift
let rows = users.map { UserRow(title: $0.name) }
```

## Modern Swift 6.x Notes

For async data streams, Swift also has `for await` with `AsyncSequence`. That belongs to concurrency, but interviewers may connect iteration with modern async APIs.

## Interview Levels

Junior:

Use `for-in` to loop over collection items.

Mid-level:

Use `enumerated()` when index is needed, and sort dictionaries or sets when order matters.

Senior:

Choose iteration style based on intent. Use loops for control flow and early exits; use higher-order functions for transformations; avoid depending on unordered collection order.

## Quick Notes

- Use `for-in` for simple loops
- Use `enumerated()` for index and value
- Use `where` for simple filtering
- Sort unordered collections when order matters
- Use loops when you need `break` or `continue`

## Interview Depth

Junior answer: Iteration means going through each item in a collection using a loop.

Mid-level answer: Use direct `for-in` when you only need values, `enumerated()` when you need index and value, and dictionary tuple iteration for key-value pairs.

Senior answer: Iteration style communicates intent. A `for` loop is best for early exit, mutation, and complex branching. Higher-order methods are best for clear transformations. Avoid relying on set or dictionary order unless sorted.

iOS use case:

```swift
for (index, section) in sections.enumerated() {
    print("Render section \(index): \(section.title)")
}
```

Common mistakes:

- Using indices when values are enough.
- Relying on unordered collection order.
- Using `forEach` when `break` is needed.

Practice:

1. Iterate users with index.
2. Print dictionary keys sorted.
3. Explain `forEach` vs `for-in`.
