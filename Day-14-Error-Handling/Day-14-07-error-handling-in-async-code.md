# Day 14: Error Handling In Async Code

## Core Idea

Async throwing functions use `async throws`.

```swift
func fetchUser() async throws -> User {
    try await apiClient.user()
}
```

Call with:

```swift
do {
    let user = try await fetchUser()
    print(user)
} catch {
    print(error)
}
```

## Real iOS Use Cases

- URLSession
- Database operations
- Auth flows
- File loading

## Modern Swift 6.x Notes

Swift 6.2 improved approachable concurrency, async debugging, and task context visibility. Error handling in async code should also respect cancellation.

```swift
try Task.checkCancellation()
```

## Interview Levels

Junior: Use `try await` for async throwing calls.

Senior: Async error handling should distinguish cancellation, retryable failures, user-facing failures, and programmer errors.

## Quick Notes

- `async throws`.
- Call with `try await`.
- Handle cancellation.
- Map errors for UI.

## Interview Depth

Junior answer: Use `try await` when calling an async function that can throw.

Mid-level answer: Async throwing code is handled with `do-catch`, just like synchronous throwing code, but calls include `await`.

Senior answer: Async error handling should distinguish user cancellation, network failure, decoding failure, and business failure. Cancellation is often not an error to show to the user.

iOS use case:

```swift
func loadProfile() async {
    do {
        state = .loading
        state = .loaded(try await service.profile())
    } catch is CancellationError {
        state = .idle
    } catch {
        state = .failed("Could not load profile")
    }
}
```

Common mistakes: ignoring cancellation, updating UI off main actor, retrying non-retryable errors.

Practice: write async throws loader, catch cancellation, map error to state enum.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Error Handling In Async Code is not only a syntax topic. In production Swift, it affects recoverability, user-facing failures, typed modeling, async propagation, and observability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while network errors, validation failures, retry flows, and Result-based APIs. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Error Handling In Async Code in an app feature:

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

1. Write a minimal example that shows Error Handling In Async Code correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Error Handling In Async Code, but it shows the kind of production shape you should connect this topic to:

```swift
enum LoginError: Error, Equatable {
    case invalidCredentials
    case lockedAccount
    case networkUnavailable
}

func userMessage(for error: LoginError) -> String {
    switch error {
    case .invalidCredentials: return "Check your email and password."
    case .lockedAccount: return "Your account is locked."
    case .networkUnavailable: return "Check your connection."
    }
}
```

Good error handling separates technical failure from user messaging. The app should know what can be retried, what needs user action, and what should be logged.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Error Handling In Async Code** is evaluated through this lens: error handling is recovery design; senior engineers model what can fail, who can recover, and what the user sees. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: an error taxonomy separating validation, transport, decoding, authorization, cancellation, retryable, and fatal errors
Topic: Error Handling In Async Code
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine reviewing login, payment, sync, networking, decoding, and async task failures. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is catch-all errors, force try, user-hostile messages, and retrying operations that should not be retried.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Can the caller recover?
- Is cancellation separate?
- Is this error useful for logs and for users?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Error Handling In Async Code**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Error Handling In Async Code** to code you might write in a SwiftUI/UIKit feature.

### Example 1: User-Friendly Error Mapping

```swift
enum ProfileError: Error {
    case notFound
    case offline
    case server
}

func message(for error: ProfileError) -> String {
    switch error {
    case .notFound: return "Profile not found."
    case .offline: return "Check your internet connection."
    case .server: return "Please try again later."
    }
}
```

Errors should help the app decide what to show or retry.

### Example 2: Async Throwing Call

```swift
func loadProfile() async {
    do {
        let profile = try await service.profile()
        print(profile)
    } catch {
        print("Show error state")
    }
}
```

`do-catch` keeps success and failure paths explicit.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Await an async service result

```swift
func refresh() async {
    do {
        products = try await service.products()
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

Async code should keep success, failure, and UI update points clear.

### Why This Fits Error Handling In Async Code

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

