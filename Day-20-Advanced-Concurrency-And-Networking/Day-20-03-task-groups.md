# Day 20: Task Groups

## What Task Groups Are

Task groups run a dynamic number of child tasks.

Use them when you do not know the exact number of tasks at compile time.

```swift
try await withThrowingTaskGroup(of: Image.self) { group in
    for url in urls {
        group.addTask {
            try await imageLoader.load(url)
        }
    }

    var images: [Image] = []
    for try await image in group {
        images.append(image)
    }
    return images
}
```

## Task Group vs `async let`

Use `async let`:

- Fixed small number of operations
- Clear local variables

Use task group:

- Dynamic number of operations
- Loop-based child tasks
- Need to collect results as they finish
- Need custom cancellation/limits

## Result Ordering

Task group results arrive in completion order, not input order.

If order matters:

```swift
group.addTask {
    let image = try await loader.load(url)
    return (index, image)
}
```

Then sort or place results by index.

## Limiting Concurrency

Do not start thousands of requests at once.

Pattern:

```swift
func loadAll(urls: [URL], limit: Int) async throws -> [UIImage] {
    var iterator = urls.enumerated().makeIterator()
    var output = Array<UIImage?>(repeating: nil, count: urls.count)

    try await withThrowingTaskGroup(of: (Int, UIImage).self) { group in
        for _ in 0..<min(limit, urls.count) {
            if let (index, url) = iterator.next() {
                group.addTask { (index, try await loadImage(url)) }
            }
        }

        while let (index, image) = try await group.next() {
            output[index] = image

            if let (nextIndex, nextURL) = iterator.next() {
                group.addTask { (nextIndex, try await loadImage(nextURL)) }
            }
        }
    }

    return output.compactMap { $0 }
}
```

This avoids overwhelming resources.

## Cancellation

If the parent task is cancelled, the group children are cancelled.

You can also cancel remaining child tasks:

```swift
group.cancelAll()
```

Useful when one successful result is enough.

## Real iOS Use Cases

- Load visible cell images
- Fetch details for many IDs
- Preload local files
- Process photo library assets
- Upload multiple attachments
- Run independent validation checks

## Common Mistakes

- Assuming results preserve input order
- Starting unbounded tasks
- Mutating shared state from child tasks
- Ignoring cancellation
- Doing UI updates from group child tasks
- Using task groups for two fixed calls instead of `async let`

## Modern Swift 6.x Notes

Task groups interact with `Sendable` because child tasks may run concurrently. Values captured into child tasks and returned from them should be safe to transfer.

## Interview Answer

Task groups are for dynamic structured parallelism. I use them when the number of child tasks is based on data, collect results safely, preserve ordering explicitly when needed, and limit concurrency for networking or CPU-heavy work.

## Practice

1. Load images for an array of URLs with a task group.
2. Preserve result order using indexes.
3. Add a concurrency limit.
4. Explain why child tasks should return values instead of mutating shared arrays.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Task Groups is not only a syntax topic. In production Swift, it affects async networking, parallelism, task groups, stream bridging, retry, timeout, and actor-isolated services. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while API clients, dashboards, image loaders, WebSocket streams, and token refresh coordination. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Task Groups in an app feature:

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

1. Write a minimal example that shows Task Groups correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Task Groups, but it shows the kind of production shape you should connect this topic to:

```swift
actor ImagePipeline {
    private var inFlight: [URL: Task<UIImage, Error>] = [:]

    func image(for url: URL) async throws -> UIImage {
        if let task = inFlight[url] {
            return try await task.value
        }

        let task = Task { try await downloadAndDecode(url) }
        inFlight[url] = task
        defer { inFlight[url] = nil }
        return try await task.value
    }
}
```

Advanced networking is not only making requests. It is also deduplicating work, respecting cancellation, protecting shared state, and keeping UI updates outside the transport layer.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Task Groups** is evaluated through this lens: advanced concurrency and networking is production coordination; senior engineers manage parallelism, retries, timeouts, streams, and shared service state. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a networking-concurrency artifact covering endpoint contract, cancellation, retry policy, timeout, parallelism limit, actor state, and UI handoff
Topic: Task Groups
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine building API clients, image pipelines, dashboard loaders, token refresh, WebSockets, and upload progress. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is unbounded parallel requests, duplicate token refreshes, retrying unsafe operations, and transport layers mutating UI state.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Is parallelism bounded?
- Are retries idempotent?
- Is shared service state actor-isolated?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Task Groups**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Task Groups** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Parallel Dashboard Load

```swift
func dashboard() async throws -> Dashboard {
    async let profile = api.profile()
    async let messages = api.messages()
    async let settings = api.settings()

    return try await Dashboard(
        profile: profile,
        messages: messages,
        settings: settings
    )
}
```

Independent network calls can run in parallel with `async let`.

### Example 2: Timeout Wrapper Usage

```swift
let products = try await withTimeout(.seconds(8)) {
    try await api.products()
}
```

Production networking needs timeouts and cancellation behavior, not only happy-path requests.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Load dynamic work in parallel

```swift
try await withThrowingTaskGroup(of: Image.self) { group in
    for url in urls {
        group.addTask { try await imageLoader.image(from: url) }
    }

    for try await image in group {
        images.append(image)
    }
}
```

Task groups are for a dynamic number of child tasks.

### Why This Fits Task Groups

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

