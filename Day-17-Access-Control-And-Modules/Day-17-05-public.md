# Day 17: `public`

## What `public` Means

`public` makes a declaration usable from outside the module.

```swift
public struct APIClient {
    public init() { }

    public func fetchUser(id: String) async throws -> User {
        User(id: id, name: "Ashish")
    }
}
```

Another module can import your module and use `APIClient`.

## Beginner Mental Model

Use `public` when you want:

```text
Other modules are allowed to use this, but not subclass or override it unless I allow that separately.
```

`public` exposes usage. `open` exposes subclassing/overriding.

## Public Initializers Are Not Automatic

A public type does not automatically get a public initializer.

```swift
public struct User {
    public let id: String
    public let name: String
}
```

The memberwise initializer is not public by default.

If external modules should create `User`, write a public initializer:

```swift
public struct User {
    public let id: String
    public let name: String

    public init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}
```

This is a common interview trap.

## Public Members Are Not Automatic

If a type is public, its members are still internal by default unless marked public.

```swift
public struct Profile {
    let id: String // internal, not accessible outside module
}
```

Better:

```swift
public struct Profile {
    public let id: String

    public init(id: String) {
        self.id = id
    }
}
```

## Real iOS Use Cases

Use `public` for:

- SDK APIs
- Design system components consumed by app modules
- Feature entry views exposed to the app shell
- Public package products
- Domain models intentionally shared across modules
- Protocols external modules should conform to or depend on

```swift
public struct AppPrimaryButton: View {
    private let title: String
    private let action: () -> Void

    public init(_ title: String, action: @escaping () -> Void) {
        self.title = title
        self.action = action
    }

    public var body: some View {
        Button(title, action: action)
    }
}
```

The component is public. Its stored properties remain private.

## Public API Is A Promise

Once a declaration is public, external code can depend on it.

Changing it later can be a breaking change.

Breaking changes include:

- Renaming a public type
- Removing a public method
- Changing parameter labels
- Changing return types
- Making a public declaration internal
- Changing behavior in a surprising way

```swift
public func loadUser(id: String) async throws -> User
```

Changing this to:

```swift
public func loadUser(userID: String) async throws -> User
```

can break callers because argument labels are part of the call site.

## Public Protocols

Public protocols should be designed carefully.

```swift
public protocol ImageLoading {
    func image(for url: URL) async throws -> UIImage
}
```

Before making a protocol public, ask:

- Do external modules need to conform to it?
- Are the requirements stable?
- Will adding new requirements later break conformers?
- Should this be internal or package instead?

Adding a requirement to a public protocol can break external conforming types unless you provide a default implementation.

## Public And Concurrency

Modern public APIs often include concurrency annotations.

```swift
public protocol UserRepository: Sendable {
    func user(id: String) async throws -> User
}
```

These annotations become part of the API contract.

Be intentional with:

- `async`
- `throws`
- `Sendable`
- `@MainActor`
- actor types
- escaping closure sendability

```swift
@MainActor
public final class ProfileViewModel: ObservableObject {
    @Published public private(set) var name = ""
}
```

External callers now know this view model is main-actor isolated.

## Public Type, Private Implementation

Good public API hides implementation.

```swift
public final class AnalyticsClient {
    private let transport: AnalyticsTransport

    public init(apiKey: String) {
        self.transport = AnalyticsTransport(apiKey: apiKey)
    }

    public func track(_ event: AnalyticsEvent) {
        transport.send(event)
    }
}

private final class AnalyticsTransport {
    let apiKey: String

    init(apiKey: String) {
        self.apiKey = apiKey
    }

    func send(_ event: AnalyticsEvent) { }
}
```

Consumers see a small API.

## Advanced Design Notes

Senior engineers treat public API like product design.

Public API should be:

- Small
- Stable
- Intention-revealing
- Hard to misuse
- Documented
- Testable
- Compatible with future evolution

Avoid exposing:

- DTOs that mirror backend responses
- Internal caches
- Low-level transport types
- Helpers used only by one implementation
- Mutable state that external code should not control

## Modern Swift 6.x Notes

`public` remains stable in Swift 6.x.

Modern Swift makes public API design more sensitive because concurrency and sendability can be visible to callers.

For example:

```swift
public func observeChanges(_ handler: @escaping @Sendable (Change) -> Void)
```

The `@Sendable` requirement affects what callers can capture safely.

## Junior Interview Answer

`public` makes a declaration accessible from other modules.

## Mid-Level Interview Answer

A public type's members are not automatically public. If another module needs to initialize or read members, the initializer and members must also be public.

## Senior Interview Answer

Public API is an external contract. I keep it small and stable, hide implementation with private/internal/package declarations, and think carefully before exposing protocols, initializers, argument labels, and concurrency annotations because they affect source compatibility.

## Points To Remember

- `public` allows use from other modules.
- Public classes cannot be subclassed outside the module unless `open`.
- Public methods cannot be overridden outside the module unless `open`.
- Public type members are internal by default.
- Public initializers must be written explicitly when needed.

## Practice

1. Create a public design system button with private implementation details.
2. Explain why a public struct may still be impossible to initialize from another module.
3. Identify breaking changes in a public function signature.
4. Design a public protocol and explain the risk of adding requirements later.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

public is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while feature modules, design systems, SDK entry points, and internal helpers. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying public in an app feature:

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

- Start restrictive and widen access only when another boundary genuinely needs it.
- Treat public and open declarations as compatibility promises, not convenience keywords.

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

1. Write a minimal example that shows public correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use public, but it shows the kind of production shape you should connect this topic to:

```swift
public struct LoginFeatureView: View {
    private let configuration: LoginConfiguration

    public init(configuration: LoginConfiguration) {
        self.configuration = configuration
    }

    public var body: some View {
        LoginContentView(configuration: configuration)
    }
}

private struct LoginContentView: View {
    let configuration: LoginConfiguration
    var body: some View { Text(configuration.title) }
}
```

Access control keeps framework API small. The app needs the feature entry point, not every helper view inside the feature.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **public** is evaluated through this lens: access control is API economics; senior engineers decide what becomes contract and what stays replaceable. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: an API surface artifact listing private, internal, package, public, and open declarations with justification
Topic: public
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine designing feature modules, SDKs, design systems, Swift packages, and testing support. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is making helpers public for convenience or making classes open without supporting subclassing as a product feature.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Who needs this symbol?
- Is this public forever?
- Could package/internal solve the need?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **public**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **public** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Public Entry, Private Implementation

```swift
public struct SettingsFeatureView: View {
    public init() { }

    public var body: some View {
        SettingsContentView()
    }
}

private struct SettingsContentView: View {
    var body: some View { Text("Settings") }
}
```

A module should expose the feature entry point, not every internal view.

### Example 2: Package-Level Helper

```swift
package struct TestUserFactory {
    package static func user() -> User {
        User(id: "1", name: "Test User")
    }
}
```

`package` is useful for shared test/support code inside a Swift package.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Expose a stable API

```swift
public struct LoginConfiguration {
    public let title: String

    public init(title: String) {
        self.title = title
    }
}
```

Public initializers and members must be intentional because callers depend on them.

### Why This Fits public

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

