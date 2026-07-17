# Day 18: Tasks

## What A Task Is

A task is a unit of asynchronous work managed by Swift concurrency.

```swift
Task {
    let user = try await service.loadUser()
    print(user)
}
```

Tasks let synchronous code start async work.

## Task Is Not The Same As Thread

A task is lightweight work. A thread is an operating-system execution resource.

Many tasks can share a smaller number of threads.

Interview point:

Swift concurrency is task-based, not "one task equals one thread."

## Unstructured Task

```swift
Task {
    await viewModel.load()
}
```

This creates an unstructured task. It has a relationship to priority and actor context, but not a strict parent-child lifetime like structured concurrency.

Use unstructured tasks when you must bridge from sync code to async code.

## SwiftUI `.task`

SwiftUI gives a lifecycle-aware task modifier.

```swift
struct ProfileView: View {
    @StateObject private var viewModel = ProfileViewModel()

    var body: some View {
        content
            .task {
                await viewModel.load()
            }
    }
}
```

SwiftUI can cancel the task when the view disappears or when the `.task(id:)` value changes.

```swift
.task(id: query) {
    await search(query)
}
```

This is useful for search screens.

## Task Handles

```swift
let task = Task {
    try await service.refresh()
}

task.cancel()
```

A task handle can be used to:

- Await the result
- Cancel the task
- Store and cancel a long-running operation

```swift
let task = Task { try await service.load() }
let value = try await task.value
```

## Detached Tasks

```swift
Task.detached {
    await performBackgroundWork()
}
```

Detached tasks do not inherit the current actor context in the same way.

Use them rarely.

Common danger:

```swift
@MainActor
final class ViewModel {
    var title = ""

    func load() {
        Task.detached {
            // Cannot safely mutate main-actor state here.
            // self.title = "Done"
        }
    }
}
```

Prefer structured tasks or normal `Task` unless you intentionally need detached work.

## Task In UIKit

```swift
final class ProfileViewController: UIViewController {
    private var loadTask: Task<Void, Never>?

    override func viewDidLoad() {
        super.viewDidLoad()

        loadTask = Task { [weak self] in
            await self?.load()
        }
    }

    deinit {
        loadTask?.cancel()
    }

    @MainActor
    private func load() async {
        // Update UI safely.
    }
}
```

UIKit does not automatically cancel your stored tasks. You own that lifecycle.

## Task Result Types

```swift
Task<String, Error>
Task<Void, Never>
```

The first generic type is the success value.

The second generic type is the failure type.

Most fire-and-forget UI tasks are `Task<Void, Never>` because they handle errors internally.

## Naming Tasks In Modern Swift

Swift 6.2 improved async debugging and task context. Named tasks can appear in debugging/profiling tools.

```swift
Task(name: "Profile refresh") {
    await viewModel.refresh()
}
```

Use names for important long-running tasks, especially in complex flows.

## Common Mistakes

- Creating a task when the function itself could be async
- Forgetting to cancel stored tasks
- Using detached tasks for normal UI work
- Capturing `self` strongly in a long-lived task
- Ignoring task priority and cancellation
- Starting duplicate tasks on repeated view appearances

## Junior Interview Answer

A task is a unit of async work. It lets Swift run asynchronous operations and suspend/resume them.

## Mid-Level Interview Answer

I use `Task` to start async work from synchronous contexts like UIKit lifecycle methods. In SwiftUI, I prefer `.task` because it is tied to the view lifecycle. I store task handles when I need cancellation.

## Senior Interview Answer

Tasks are the runtime units of Swift concurrency. I avoid unstructured tasks unless bridging from sync to async, prefer structured concurrency for child work, keep task lifetimes tied to UI or object lifetimes, and use cancellation cooperatively. I avoid detached tasks unless I explicitly want to leave actor/task context.

## Points To Remember

- A task is not a thread.
- `.task` is usually better than manual `Task` in SwiftUI.
- Store task handles when you need cancellation.
- Detached tasks are advanced and easy to misuse.
- Named tasks help modern debugging.

## Practice

1. Start a task from a UIKit `viewDidLoad`.
2. Cancel a task in `deinit`.
3. Replace a manual SwiftUI `Task` with `.task`.
4. Explain when `Task.detached` is risky.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Tasks is not only a syntax topic. In production Swift, it affects suspension, task lifetime, structured work, cancellation, priority, streams, and UI isolation. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Tasks in an app feature:

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

1. Write a minimal example that shows Tasks correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Tasks, but it shows the kind of production shape you should connect this topic to:

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

