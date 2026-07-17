# Day 15: Escaping Vs Non-Escaping Closures

## Core Idea

A non-escaping closure is called before the function returns.

An escaping closure may be called after the function returns.

```swift
func load(completion: @escaping () -> Void) {
    DispatchQueue.main.async {
        completion()
    }
}
```

## Real iOS Use Cases

- Network callbacks
- Animation completions
- Stored actions

## Modern Swift 6.x Notes

Swift concurrency often replaces escaping callbacks with `async` functions, but escaping closures are still common in UIKit, SwiftUI actions, and legacy APIs.

## Interview Levels

Junior: `@escaping` means the closure can run later.

Senior: Escaping closures require attention to captures, retain cycles, actor isolation, and lifetime.

## Quick Notes

- Non-escaping is default.
- Use `@escaping` when stored/called later.
- Escaping closures can retain `self`.

## Interview Depth

Junior answer: Escaping closures can run after the function returns.

Mid-level answer: Non-escaping closures are default. Add `@escaping` when the closure is stored or called asynchronously.

Senior answer: Escaping closures affect lifetime and actor isolation. They commonly require capture lists and careful UI-thread/main-actor handling.

iOS use case:

```swift
func fetch(completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, _, error in
        completion(data.map(Result.success) ?? .failure(error!))
    }.resume()
}
```

Common mistakes: missing `@escaping`, strong self in callback, calling UI update off main actor.

Practice: write escaping completion, store closure, convert callback to async idea.
