# Day 3: Default Parameters

## Core Idea

Default parameters let a function provide default values for arguments.

```swift
func greet(name: String, punctuation: String = "!") {
    print("Hello, \(name)\(punctuation)")
}

greet(name: "Aarav")
greet(name: "Aarav", punctuation: ".")
```

If the caller does not pass `punctuation`, Swift uses `"!"`.

## Why Default Parameters Matter

They reduce overloads and keep common calls simple.

Without default parameter:

```swift
func fetchUsers(page: Int, pageSize: Int) { }
func fetchUsers(page: Int) {
    fetchUsers(page: page, pageSize: 20)
}
```

With default parameter:

```swift
func fetchUsers(page: Int, pageSize: Int = 20) { }
```

## Real iOS Use Cases

### Animation

```swift
func showToast(message: String, duration: TimeInterval = 2.0) {
    print("Show \(message) for \(duration) seconds")
}
```

### Networking

```swift
func request(
    path: String,
    method: String = "GET",
    timeout: TimeInterval = 30
) {
    print("\(method) \(path), timeout: \(timeout)")
}
```

### Analytics

```swift
func track(event name: String, properties: [String: String] = [:]) {
    print("Event: \(name), properties: \(properties)")
}
```

## Good Defaults

Good defaults should represent the common case.

```swift
func loadImage(from url: URL, useCache: Bool = true) { }
```

The common behavior is usually to use cache.

## Bad Defaults

Avoid defaults that hide important decisions.

```swift
func deleteUser(id: String, confirm: Bool = true) { }
```

Deletion is important enough that confirmation should probably be explicit.

## Default Parameters And API Design

Default parameters are useful, but too many can make a function unclear.

```swift
func createButton(
    title: String,
    color: String = "blue",
    isEnabled: Bool = true,
    showsIcon: Bool = false,
    cornerRadius: Double = 8
) { }
```

This may be fine for small utilities, but if configuration grows, consider a configuration struct.

```swift
struct ButtonConfiguration {
    var color: String = "blue"
    var isEnabled: Bool = true
    var showsIcon: Bool = false
    var cornerRadius: Double = 8
}
```

## Junior-Level Interview Answer

Default parameters are values used when the caller does not provide an argument.

## Mid-Level Interview Answer

Default parameters simplify common function calls and reduce the need for multiple overloads. They should represent sensible defaults.

## Senior-Level Interview Answer

Default parameters are API ergonomics. They make common cases concise, but they can hide important behavior if overused. For critical actions, security-sensitive operations, or domain decisions, I prefer explicit arguments. If a function has many defaults, a configuration type may be clearer and easier to evolve.

## Quick Interview Notes

- Default values are declared in the parameter list.
- Callers can omit arguments with defaults.
- Defaults should represent common safe behavior.
- Too many defaults can signal a configuration object is needed.
- Avoid defaults for dangerous or business-critical behavior.

## Practice Questions

1. What is a default parameter?
2. Why are default parameters useful?
3. When can default parameters be harmful?
4. How can default parameters reduce overloads?
5. When should you use a configuration struct instead?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Default Parameters is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while building service methods, validation helpers, and reusable transformation functions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Default Parameters in an app feature:

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

- Prefer code that communicates intent at the call site, not only code that compiles.
- When a feature grows, revisit whether the type still owns the right responsibilities.

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

1. Write a minimal example that shows Default Parameters correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Default Parameters, but it shows the kind of production shape you should connect this topic to:

```swift
struct PasswordRule {
    let message: String
    let validate: (String) -> Bool
}

func validatePassword(_ password: String, rules: [PasswordRule]) -> [String] {
    rules.compactMap { rule in
        rule.validate(password) ? nil : rule.message
    }
}

let rules = [
    PasswordRule(message: "Use at least 8 characters") { $0.count >= 8 },
    PasswordRule(message: "Add one number") { $0.contains(where: \.isNumber) }
]
```

This connects function design to reuse. The validation function has clear inputs and output, while each rule is injectable and easy to test.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Default Parameters** is evaluated through this lens: functions are API design units; labels, return types, async/throws, and side effects define how other engineers use the code. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a function contract describing inputs, output, side effects, error behavior, actor isolation, and test examples
Topic: Default Parameters
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine designing validation, formatting, repository, or service APIs that many features will call. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is functions that do too much, hide mutation, or make the call site read like implementation instead of intent.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Does the call site read naturally?
- Are side effects obvious?
- Can the function be tested without UI or network?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Default Parameters**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Default Parameters** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Formatter Function

```swift
func formattedName(first: String, last: String) -> String {
    "\(first) \(last)".trimmingCharacters(in: .whitespacesAndNewlines)
}

let title = formattedName(first: "Ashish", last: "Langhe")
```

A small function keeps formatting consistent across labels, cells, and detail screens.

### Example 2: Async Service Function

```swift
func loadProfileTitle(userID: UUID, service: ProfileService) async throws -> String {
    let profile = try await service.profile(id: userID)
    return formattedName(first: profile.firstName, last: profile.lastName)
}
```

This keeps async loading and formatting separate but composable.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Keep common calls short

```swift
func makeRequest(path: String, timeout: TimeInterval = 30) -> URLRequest {
    var request = URLRequest(url: URL(string: "https://api.example.com\(path)")!)
    request.timeoutInterval = timeout
    return request
}

let request = makeRequest(path: "/profile")
```

Defaults work best when the default is genuinely safe for most callers.

### Why This Fits Default Parameters

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

