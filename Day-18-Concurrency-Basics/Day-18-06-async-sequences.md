# Day 18: Async Sequences

## What An Async Sequence Is

An async sequence produces values over time.

Normal sequence:

```swift
for item in items {
    print(item)
}
```

Async sequence:

```swift
for await value in stream {
    print(value)
}
```

Use async sequences when values arrive gradually instead of all at once.

## Beginner Mental Model

An array gives you values now.

An async sequence gives you values later, one by one.

Examples:

- Notifications
- Lines from a file
- WebSocket messages
- Location updates
- Progress updates
- State observation
- Async streams

## `for await`

```swift
for await message in messages {
    print(message)
}
```

If the sequence can throw:

```swift
do {
    for try await line in fileLines {
        print(line)
    }
} catch {
    print(error)
}
```

## Async Sequence And Cancellation

A `for await` loop usually runs until:

- The sequence ends
- The task is cancelled
- The loop breaks
- An error is thrown

```swift
for await event in events {
    if Task.isCancelled { break }
    handle(event)
}
```

## Real iOS Example: Notification Stream

Modern Foundation continues to improve typed async-friendly notification APIs, but the mental model is the same: observe values over time.

```swift
for await notification in NotificationCenter.default.notifications(
    named: UIApplication.didBecomeActiveNotification
) {
    await refresh()
}
```

This is easier to reason about than manually adding and removing observers.

## Real iOS Example: Progress

```swift
func observeUploadProgress(_ progress: AsyncStream<Double>) async {
    for await value in progress {
        self.progress = value
    }
}
```

Each progress value updates UI.

## Creating A Simple AsyncStream

```swift
func countdown(from start: Int) -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for value in stride(from: start, through: 0, by: -1) {
                continuation.yield(value)
                try? await Task.sleep(for: .seconds(1))
            }
            continuation.finish()
        }
    }
}
```

Usage:

```swift
for await value in countdown(from: 3) {
    print(value)
}
```

## Swift 6.2 Observations

Swift 6.2 introduced `Observations`, an async sequence for streaming transactional changes from observable state.

Conceptually:

```swift
for await value in Observations({ model.status }) {
    print(value)
}
```

This matters because modern Swift increasingly models long-lived change streams as async sequences.

## When To Use Async Sequence

Use async sequence when:

- There can be multiple values
- Values arrive over time
- The consumer should use async/await style
- Cancellation should stop observation
- You want structured control instead of callback registration

Do not use it for a single value. Use an async function instead.

## Common Mistakes

- Using async sequence for one-time network calls
- Forgetting that `for await` can run forever
- Updating UI outside the main actor
- Not finishing custom streams
- Retaining self forever in stream producers
- Ignoring cancellation cleanup

## Junior Interview Answer

An async sequence is like a sequence whose values arrive asynchronously. You consume it with `for await`.

## Mid-Level Interview Answer

I use async sequences for streams of values like notifications, progress, WebSocket messages, and state changes. The consuming task can be cancelled, which stops observation.

## Senior Interview Answer

AsyncSequence is Swift's abstraction for asynchronous streams. It composes with structured concurrency, cancellation, and actor isolation. I use it to replace callback-based observers when the domain naturally produces many values over time, and I design custom streams carefully so they finish and clean up resources.

## Points To Remember

- Single async result: async function.
- Multiple async values: async sequence.
- Consume with `for await`.
- Throwing streams use `for try await`.
- Long-lived streams need cancellation and cleanup.

## Practice

1. Write a countdown `AsyncStream`.
2. Consume notifications using `for await`.
3. Explain why a WebSocket fits async sequence better than a plain async function.
4. Add cancellation cleanup to a stream producer.
