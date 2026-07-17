# Day 32: Main-Thread Work And Responsiveness

## Why Main Thread Matters

The main thread/main actor handles UI events and rendering.

If you block it, users see:

- Frozen taps
- Delayed typing
- Stuttering scroll
- Slow animations
- Watchdog termination in extreme cases

## Common Main-Thread Blockers

- JSON decoding large payloads
- Image resizing
- File writes
- Database queries
- Sorting/filtering large arrays
- Compression/encryption
- PDF rendering
- Regex over large text
- Synchronous network calls
- Excessive SwiftUI updates

## Bad Example

```swift
@MainActor
final class ProductsModel {
    var rows: [ProductRowViewModel] = []

    func load() async {
        let data = try! Data(contentsOf: fileURL)
        let products = try! JSONDecoder().decode([Product].self, from: data)
        rows = products.map(ProductRowViewModel.init)
    }
}
```

This performs file and decoding work on Main Actor.

## Better Example

```swift
struct ProductLoader {
    func load(from url: URL) async throws -> [ProductRowViewModel] {
        try await Task.detached(priority: .userInitiated) {
            let data = try Data(contentsOf: url)
            let products = try JSONDecoder().decode([Product].self, from: data)
            return products.map(ProductRowViewModel.init)
        }.value
    }
}

@MainActor
final class ProductsModel {
    var rows: [ProductRowViewModel] = []
    let loader: ProductLoader

    func load() async {
        do {
            rows = try await loader.load(from: fileURL)
        } catch {
            rows = []
        }
    }
}
```

Keep UI mutation on Main Actor, move CPU/blocking work away.

## Swift 6.2 Main Actor Context

Swift 6.2 introduced approachable concurrency changes, including options around default main actor isolation and caller-context async execution.

Senior implication:

- UI code may stay serialized on Main Actor more naturally.
- CPU-heavy work can accidentally remain on Main Actor.
- Use explicit concurrency boundaries for heavy parallel work.
- Use `@concurrent` when code should run concurrently away from actor isolation.

## Using @concurrent Conceptually

```swift
@concurrent
func renderThumbnails(_ images: [ImageData]) async -> [Thumbnail] {
    images.map(renderThumbnail)
}
```

The idea is to make concurrency intent explicit: this work may run concurrently and should not block Main Actor.

Exact usage depends on project language mode and compiler settings.

## Detecting Main Thread Hangs

Use:

- Time Profiler
- System Trace
- Swift Concurrency instrument
- Hangs diagnostics
- Signposts
- Main Thread Checker for API misuse

Trace question:

```text
What was the main thread doing while the user waited?
```

## File I/O Example

Bad:

```swift
Button("Save") {
    try? data.write(to: url)
}
```

Better:

```swift
Button("Save") {
    Task {
        await model.save()
    }
}
```

Model:

```swift
func save() async {
    let snapshot = state

    do {
        try await fileWriter.write(snapshot)
        saveStatus = .saved
    } catch {
        saveStatus = .failed
    }
}
```

## Common Mistakes

- Assuming `async` automatically means off-main.
- Calling blocking Foundation APIs from Main Actor.
- Creating tasks in SwiftUI that inherit Main Actor and do CPU work.
- Updating UI state from background threads.
- No signposts around suspected hangs.
- Fixing by sprinkling `DispatchQueue.global` randomly.

## Senior Artifact

```text
Artifact: Main Thread Investigation
Symptom:
Trace interval:
Main thread stack:
Blocking operation:
Actor context:
Work moved to:
UI mutation point:
Verification:
```

## Interview Notes

Junior:

Do not block the main thread because it handles UI.

Mid-level:

Move heavy work off main, then update UI on Main Actor.

Senior:

I inspect traces to identify main-thread blocking, understand actor inheritance, move CPU/blocking work to explicit concurrency boundaries, and keep UI mutation isolated to Main Actor.

## Practice

1. Move JSON decoding off Main Actor.
2. Add signposts around save-to-file.
3. Explain why `async` does not always mean background.
4. Identify a main-thread stack in Time Profiler.
