# Day 8: Copy-On-Write Intuition

## Core Idea

Copy-on-write lets value types share storage until mutation.

```swift
var first = [1, 2, 3]
var second = first
second.append(4)
```

Swift avoids copying array storage until `second` mutates.

## Why It Matters

You get value semantics with good performance.

## Real iOS Use Cases

- Arrays of view models
- App state snapshots
- Editing drafts

## Interview Levels

Junior:

Swift copies values when needed.

Senior:

Copy-on-write is an optimization behind value semantics. It gives predictable behavior without eagerly copying large storage. Custom COW types are advanced and require careful uniqueness checks.

## Quick Notes

- Shared storage until mutation
- Used by Swift collections
- Preserves value semantics
- Performance optimization, not different behavior

## Interview Depth

Junior answer: Copy-on-write means Swift delays copying until a value changes.

Mid-level answer: Arrays, dictionaries, sets, and strings use copy-on-write so assignments are efficient while still behaving like independent values.

Senior answer: Copy-on-write is an implementation optimization that preserves value semantics. Custom copy-on-write types are possible but require careful uniqueness checks and are usually advanced.

iOS use case:

```swift
var rows = viewModel.rows
rows.append(newRow)
```

The original row storage may be shared until mutation.

Common mistakes:

- Thinking assignment always deeply copies immediately.
- Thinking copy-on-write changes visible behavior.
- Ignoring reference types stored inside collections.

Practice:

1. Explain COW with arrays.
2. Explain why COW helps performance.
3. Explain value behavior vs storage sharing.
