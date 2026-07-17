# Day 18: `async` And `await`

## What Problem Concurrency Solves

Concurrency lets a program make progress on long-running work without blocking important execution, especially the UI.

Common iOS examples:

- Download JSON without freezing the screen
- Load images while a list scrolls
- Save files in the background
- Wait for location, camera, or HealthKit permissions
- Run multiple independent requests at the same time

Before Swift concurrency, iOS code often used completion handlers:

```swift
func loadUser(id: String, completion: @escaping (Result<User, Error>) -> Void) {
    api.get("/users/\(id)") { result in
        completion(result)
    }
}
```

With async/await:

```swift
func loadUser(id: String) async throws -> User {
    try await api.get("/users/\(id)")
}
```

The second version reads like normal sequential code, but it can suspend while waiting.

## What `async` Means

`async` means a function can suspend.

```swift
func fetchProfile() async throws -> Profile {
    let url = URL(string: "https://example.com/profile")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(Profile.self, from: data)
}
```

An async function does not necessarily run on a new thread. It means the function may pause and let other work run while it waits.

## What `await` Means

`await` marks a suspension point.

```swift
let profile = try await fetchProfile()
```

At an `await`:

- The current task can suspend.
- The thread is not blocked while waiting.
- Other work may run.
- When the awaited operation finishes, the task resumes.

Important interview line:

`await` is not "wait and block the thread"; it is "this task may suspend here."

## Beginner Mental Model

Think of async code as a bookmark.

```text
Start fetch
Put bookmark here while network is running
Let system do other useful work
Resume from bookmark when result arrives
```

This is why async/await is excellent for network and file operations.

## `async throws`

Most real iOS async APIs can fail, so they are often both `async` and `throws`.

```swift
func products() async throws -> [Product] {
    let (data, response) = try await URLSession.shared.data(from: productsURL)
    try validate(response)
    return try JSONDecoder().decode([Product].self, from: data)
}
```

Call site:

```swift
do {
    let products = try await service.products()
    self.products = products
} catch {
    self.errorMessage = error.localizedDescription
}
```

## Calling Async Code From Sync Code

Synchronous code cannot directly call async code.

```swift
func viewDidLoad() {
    // let user = try await loadUser() // Not allowed in sync function.
}
```

Create a task:

```swift
func viewDidLoad() {
    Task {
        let user = try await loadUser()
        print(user)
    }
}
```

SwiftUI:

```swift
.task {
    await viewModel.load()
}
```

## Sequential Async Work

```swift
let user = try await api.user(id: id)
let orders = try await api.orders(userID: user.id)
```

This is sequential because `orders` depends on `user`.

Do not parallelize dependent work.

## Independent Async Work

If values do not depend on each other, use `async let` or task groups later in this curriculum.

```swift
async let profile = api.profile()
async let settings = api.settings()

let screen = try await ProfileScreenData(
    profile: profile,
    settings: settings
)
```

## Real iOS ViewModel Example

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published private(set) var state: State = .idle

    enum State {
        case idle
        case loading
        case loaded(Profile)
        case failed(String)
    }

    private let service: ProfileService

    init(service: ProfileService) {
        self.service = service
    }

    func load() async {
        state = .loading

        do {
            let profile = try await service.profile()
            state = .loaded(profile)
        } catch {
            state = .failed(error.localizedDescription)
        }
    }
}
```

Because the view model is `@MainActor`, its UI state updates are isolated to the main actor.

## Common Mistakes

- Thinking `async` means "background thread"
- Forgetting `try` with async throwing functions
- Calling async functions from sync functions without a task
- Doing CPU-heavy work on the main actor
- Updating UI from non-main actor code
- Running independent requests sequentially by accident

## Modern Swift 6.x Notes

Swift 6 made data-race safety a major language goal. Swift 6.2 made concurrency more approachable:

- Default actor isolation can make unannotated code main-actor isolated for selected targets.
- Nonisolated async functions can opt into caller-context behavior through the Swift 6.2 upcoming feature.
- `@concurrent` marks async work that should leave actor isolation and run concurrently.
- Async debugging improved with task context and named tasks.

## Junior Interview Answer

`async` marks a function that can suspend. `await` marks where the current task may suspend while waiting for the async result.

## Mid-Level Interview Answer

Async/await replaces callback-heavy code with structured, readable code. `await` does not block the thread; it suspends the task. Errors are modeled with `async throws`, and UI state updates should be isolated to the main actor.

## Senior Interview Answer

Async/await is a suspension model, not a thread API. I use it to express asynchronous dependencies clearly, keep dependent work sequential, parallelize independent work with structured primitives, propagate cancellation, and maintain actor isolation for shared mutable state. In Swift 6.x, concurrency annotations also become part of API correctness and migration strategy.

## Points To Remember

- `async` means can suspend.
- `await` marks a possible suspension point.
- Suspension is not blocking.
- Async code still needs error handling.
- UI updates belong on the main actor.
- Independent async work should not be accidentally serialized.

## Practice

1. Convert a completion-handler network method to async/await.
2. Explain why `await` does not freeze the UI.
3. Write an async throwing service method.
4. Identify which awaits in a flow are dependent and which can run in parallel.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

async And await is not only a syntax topic. In production Swift, it affects suspension, task lifetime, structured work, cancellation, priority, streams, and UI isolation. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying async And await in an app feature:

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

1. Write a minimal example that shows async And await correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use async And await, but it shows the kind of production shape you should connect this topic to:

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

