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
