# Day 9: Initializers And Deinitializers

## Core Idea

Initializers create instances. Deinitializers run before class instances are destroyed.

```swift
final class Session {
    init() {
        print("Created")
    }

    deinit {
        print("Destroyed")
    }
}
```

## Real iOS Use Cases

- Starting/stopping observers
- Cleaning resources
- Debugging lifetime

## Interview Levels

Junior:

`init` creates an object, `deinit` runs when it is removed from memory.

Senior:

Deinit is useful for cleanup and leak debugging, but ARC timing depends on references. If deinit never runs, a retain cycle may exist.

## Quick Notes

- Classes can have deinit
- Structs do not have deinit
- ARC calls deinit when last strong reference is gone
- Useful for cleanup

## Interview Depth

Junior answer: `init` creates an object. `deinit` runs when a class instance is removed from memory.

Mid-level answer: Deinitializers are useful for cleanup, removing observers, and debugging object lifetime.

Senior answer: If `deinit` does not run when expected, investigate strong references and retain cycles. Do not rely on deinit for critical user-facing workflow completion.

iOS use case:

```swift
final class KeyboardObserver {
    init() {
        print("Start observing")
    }

    deinit {
        print("Stop observing")
    }
}
```

Common mistakes:

- Expecting immediate deinit while references still exist.
- Creating retain cycles.
- Forgetting to remove legacy observers/resources.

Practice:

1. Add print in deinit.
2. Create and release an object.
3. Explain why deinit might not run.
