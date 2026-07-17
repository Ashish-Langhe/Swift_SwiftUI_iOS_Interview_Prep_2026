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
