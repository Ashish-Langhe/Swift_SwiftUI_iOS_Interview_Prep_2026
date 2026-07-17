# Day 20: AsyncStream

## What AsyncStream Is

`AsyncStream` lets you wrap callback-style or event-based APIs as an async sequence.

```swift
func values() -> AsyncStream<Int> {
    AsyncStream { continuation in
        continuation.yield(1)
        continuation.yield(2)
        continuation.finish()
    }
}
```

Consume:

```swift
for await value in values() {
    print(value)
}
```

## When To Use AsyncStream

Use it for:

- Delegate callbacks
- Timer events
- Progress updates
- WebSocket messages
- Location updates
- Legacy observer APIs

Do not use it for a single network result.

## Wrapping A Delegate

```swift
final class LocationStream: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private var continuation: AsyncStream<CLLocation>.Continuation?

    func locations() -> AsyncStream<CLLocation> {
        AsyncStream { continuation in
            self.continuation = continuation
            manager.delegate = self
            manager.startUpdatingLocation()

            continuation.onTermination = { [weak self] _ in
                self?.manager.stopUpdatingLocation()
                self?.continuation = nil
            }
        }
    }

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        locations.forEach { continuation?.yield($0) }
    }
}
```

## AsyncThrowingStream

Use `AsyncThrowingStream` when the stream can fail.

```swift
func messages() -> AsyncThrowingStream<Message, Error> {
    AsyncThrowingStream { continuation in
        socket.onMessage = { continuation.yield($0) }
        socket.onError = { continuation.finish(throwing: $0) }
        socket.onClose = { continuation.finish() }
    }
}
```

## Cleanup Is Critical

Always handle termination:

```swift
continuation.onTermination = { _ in
    token.invalidate()
}
```

Without cleanup, streams can leak observers, delegates, sockets, or tasks.

## Backpressure And Buffering

Streams can produce values faster than consumers read them.

Use buffering policies when needed:

```swift
AsyncStream(bufferingPolicy: .bufferingNewest(1)) { continuation in
    // Yield latest value only.
}
```

This is useful for UI progress or location where latest value matters most.

## Common Mistakes

- Not calling `finish()`
- Not cleaning up in `onTermination`
- Capturing self strongly forever
- Using unbounded buffering for high-frequency events
- Updating UI from stream producer instead of consumer
- Ignoring cancellation

## Modern Swift 6.x Notes

Async streams fit Swift's structured concurrency model, but producer code must still be memory-safe and cancellation-aware. With Swift 6 strict checking, captured values and continuation usage should be designed carefully.

Swift 6.2's `Observations` shows the same direction: model state changes as async sequences.

## Interview Answer

`AsyncStream` bridges callback/event APIs into Swift's async sequence world. I use it for repeated values over time, add termination cleanup, choose buffering carefully, and consume it from a cancellable task.

## Practice

1. Wrap a timer as `AsyncStream<Date>`.
2. Wrap a delegate callback as `AsyncStream`.
3. Add `onTermination` cleanup.
4. Explain bufferingNewest for search/progress updates.
