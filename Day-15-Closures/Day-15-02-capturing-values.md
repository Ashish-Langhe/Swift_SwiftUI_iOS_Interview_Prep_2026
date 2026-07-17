# Day 15: Capturing Values

## Core Idea

Closures can capture values from surrounding scope.

```swift
func makeCounter() -> () -> Int {
    var count = 0
    return {
        count += 1
        return count
    }
}
```

## Interview Levels

Junior: Closures can use variables outside themselves.

Senior: Capturing is powerful but affects lifetime and memory. Captured reference types can create retain cycles.

## Quick Notes

- Closures capture surrounding values.
- Captures can extend lifetime.
- Be careful with classes and `self`.

## Interview Depth

Junior answer: A closure can use variables from the scope where it was created.

Mid-level answer: Captured values can stay alive as long as the closure exists.

Senior answer: Capturing is both useful and dangerous. Capturing class instances strongly in escaping closures can create memory leaks or unexpected lifetimes.

iOS use case:

```swift
func makeFilter(query: String) -> (User) -> Bool {
    { user in user.name.localizedCaseInsensitiveContains(query) }
}
```

Common mistakes: accidental strong self, assuming captured var is copied always, hidden lifetime extension.

Practice: make counter closure, capture query, explain captured lifetime.
