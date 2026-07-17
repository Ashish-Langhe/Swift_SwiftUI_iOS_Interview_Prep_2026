# Day 18: Structured Concurrency

## What Structured Concurrency Means

Structured concurrency means child async work has a clear parent scope.

When the parent scope exits:

- Child work must finish, throw, or be cancelled.
- Results are collected in a predictable place.
- Errors and cancellation propagate in controlled ways.

This is different from starting random tasks that outlive the code that created them.

## Beginner Mental Model

Structured concurrency is like assigning work inside a function and not leaving until all assigned work is accounted for.

```text
Parent task
  child task A
  child task B
  child task C
```

The parent knows about its children.

## Why It Matters

Without structure:

```swift
func loadScreen() {
    Task { await loadProfile() }
    Task { await loadPosts() }
}
```

This starts work but does not clearly manage results, failure, or cancellation.

With structure:

```swift
func loadScreen() async throws -> ScreenData {
    async let profile = api.profile()
    async let posts = api.posts()

    return try await ScreenData(
        profile: profile,
        posts: posts
    )
}
```

The work belongs to the function.

## `async let`

Use `async let` for a small, known number of independent async operations.

```swift
async let user = api.user()
async let settings = api.settings()

let dashboard = try await DashboardData(
    user: user,
    settings: settings
)
```

The child tasks are automatically awaited before leaving scope.

## Task Groups

Use task groups when the number of child tasks is dynamic.

```swift
func loadImages(urls: [URL]) async throws -> [UIImage] {
    try await withThrowingTaskGroup(of: UIImage.self) { group in
        for url in urls {
            group.addTask {
                try await imageLoader.image(from: url)
            }
        }

        var images: [UIImage] = []
        for try await image in group {
            images.append(image)
        }
        return images
    }
}
```

Task groups are covered deeply on Day 20.

## Error Propagation

In throwing structured concurrency, one child error can cause sibling work to be cancelled.

```swift
async let profile = api.profile()
async let orders = api.orders()

let data = try await ProfileData(profile: profile, orders: orders)
```

If `profile` throws, the operation fails and unrelated child work does not need to continue.

## Cancellation Propagation

If a parent task is cancelled, child tasks are also cancelled.

```swift
.task(id: userID) {
    await viewModel.load(userID: userID)
}
```

When `userID` changes, SwiftUI cancels the previous task and starts a new one.

Your code still needs to check cancellation if it does CPU work or loops.

## Structured vs Unstructured

Structured:

```swift
async let a = loadA()
async let b = loadB()
let result = await combine(a, b)
```

Unstructured:

```swift
let task = Task { await loadA() }
```

Unstructured tasks are not bad, but they require explicit lifecycle management.

## Real iOS Example

```swift
struct HomeData {
    let profile: Profile
    let recommendations: [Product]
    let unreadCount: Int
}

func loadHome() async throws -> HomeData {
    async let profile = profileService.profile()
    async let recommendations = productService.recommendations()
    async let unreadCount = messageService.unreadCount()

    return try await HomeData(
        profile: profile,
        recommendations: recommendations,
        unreadCount: unreadCount
    )
}
```

This is faster than sequential loading because the requests are independent.

## Common Mistakes

- Using `Task {}` inside async functions instead of `async let`
- Forgetting to await async-let values
- Parallelizing dependent work
- Ignoring cancellation in child loops
- Mutating shared state from multiple child tasks
- Returning before unstructured tasks complete

## Modern Swift 6.x Notes

Swift's data-race safety model is built around structured reasoning. The compiler can help more when work and ownership are clear.

Swift 6.x concurrency encourages:

- Clear task trees
- Explicit actor isolation
- `Sendable` values crossing concurrency boundaries
- Avoiding shared mutable state between child tasks

## Junior Interview Answer

Structured concurrency means async child work is tied to a parent scope and must be completed, cancelled, or handled before the parent finishes.

## Mid-Level Interview Answer

I use `async let` for a fixed number of independent async operations and task groups for dynamic parallel work. Structured concurrency gives predictable error and cancellation behavior.

## Senior Interview Answer

Structured concurrency is the foundation for reasoning about async lifetime, cancellation, and data isolation. It prevents orphaned work, keeps parent-child relationships explicit, and works with Swift 6's data-race checks because values crossing task boundaries must be safe.

## Points To Remember

- Prefer structured concurrency inside async functions.
- Use `async let` for fixed independent work.
- Use task groups for dynamic work.
- Parent cancellation propagates to child tasks.
- Do not mutate unsafely shared state from child tasks.

## Practice

1. Convert three sequential independent requests into `async let`.
2. Explain why `Task {}` inside an async function may be a smell.
3. Describe what happens when one child task throws.
4. Design a structured load flow for a dashboard.
