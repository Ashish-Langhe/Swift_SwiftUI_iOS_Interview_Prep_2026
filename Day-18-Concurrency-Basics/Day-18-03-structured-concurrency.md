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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Structured Concurrency is not only a syntax topic. In production Swift, it affects suspension, task lifetime, structured work, cancellation, priority, streams, and UI isolation. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Structured Concurrency in an app feature:

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

1. Write a minimal example that shows Structured Concurrency correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Structured Concurrency, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Structured Concurrency** is evaluated through this lens: basic concurrency is lifecycle design; senior engineers pair every async operation with ownership, cancellation, and UI isolation. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Structured Concurrency
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

For **Structured Concurrency**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

