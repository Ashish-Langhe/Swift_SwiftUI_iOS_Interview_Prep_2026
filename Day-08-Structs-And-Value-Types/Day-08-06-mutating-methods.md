# Day 8: Mutating Methods

## Core Idea

Struct methods cannot change properties unless marked `mutating`.

```swift
struct Counter {
    var value = 0

    mutating func increment() {
        value += 1
    }
}
```

The instance must be `var`.

```swift
var counter = Counter()
counter.increment()
```

## Real iOS Use Cases

- Updating draft models
- Toggling selection
- Changing local state

```swift
struct Selection {
    var ids: Set<String> = []

    mutating func toggle(_ id: String) {
        if ids.contains(id) {
            ids.remove(id)
        } else {
            ids.insert(id)
        }
    }
}
```

## Interview Levels

Junior:

Use `mutating` when a struct method changes its properties.

Senior:

Mutating methods are explicit value changes. They are useful for local state, but returning modified copies can sometimes be easier for functional composition.

## Quick Notes

- Required to modify struct properties
- Instance must be `var`
- Shows mutation clearly
- Useful for value-state updates

## Interview Depth

Junior answer: `mutating` lets a struct method change its properties.

Mid-level answer: Struct methods are non-mutating by default. If a method changes state, Swift requires `mutating` so the mutation is explicit.

Senior answer: Mutating methods are clear for local value updates, but returning a new modified value can be better for immutable pipelines, reducers, and predictable state transitions.

iOS use case:

```swift
struct FormDraft {
    var name = ""

    mutating func normalize() {
        name = name.trimmingCharacters(in: .whitespacesAndNewlines)
    }
}
```

Common mistakes:

- Forgetting `mutating`.
- Calling mutating method on `let`.
- Mutating too much hidden state.

Practice:

1. Write a `toggle` mutating method.
2. Try calling it on `let`.
3. Compare mutating method vs returning copy.
