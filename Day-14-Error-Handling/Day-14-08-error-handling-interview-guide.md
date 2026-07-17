# Day 14: Error Handling Interview Guide

## One-Minute Interview Answer

Swift error handling uses types conforming to `Error`, throwing functions marked with `throws`, and calls marked with `try`. `do-catch` handles errors, `try?` converts errors to nil, and `try!` crashes if an error occurs. `Result` stores success or failure as a value. Modern Swift also supports typed throws syntax, and async code uses `async throws` with `try await`.

## Modern Swift 6.x Notes

Typed throws are documented as `throws(MyError)`. Swift 6.2 improved async debugging and approachable concurrency, which matters when handling errors in async tasks. Swift Testing also gained exit testing and richer attachments in Swift 6.2/6.3, useful for testing failure paths.

## Common Traps

- Using `try?` and losing important error details.
- Using `try!` with dynamic data.
- Catching all errors and doing nothing.
- Showing raw technical errors to users.
- Ignoring cancellation in async work.

## Topic-By-Topic Deep Dive

### `Error` Protocol

Swift errors are values that conform to `Error`.

```swift
enum PaymentError: Error {
    case cardDeclined
    case insufficientFunds
    case networkUnavailable
}
```

Senior answer:

Use error cases that match recovery decisions. If the UI handles two failures the same way, they may not need separate public cases. If logging/debugging needs more detail, use associated values.

### `throw`, `throws`, `try`

```swift
func charge(amount: Decimal) throws {
    guard amount > 0 else {
        throw PaymentError.insufficientFunds
    }
}
```

`throws` is part of the function contract. `try` makes failure visible at the call site.

### `try?` And `try!`

`try?` is good when only success/failure matters:

```swift
let image = try? loadCachedImage()
```

`try!` is only appropriate when failure means programmer error or impossible invariant failure:

```swift
let regex = try! Regex("[0-9]+")
```

### `do-catch`

```swift
do {
    try await viewModel.login()
} catch AuthError.invalidCredentials {
    message = "Invalid email or password"
} catch {
    message = "Something went wrong"
}
```

Senior rule:

Catch specifically when recovery differs. Avoid one giant catch that loses meaning.

### Typed Error Modeling

Typed throws:

```swift
func parseToken(_ raw: String) throws(AuthError) -> Token {
    guard !raw.isEmpty else {
        throw .missingToken
    }
    return Token(rawValue: raw)
}
```

Senior answer:

Typed throws are useful when the failure domain is stable and meaningful. Untyped `throws` remains practical when composing many possible error sources.

### `Result`

Use `Result` when success/failure must be stored or passed as a value.

```swift
let result: Result<User, AuthError>
```

With async/await, direct `async throws` APIs are often cleaner than callback `Result`.

### Async Error Handling

```swift
func load() async {
    do {
        state = .loading
        let user = try await service.fetchUser()
        state = .loaded(user)
    } catch is CancellationError {
        state = .idle
    } catch {
        state = .failed("Could not load user")
    }
}
```

Senior answer:

Cancellation is not the same as failure. Treat it separately when user intent or task lifecycle matters.

## Production Decision Table

| Situation | Best Tool |
| --- | --- |
| Recoverable failure | `throws` |
| Failure as stored value | `Result` |
| Ignore details safely | `try?` |
| Programmer invariant | rare `try!` |
| Stable failure domain | typed throws / error enum |
| Async operation failure | `async throws` |

## Final Revision

- `Error` marks error types.
- `throw` emits.
- `throws` declares.
- `try` calls.
- `do-catch` handles.
- `Result` stores outcome.
- `async throws` combines concurrency and failure.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Error Handling Interview Guide is not only a syntax topic. In production Swift, it affects recoverability, user-facing failures, typed modeling, async propagation, and observability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Error Handling Interview Guide in an app feature:

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

1. Write a minimal example that shows Error Handling Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Error Handling Interview Guide, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Error Handling Interview Guide** is evaluated through this lens: error handling is recovery design; senior engineers model what can fail, who can recover, and what the user sees. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Error Handling Interview Guide
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

For **Error Handling Interview Guide**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

