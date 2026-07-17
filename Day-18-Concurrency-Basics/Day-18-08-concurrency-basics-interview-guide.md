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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Concurrency Basics Interview Guide is not only a syntax topic. In production Swift, it affects suspension, task lifetime, structured work, cancellation, priority, streams, and UI isolation. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while screen loading, search debounce, progress streams, and main-actor view models. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Concurrency Basics Interview Guide in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.
- Always separate task lifetime from object lifetime; a task can outlive the screen that started it.
- State crossing a concurrency boundary should be immutable, actor-isolated, or clearly Sendable.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Concurrency Basics Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Concurrency Basics Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
@MainActor
final class SearchViewModel: ObservableObject {
    @Published private(set) var results: [SearchResult] = []
    private var searchTask: Task<Void, Never>?

    func search(_ query: String) {
        searchTask?.cancel()
        searchTask = Task {
            do {
                try await Task.sleep(for: .milliseconds(300))
                let results = try await service.search(query)
                try Task.checkCancellation()
                self.results = results
            } catch is CancellationError {
                return
            } catch {
                self.results = []
            }
        }
    }
}
```

Concurrency basics become real in screens like search: debounce, cancel stale work, await the service, and update UI state on the main actor.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

