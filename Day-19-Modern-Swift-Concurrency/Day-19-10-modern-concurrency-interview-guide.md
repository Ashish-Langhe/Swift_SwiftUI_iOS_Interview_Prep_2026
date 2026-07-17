# Day 19: Modern Swift Concurrency Interview Guide

## One-Minute Interview Answer

Modern Swift concurrency is about data-race safety. Actors protect mutable state, global actors like `MainActor` isolate UI, `Sendable` controls values crossing concurrency boundaries, and strict concurrency checks catch unsafe access at compile time. Swift 6 made data-race safety a major language-mode feature, while Swift 6.2 made concurrency more approachable with default actor isolation, caller-context async behavior, explicit `@concurrent`, named tasks, and better debugging.

## Junior Questions

What is an actor?

An actor is a reference type that protects mutable state by serializing access.

What is `MainActor`?

The global actor used for UI-related work.

What is `Sendable`?

A protocol indicating a type can be safely passed across concurrency boundaries.

## Mid-Level Questions

When do you use an actor?

For shared mutable state such as caches, token stores, download coordinators, and analytics buffers.

When do you use `@MainActor`?

For UI state and UI-updating methods.

What is strict concurrency?

Compiler checking for potential data races, unsafe actor access, and non-sendable sharing.

## Senior Questions

What is actor reentrancy?

An actor may process other work while a method is suspended at `await`. Do not assume actor state remains unchanged across suspension points.

How do you migrate to Swift 6 concurrency?

Classify state ownership, mark UI state `@MainActor`, convert shared mutable services to actors or safe synchronization, make transferred values `Sendable`, and avoid unsafe escape hatches unless justified.

What is the value of Swift 6.2 approachable concurrency?

It reduces boilerplate for common UI-style code while making intentional concurrent execution explicit with `@concurrent`.

## Scenario: Token Refresh

Strong design:

```swift
actor AuthTokenStore {
    private var token: String?
    private var refreshTask: Task<String, Error>?

    func validToken() async throws -> String {
        if let token { return token }

        if let refreshTask {
            return try await refreshTask.value
        }

        let task = Task { try await requestNewToken() }
        refreshTask = task

        do {
            let token = try await task.value
            self.token = token
            refreshTask = nil
            return token
        } catch {
            refreshTask = nil
            throw error
        }
    }
}
```

Why this is senior:

- Shared mutable token state is actor-isolated.
- Duplicate refreshes are coalesced.
- Errors reset state.
- Callers use async safely.

## Scenario: UI ViewModel

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published private(set) var state: State = .idle

    func load() async {
        state = .loading
        do {
            let profile = try await service.profile()
            state = .loaded(profile)
        } catch is CancellationError {
            return
        } catch {
            state = .failed(error.localizedDescription)
        }
    }
}
```

Strong points:

- UI state is main-actor isolated.
- Errors and cancellation are separated.
- External mutation is controlled.

## Common Traps

- "Actor means no race conditions" is incomplete.
- `@MainActor` everywhere can hide performance problems.
- `@unchecked Sendable` is not a real fix by itself.
- `Task.detached` should not be a default migration tool.
- Public protocol isolation affects all conformers.
- `private` is not concurrency isolation.

## Latest Swift 6.x Notes

- Swift 6 language mode strengthens data-race safety.
- Swift 6.2 adds default actor isolation settings.
- Swift 6.2 introduces explicit `@concurrent` for work that should run concurrently.
- Swift 6.2 improves async debugging and task context.
- `Sendable` and actor isolation remain core concepts.

## Interview Closing Answer

I design Swift concurrency around ownership. UI state belongs to the main actor, shared mutable services belong behind actors or synchronization, data crossing boundaries should be immutable and `Sendable`, and long-running flows should respect cancellation. Swift 6.x gives compiler support for these choices, but the architecture still needs clear boundaries.

## Practice Prompts

1. Explain actor reentrancy with a bank account example.
2. Migrate a mutable singleton cache to an actor.
3. Decide whether a model should be `Sendable`.
4. Explain Swift 6.2 approachable concurrency in two minutes.
5. Design a public async API with actor and sendability in mind.
