# Day 12: Protocol Extensions

## Core Idea

Protocol extensions provide default behavior.

```swift
protocol Loggable {
    var logName: String { get }
}

extension Loggable {
    func log() {
        print(logName)
    }
}
```

## Real iOS Use Cases

- Shared default behavior
- Convenience helpers
- Reducing duplicate conformer code

## Interview Levels

Junior: Protocol extensions add default methods.

Senior: Protocol extensions are powerful, but be careful with static dispatch differences and hidden default behavior. Keep defaults simple and unsurprising.

## Quick Notes

- Add default implementations.
- Can constrain extensions with `where`.
- Helps protocol-oriented programming.

## Interview Depth

Junior answer: Protocol extensions add shared methods or default implementations.

Mid-level answer: They reduce duplicate code across conforming types.

Senior answer: Default implementations should be unsurprising. Understand whether a method is a protocol requirement or only an extension method, because dispatch behavior can differ.

iOS use case:

```swift
extension AnalyticsTracking {
    func trackScreen(_ name: String) {
        track("screen_view:\(name)")
    }
}
```

Common mistakes: hiding important behavior in defaults, making extensions too broad, relying on confusing dispatch.

Practice: add default method, add constrained extension, explain dispatch caution.
