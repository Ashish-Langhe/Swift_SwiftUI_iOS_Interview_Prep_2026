# Day 9: Memory Implications

## Core Idea

Classes are managed by ARC: Automatic Reference Counting.

```swift
final class User { }

var user: User? = User()
user = nil
```

When no strong references remain, the object is deallocated.

## Retain Cycles

```swift
final class Owner {
    var closure: (() -> Void)?

    func setup() {
        closure = { [weak self] in
            self?.doWork()
        }
    }

    func doWork() { }
}
```

Use `weak` to avoid cycles when appropriate.

## Modern Swift 6.x Notes

Reference types with mutable state are more delicate in concurrency. Actors are reference types too, but protect isolated mutable state.

## Interview Levels

Junior:

ARC manages class memory.

Senior:

Memory issues with classes often involve ownership, retain cycles, closure captures, and shared mutable state. I use weak references for delegates and capture lists for closures when needed.

## Quick Notes

- Classes use ARC
- Strong references keep objects alive
- Weak references avoid ownership
- Closures can capture strongly
- Deinit helps detect leaks

## Interview Depth

Junior answer: Classes are managed by ARC, which keeps objects alive while strong references exist.

Mid-level answer: Retain cycles happen when objects strongly reference each other. Use `weak` for delegates and capture lists for closures.

Senior answer: Memory management is ownership design. Decide who owns what. Use `weak` for back-references, avoid stored closures capturing owners strongly, and use Instruments or memory graph debugging when leaks appear.

iOS use case:

```swift
protocol CoordinatorDelegate: AnyObject { }

final class Coordinator {
    weak var delegate: CoordinatorDelegate?
}
```

Common mistakes:

- Strong delegates.
- Strong `self` in stored escaping closures.
- Using `unowned` without guaranteed lifetime.

Practice:

1. Create retain cycle example.
2. Fix with weak.
3. Explain ARC.
