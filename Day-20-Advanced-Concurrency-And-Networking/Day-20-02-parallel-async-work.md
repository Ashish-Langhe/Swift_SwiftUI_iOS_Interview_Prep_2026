# Day 20: Parallel Async Work

## What Parallel Async Work Means

Parallel async work means starting independent operations so they can make progress at the same time.

Sequential:

```swift
let profile = try await api.profile()
let posts = try await api.posts()
let settings = try await api.settings()
```

Parallel:

```swift
async let profile = api.profile()
async let posts = api.posts()
async let settings = api.settings()

let data = try await DashboardData(
    profile: profile,
    posts: posts,
    settings: settings
)
```

## When To Parallelize

Parallelize when:

- Operations are independent
- Results are all needed
- Work is mostly waiting on I/O
- The server can handle concurrent requests
- Cancellation/error behavior is acceptable

Do not parallelize when one result depends on another.

## Real iOS Example

```swift
func loadHome() async throws -> HomeData {
    async let profile = profileService.profile()
    async let feed = feedService.feed()
    async let promos = marketingService.promotions()

    return try await HomeData(
        profile: profile,
        feed: feed,
        promotions: promos
    )
}
```

This reduces perceived screen load time.

## Avoid Shared Mutable State

Bad:

```swift
var results: [Item] = []

async let a: Void = results.append(contentsOf: serviceA.items())
async let b: Void = results.append(contentsOf: serviceB.items())
```

Better:

```swift
async let a = serviceA.items()
async let b = serviceB.items()

let (itemsA, itemsB) = try await (a, b)
let combined = itemsA + itemsB
```

Each task returns values. Combine them after awaiting.

## Error Behavior

If one child operation throws, the overall operation throws and sibling work may be cancelled.

This is often what you want for all-or-nothing screens.

For partial success, model it explicitly.

```swift
func capture<T>(_ operation: () async throws -> T) async -> Result<T, Error> {
    do {
        return .success(try await operation())
    } catch {
        return .failure(error)
    }
}

async let profile = capture { try await api.profile() }
async let recommendations = capture { try await api.recommendations() }
```

This makes partial failure explicit instead of accidentally hiding it.

## Performance Considerations

Parallelism is not free.

Watch for:

- Too many network requests
- Server rate limits
- Battery impact
- Memory pressure
- CPU contention
- Priority inversion

Use task groups with limits for large collections.

## Common Mistakes

- Parallelizing dependent work
- Mutating shared arrays from child tasks
- Starting unlimited network requests
- Ignoring cancellation
- Returning partial data accidentally
- Making the UI wait for non-critical work

## Modern Swift 6.x Notes

Swift 6's data-race checks make unsafe shared mutation across child tasks more visible. Prefer immutable results and post-processing after `await`.

## Interview Answer

I parallelize independent async work using `async let` for a fixed small number of operations and task groups for dynamic collections. I avoid shared mutable state inside child tasks and design error behavior intentionally.

## Practice

1. Convert three independent sequential calls to `async let`.
2. Explain why appending to one shared array from child tasks is unsafe.
3. Design partial-success loading for a dashboard.
4. Identify when parallelism could hurt performance.
