# Day 11: Lazy Properties

## Core Idea

A `lazy` property is initialized only when first accessed.

```swift
final class ImageLoader {
    lazy var cache: [URL: Data] = [:]
}
```

Lazy properties must be `var` because their value changes from uninitialized to initialized.

## Closure Initialization

```swift
final class DateTextFormatter {
    lazy var formatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        return formatter
    }()
}
```

## Real iOS Use Cases

- Formatters
- Heavy helper objects
- Views created only when needed
- Caches

## Modern Swift 6.x Notes

Be careful using lazy properties from multiple concurrent contexts. Lazy initialization is not automatically a full synchronization strategy for shared mutable objects.

## Interview Levels

Junior: `lazy` delays property creation until first use.

Senior: Lazy properties are useful for expensive setup, but they hide initialization timing. In concurrent or shared code, prefer explicit initialization or actor isolation when thread-safety matters.

## Quick Notes

- Initialized on first access.
- Must be `var`.
- Useful for expensive objects.
- Be careful with concurrency and side effects.

## Interview Depth

Junior answer: A lazy property is created only when first used.

Mid-level answer: Lazy properties help defer expensive setup and can reference `self` after initialization.

Senior answer: Lazy hides initialization timing. Do not assume it solves thread safety. For shared objects in concurrent code, use explicit initialization, actors, or another synchronization strategy.

iOS use case:

```swift
lazy var currencyFormatter: NumberFormatter = {
    let formatter = NumberFormatter()
    formatter.numberStyle = .currency
    return formatter
}()
```

Common mistakes: using lazy for everything, doing surprising side effects, accessing mutable lazy state concurrently.

Practice: create lazy formatter, explain why lazy must be var, discuss concurrency risk.
