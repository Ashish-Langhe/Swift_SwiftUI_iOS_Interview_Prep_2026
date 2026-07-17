# Day 10: Recursive Enums

## Core Idea

A recursive enum references itself.

```swift
indirect enum Expression {
    case number(Int)
    case addition(Expression, Expression)
}
```

Use `indirect` so Swift stores recursive cases safely.

## Real Use Cases

- Trees
- Expressions
- Nested menus
- File systems

## Interview Levels

Junior:

A recursive enum can contain another value of the same enum.

Senior:

Recursive enums are useful for tree-like data. They are powerful but less common in everyday iOS UI code.

## Quick Notes

- Use `indirect`
- Models trees
- Advanced enum feature

## Interview Depth

Junior answer: A recursive enum can contain another value of the same enum.

Mid-level answer: Use `indirect` so Swift stores recursive enum cases safely.

Senior answer: Recursive enums are ideal for tree-like data, expressions, nested menus, and parsers. They are powerful but should not be forced into ordinary flat state.

iOS use case:

```swift
indirect enum SettingsNode {
    case item(title: String)
    case group(title: String, children: [SettingsNode])
}
```

Common mistakes:

- Forgetting `indirect`.
- Using recursive enums when a simple array is enough.
- Making recursion hard to render or test.

Practice:

1. Model nested menu.
2. Count nodes recursively.
3. Explain why `indirect` is needed.
