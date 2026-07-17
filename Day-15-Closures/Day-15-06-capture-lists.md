# Day 15: Capture Lists

## Core Idea

Capture lists control how closures capture values.

```swift
closure = { [weak self] in
    self?.doWork()
}
```

Capture by value:

```swift
let name = "Aarav"
let closure = { [name] in
    print(name)
}
```

## Weak Vs Unowned

Use `weak` when the captured object may become nil.

Use `unowned` only when the object must outlive the closure.

## Interview Levels

Junior: Capture lists change how closures capture variables.

Senior: Capture lists are memory-management tools. Choose weak/unowned based on ownership and lifetime, not habit.

## Quick Notes

- Written before `in`.
- `[weak self]` prevents cycles.
- `unowned` can crash if lifetime assumption is wrong.

## Interview Depth

Junior answer: Capture lists control how values are captured by a closure.

Mid-level answer: `[weak self]` captures self weakly to avoid retain cycles. `[unowned self]` avoids optional self but can crash if self is gone.

Senior answer: Capture choice should follow ownership. Weak means optional lifetime; unowned means guaranteed lifetime. Capturing values by copy can also freeze state intentionally.

iOS use case:

```swift
service.load { [weak self] result in
    guard let self else { return }
    self.handle(result)
}
```

Common mistakes: unowned with uncertain lifetime, forgetting guard self, capturing stale values unintentionally.

Practice: use weak self, use capture by value, compare weak/unowned.
