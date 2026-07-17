# Day 19: `@concurrent`

## What `@concurrent` Means

`@concurrent` marks code that should run concurrently rather than remaining serialized on the caller's actor.

In Swift 6.2's approachable concurrency model, this makes concurrent execution explicit.

```swift
@concurrent
func decode(_ data: Data) async throws -> Image {
    try await decoder.decode(data)
}
```

## Why It Exists

If default isolation keeps code on the main actor, you need a clear way to say:

```text
This work should run off the actor because it is expensive or independently concurrent.
```

That is where `@concurrent` fits.

## Real iOS Example

```swift
struct ImagePipeline {
    static func image(from url: URL) async throws -> UIImage {
        let (data, _) = try await URLSession.shared.data(from: url)
        return try await decode(data)
    }

    @concurrent
    static func decode(_ data: Data) async throws -> UIImage {
        guard let image = UIImage(data: data) else {
            throw ImageError.invalidData
        }
        return image
    }
}
```

Network wait and decoding are separated. CPU work does not need to sit on UI isolation.

## What To Put In `@concurrent`

Good candidates:

- Image decoding
- JSON parsing for large payloads
- Compression
- Hashing
- Data transformation
- File processing
- CPU-heavy pure functions

Bad candidates:

- Direct UIKit updates
- Main-actor state mutation
- Unsafely shared mutable class access
- Code that depends on actor-isolated state

## Data Transfer Still Matters

The data passed into concurrent code should be safe to send.

```swift
struct DecodeRequest: Sendable {
    let data: Data
    let scale: CGFloat
}
```

Do not use `@concurrent` to bypass isolation rules.

## Common Mistakes

- Marking UI methods `@concurrent`
- Passing non-sendable mutable references
- Using it as a warning silencer
- Forgetting cancellation in expensive work
- Creating too much parallel CPU work

## Modern Swift 6.2 Notes

`@concurrent` is part of Swift 6.2's approachable concurrency story. It pairs with default actor isolation and caller-context async behavior by making opt-in concurrency visible at the function boundary.

## Junior Interview Answer

`@concurrent` marks async work that should be allowed to run concurrently instead of staying on the caller's actor.

## Mid-Level Interview Answer

I use `@concurrent` for CPU-heavy or independent work like image decoding, while keeping UI state on the main actor.

## Senior Interview Answer

`@concurrent` is an API signal. It says this operation is intentionally concurrent, so its inputs and captured state must be safe to transfer. I use it to separate expensive work from actor-isolated UI flows without weakening data-race safety.

## Practice

1. Mark image decoding as `@concurrent`.
2. Explain why UI updates should not be `@concurrent`.
3. Create a `Sendable` request type for concurrent work.
4. Add cancellation checks to a CPU-heavy concurrent loop.
