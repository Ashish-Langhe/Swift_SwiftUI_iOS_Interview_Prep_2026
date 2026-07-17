# Day 20: Task Groups

## What Task Groups Are

Task groups run a dynamic number of child tasks.

Use them when you do not know the exact number of tasks at compile time.

```swift
try await withThrowingTaskGroup(of: Image.self) { group in
    for url in urls {
        group.addTask {
            try await imageLoader.load(url)
        }
    }

    var images: [Image] = []
    for try await image in group {
        images.append(image)
    }
    return images
}
```

## Task Group vs `async let`

Use `async let`:

- Fixed small number of operations
- Clear local variables

Use task group:

- Dynamic number of operations
- Loop-based child tasks
- Need to collect results as they finish
- Need custom cancellation/limits

## Result Ordering

Task group results arrive in completion order, not input order.

If order matters:

```swift
group.addTask {
    let image = try await loader.load(url)
    return (index, image)
}
```

Then sort or place results by index.

## Limiting Concurrency

Do not start thousands of requests at once.

Pattern:

```swift
func loadAll(urls: [URL], limit: Int) async throws -> [UIImage] {
    var iterator = urls.enumerated().makeIterator()
    var output = Array<UIImage?>(repeating: nil, count: urls.count)

    try await withThrowingTaskGroup(of: (Int, UIImage).self) { group in
        for _ in 0..<min(limit, urls.count) {
            if let (index, url) = iterator.next() {
                group.addTask { (index, try await loadImage(url)) }
            }
        }

        while let (index, image) = try await group.next() {
            output[index] = image

            if let (nextIndex, nextURL) = iterator.next() {
                group.addTask { (nextIndex, try await loadImage(nextURL)) }
            }
        }
    }

    return output.compactMap { $0 }
}
```

This avoids overwhelming resources.

## Cancellation

If the parent task is cancelled, the group children are cancelled.

You can also cancel remaining child tasks:

```swift
group.cancelAll()
```

Useful when one successful result is enough.

## Real iOS Use Cases

- Load visible cell images
- Fetch details for many IDs
- Preload local files
- Process photo library assets
- Upload multiple attachments
- Run independent validation checks

## Common Mistakes

- Assuming results preserve input order
- Starting unbounded tasks
- Mutating shared state from child tasks
- Ignoring cancellation
- Doing UI updates from group child tasks
- Using task groups for two fixed calls instead of `async let`

## Modern Swift 6.x Notes

Task groups interact with `Sendable` because child tasks may run concurrently. Values captured into child tasks and returned from them should be safe to transfer.

## Interview Answer

Task groups are for dynamic structured parallelism. I use them when the number of child tasks is based on data, collect results safely, preserve ordering explicitly when needed, and limit concurrency for networking or CPU-heavy work.

## Practice

1. Load images for an array of URLs with a task group.
2. Preserve result order using indexes.
3. Add a concurrency limit.
4. Explain why child tasks should return values instead of mutating shared arrays.
