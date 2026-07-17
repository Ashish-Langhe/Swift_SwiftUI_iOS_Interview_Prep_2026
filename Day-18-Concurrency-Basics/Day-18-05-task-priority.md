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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Task Priority is not only a syntax topic. In production Swift, it affects suspension, task lifetime, structured work, cancellation, priority, streams, and UI isolation. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Task Priority in an app feature:

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

1. Write a minimal example that shows Task Priority correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Task Priority, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Task Priority** is evaluated through this lens: basic concurrency is lifecycle design; senior engineers pair every async operation with ownership, cancellation, and UI isolation. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a task-lifecycle artifact showing task owner, cancellation trigger, actor context, priority, and result application point
Topic: Task Priority
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine building search, refresh, progress, infinite scroll, async loading, and main-actor view models. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is orphaned tasks, stale UI updates, accidental main-actor heavy work, and swallowed cancellation.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Who cancels this task?
- Where does it resume?
- Can stale results update the UI?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Task Priority**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

