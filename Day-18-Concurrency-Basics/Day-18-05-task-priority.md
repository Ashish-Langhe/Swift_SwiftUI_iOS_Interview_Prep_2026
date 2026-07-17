# Day 18: Task Priority

## What Task Priority Means

Task priority is a hint to the scheduler about how important work is.

```swift
Task(priority: .userInitiated) {
    await loadVisibleScreenData()
}
```

Priority helps Swift decide how to schedule work, but it is not a strict guarantee.

## Common Priorities

```swift
.high
.userInitiated
.medium
.low
.utility
.background
```

Practical iOS mapping:

- User tapped a button and waits for result: `.userInitiated`
- Background sync: `.background`
- Cache warmup: `.utility`
- Non-urgent analytics upload: `.background`
- Visible image loading: often `.userInitiated` or inherited

## Beginner Mental Model

Priority answers:

```text
If the system has many tasks, which one should matter more?
```

It does not mean:

```text
Run this task immediately no matter what.
```

## Priority Inheritance

Child tasks can inherit priority from parent context.

```swift
Task(priority: .userInitiated) {
    async let profile = api.profile()
    async let feed = api.feed()
    _ = try await (profile, feed)
}
```

The child work is associated with the parent priority.

## Checking Current Priority

```swift
let priority = Task.currentPriority
```

This can help debug scheduling choices.

## Real iOS Examples

### User-Initiated Refresh

```swift
Task(priority: .userInitiated) {
    await viewModel.refresh()
}
```

The user is waiting for this work.

### Background Prefetch

```swift
Task(priority: .utility) {
    await imageCache.prefetch(urls)
}
```

The user benefits later, but it should not compete too aggressively with visible UI work.

### Analytics

```swift
Task(priority: .background) {
    await analytics.flush()
}
```

Analytics should not block user experience.

## Priority Is Not A Fix For Bad Work

If CPU-heavy work runs on the main actor, lowering priority does not make UI updates safe.

Bad:

```swift
@MainActor
func decodeHugeImage() {
    // Heavy CPU work on main actor.
}
```

Better:

```swift
func decodeHugeImage() async throws -> UIImage {
    try await imageDecoder.decode()
}
```

Then update UI on the main actor.

## Priority Inversion

Priority inversion happens when high-priority work waits on lower-priority work.

Example:

```text
High-priority UI refresh waits for low-priority cache update
```

Senior engineers watch for this in shared services, locks, actors, and queues.

## Common Mistakes

- Treating priority as a hard guarantee
- Marking everything `.high`
- Using priority to hide main-thread blocking
- Starting background tasks that update UI late
- Ignoring cancellation with background work
- Creating priority inversion through shared resources

## Modern Swift 6.x Notes

Swift 6.2 improved async debugging, including better task context. This makes it easier to inspect which task is running and reason about priority-related behavior.

Named tasks plus meaningful priority can make complex concurrency bugs much easier to diagnose.

```swift
Task(name: "Warm image cache", priority: .utility) {
    await cache.warm()
}
```

## Junior Interview Answer

Task priority is a scheduling hint that tells Swift how important a task is.

## Mid-Level Interview Answer

I use `.userInitiated` for work the user is waiting on, `.utility` for useful background work, and `.background` for non-urgent tasks like analytics.

## Senior Interview Answer

Task priority is cooperative scheduling metadata, not a correctness tool. I use it to express intent, but I still design proper actor isolation, cancellation, and workload placement. I also watch for priority inversion when high-priority flows depend on shared lower-priority resources.

## Points To Remember

- Priority is a hint.
- Do not mark everything high.
- Priority does not fix main actor blocking.
- Child tasks can inherit priority.
- Consider cancellation with low-priority work.

## Practice

1. Assign priorities to image prefetch, screen refresh, and analytics upload.
2. Explain priority inversion with an actor or cache.
3. Add a named low-priority task for cache warming.
4. Explain why priority is not the same as thread control.
