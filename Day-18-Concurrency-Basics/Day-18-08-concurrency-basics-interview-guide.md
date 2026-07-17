# Day 18: Concurrency Basics Interview Guide

## One-Minute Interview Answer

Swift concurrency uses async/await, tasks, structured concurrency, cancellation, priority, async sequences, and actors to write asynchronous code safely. `async` functions can suspend, `await` marks suspension points, tasks are units of async work, structured concurrency keeps child work tied to parent scopes, cancellation is cooperative, async sequences represent streams of values over time, and the main actor protects UI state. Swift 6.x adds stronger data-race safety, and Swift 6.2 makes concurrency more approachable with default actor isolation and `@concurrent`.

## Must-Know Vocabulary

- `async`: function can suspend
- `await`: possible suspension point
- `Task`: unit of async work
- Structured concurrency: parent-child task lifetime
- Cancellation: cooperative stop signal
- Priority: scheduling hint
- `AsyncSequence`: async values over time
- `@MainActor`: UI/global actor isolation

## Junior-Level Questions

What does `async` mean?

It means a function can suspend while waiting for asynchronous work.

What does `await` mean?

It marks a point where the current task may suspend.

Does `await` block the thread?

No. It suspends the task, allowing the system to run other work.

What is a task?

A task is a unit of asynchronous work managed by Swift concurrency.

## Mid-Level Questions

When do you use `Task {}`?

When starting async work from a synchronous context, such as UIKit lifecycle code. In SwiftUI, prefer `.task` when tied to view lifecycle.

What is structured concurrency?

Child async work belongs to a parent scope. The parent waits for child work to complete, throw, or be cancelled.

What is cancellation?

Cancellation is a cooperative signal. Code should check cancellation and exit safely when the result is no longer needed.

What is an async sequence?

An async sequence produces values over time and is consumed with `for await`.

## Senior-Level Questions

How do you decide between sequential, `async let`, and task groups?

Sequential when each step depends on the previous result. `async let` for a small fixed number of independent operations. Task groups for a dynamic number of child operations.

How do you prevent stale UI updates?

Tie work to lifecycle, cancel old tasks, check cancellation before applying results, and update UI on the main actor.

What does Swift 6 data-race safety change?

Swift 6 language mode can diagnose data-race risks as errors. It pushes code toward actor isolation, `Sendable`, structured tasks, and safer transfer of mutable state.

What changed in Swift 6.2?

Swift 6.2 made concurrency more approachable with default actor isolation options, caller-context behavior for some async functions through upcoming features, `@concurrent` for explicitly concurrent work, and improved async debugging.

## Real iOS Scenario Answer

Question:

How would you load a dashboard with profile, feed, and unread count?

Strong answer:

```swift
func loadDashboard() async {
    state = .loading

    do {
        async let profile = profileService.profile()
        async let feed = feedService.feed()
        async let unreadCount = messageService.unreadCount()

        let data = try await DashboardData(
            profile: profile,
            feed: feed,
            unreadCount: unreadCount
        )

        try Task.checkCancellation()
        state = .loaded(data)
    } catch is CancellationError {
        return
    } catch {
        state = .failed(error.localizedDescription)
    }
}
```

Why this is strong:

- Independent requests run in parallel.
- Errors are handled.
- Cancellation is respected.
- UI state is updated in one place.
- It fits a main-actor view model.

## Common Interview Traps

- Saying async means background thread
- Using `Task.detached` for UI work
- Forgetting cancellation in search
- Updating UI outside main actor
- Making all code `@MainActor`
- Parallelizing dependent work
- Treating task priority as a guarantee
- Using async sequence for one result

## Debugging Checklist

1. Is the task still alive?
2. Was it cancelled?
3. Is UI state main-actor isolated?
4. Are independent awaits accidentally sequential?
5. Is there a long operation blocking the main actor?
6. Are errors and cancellation separated?
7. Are child tasks structured?

## Points To Remember

- Suspension is not blocking.
- Tasks are not threads.
- Prefer structured concurrency.
- Cancellation must be cooperatively handled.
- Main actor is for UI state, not heavy work.
- Async sequence is for streams.

## Practice Prompts

1. Convert a callback API to async/await.
2. Build a cancellable search screen.
3. Load three independent resources with `async let`.
4. Consume a notification stream with `for await`.
5. Explain main actor vs main thread in one minute.
