# Day 18: Task Cancellation

## What Cancellation Means

Cancellation is a cooperative signal that a task's result is no longer needed.

It does not forcibly kill your code at a random line.

```swift
let task = Task {
    await loadSearchResults()
}

task.cancel()
```

The task is marked as cancelled. Your async code should observe that signal and stop when appropriate.

## Beginner Mental Model

Cancellation means:

```text
Please stop as soon as it is safe and reasonable.
```

It is not:

```text
The runtime instantly terminates your function.
```

## Checking Cancellation

```swift
try Task.checkCancellation()
```

This throws `CancellationError` if the current task is cancelled.

```swift
func process(items: [Item]) async throws -> [ResultItem] {
    var results: [ResultItem] = []

    for item in items {
        try Task.checkCancellation()
        let result = try await process(item)
        results.append(result)
    }

    return results
}
```

## Reading Cancellation State

```swift
if Task.isCancelled {
    return
}
```

Use this when throwing is not appropriate.

## Cancellation In SwiftUI

```swift
.task(id: query) {
    await viewModel.search(query)
}
```

When `query` changes, SwiftUI cancels the old task.

Good search implementation:

```swift
func search(_ query: String) async {
    guard !query.isEmpty else { return }

    do {
        try await Task.sleep(for: .milliseconds(300))
        try Task.checkCancellation()
        results = try await service.search(query)
    } catch is CancellationError {
        // Ignore expected cancellation.
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

This prevents stale search results from overwriting newer results.

## Cancellation In UIKit

```swift
final class SearchViewController: UIViewController {
    private var searchTask: Task<Void, Never>?

    func searchTextChanged(_ text: String) {
        searchTask?.cancel()
        searchTask = Task { [weak self] in
            await self?.performSearch(text)
        }
    }

    deinit {
        searchTask?.cancel()
    }
}
```

Store the task if you need to cancel previous work.

## Cancellation And Networking

Swift async APIs like `URLSession.shared.data(from:)` generally respond to task cancellation.

```swift
let task = Task {
    try await URLSession.shared.data(from: url)
}

task.cancel()
```

Your higher-level service should still handle `CancellationError` correctly.

## Cancellation And Cleanup

Use `defer` for cleanup.

```swift
func upload(file: URL) async throws {
    isUploading = true
    defer { isUploading = false }

    try Task.checkCancellation()
    try await uploader.upload(file)
}
```

Cleanup should happen whether work succeeds, fails, or is cancelled.

## Common Mistakes

- Treating cancellation as an error message for the user
- Not cancelling old search tasks
- Not checking cancellation in long loops
- Catching all errors and showing "Something went wrong" for cancellation
- Forgetting cleanup on cancellation
- Assuming cancellation instantly stops CPU-bound work

## Modern Swift 6.x Notes

Swift 6.x did not remove the cooperative nature of cancellation. It made debugging async flows better, especially in Swift 6.2 with improved task context.

Cancellation becomes easier to reason about when code uses structured concurrency because child tasks receive parent cancellation.

## Junior Interview Answer

Cancellation is a signal that a task should stop. Swift tasks cooperate with cancellation by checking `Task.isCancelled` or `Task.checkCancellation()`.

## Mid-Level Interview Answer

I use cancellation for search, disappearing screens, retry flows, and duplicate requests. I handle `CancellationError` separately so users do not see false error messages.

## Senior Interview Answer

Cancellation is part of API design. I make async operations cancellation-aware, avoid stale UI updates, propagate cancellation through structured concurrency, and ensure cleanup with `defer`. For CPU-heavy work, I check cancellation inside loops because there may be no suspension point.

## Points To Remember

- Cancellation is cooperative.
- `Task.checkCancellation()` throws.
- `Task.isCancelled` returns a boolean.
- SwiftUI `.task(id:)` cancels previous work.
- Do not show cancellation as a user-facing failure.

## Practice

1. Write a cancellable search function with debounce.
2. Add cancellation checks to a long loop.
3. Explain how SwiftUI `.task(id:)` uses cancellation.
4. Handle `CancellationError` separately from real failures.
