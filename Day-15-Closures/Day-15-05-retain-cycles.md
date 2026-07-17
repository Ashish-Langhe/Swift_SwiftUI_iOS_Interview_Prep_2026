# Day 15: Retain Cycles

## Core Idea

A retain cycle happens when objects keep each other alive.

```swift
final class ViewModel {
    var onUpdate: (() -> Void)?

    func setup() {
        onUpdate = {
            self.refresh()
        }
    }

    func refresh() { }
}
```

The closure captures `self` strongly.

## Fix

```swift
onUpdate = { [weak self] in
    self?.refresh()
}
```

## Interview Levels

Junior: Retain cycles cause memory leaks.

Senior: Escaping closures stored by `self` often need capture lists. Use `weak` when `self` may disappear, `unowned` only when lifetime is guaranteed.

## Quick Notes

- Closures capture strongly by default.
- Use `[weak self]`.
- Stored escaping closures are common leak sources.

## Interview Depth

Junior answer: A retain cycle happens when objects keep each other alive.

Mid-level answer: Closures capture `self` strongly by default. If `self` stores the closure, a cycle can happen.

Senior answer: Retain cycles are ownership bugs. Use `[weak self]` when the closure should not keep the object alive. Use memory graph debugging and `deinit` logs to verify.

iOS use case:

```swift
viewModel.onUpdate = { [weak self] in
    self?.render()
}
```

Common mistakes: strong self in stored closure, using unowned unsafely, weak self everywhere without thinking.

Practice: create cycle, fix with weak, explain delegate weak pattern.
