# Day 17: Framework Design Basics

## What Framework Design Means

Framework design is the practice of deciding what another module can use, customize, and depend on.

In Swift, this is strongly connected to access control.

Framework API design answers:

- What should be public?
- What should be open?
- What should stay internal or package?
- What should be private implementation?
- What can change later without breaking users?

## Beginner Mental Model

A framework should feel like a clean toolkit.

Consumers should see:

- Clear entry points
- Stable types
- Useful configuration
- Helpful protocols
- Minimal implementation details

They should not see:

- Random helpers
- Backend DTOs
- Temporary mappers
- Internal caches
- Experimental types

## Basic Framework Shape

```text
DesignSystem framework
  Public:
    PrimaryButton
    AppTextField
    AppTheme

  Internal:
    ButtonStyleResolver
    ThemeStorage
    ColorParser

  Private:
    One-type helper methods
```

Public API should be intentionally small.

## Public Entry Points

```swift
public struct PrimaryButton: View {
    private let title: String
    private let action: () -> Void

    public init(_ title: String, action: @escaping () -> Void) {
        self.title = title
        self.action = action
    }

    public var body: some View {
        Button(title, action: action)
            .buttonStyle(PrimaryButtonStyle())
    }
}

private struct PrimaryButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
    }
}
```

The framework exposes `PrimaryButton`, not every styling helper.

## Initializer Design

Initializers are part of API design.

```swift
public struct AnalyticsConfiguration {
    public let apiKey: String
    public let environment: Environment

    public init(apiKey: String, environment: Environment = .production) {
        self.apiKey = apiKey
        self.environment = environment
    }
}
```

Good initializers:

- Require essential values
- Provide safe defaults
- Avoid exposing internal dependencies
- Use clear argument labels
- Avoid invalid states

## Avoid Leaking DTOs

Bad:

```swift
public struct UserResponse: Decodable {
    public let user_id: String
    public let full_name: String
}
```

If this is public, external modules may depend on backend naming.

Better:

```swift
public struct User {
    public let id: String
    public let name: String

    public init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}

internal struct UserResponse: Decodable {
    let user_id: String
    let full_name: String
}
```

Keep transport details internal.

## Public Protocol Design

```swift
public protocol ImageLoading {
    func image(for url: URL) async throws -> UIImage
}
```

Before exposing a protocol, ask:

- Will external modules implement this?
- Are requirements stable?
- Should this be a concrete type instead?
- Can default implementations reduce future breakage?
- Does it need `Sendable`?

Adding a requirement to a public protocol can break external conformers.

## Open Class Design

Only use `open` when subclassing is supported.

```swift
open class BaseAnalyticsProvider {
    public init() { }

    open func track(event: String, properties: [String: String]) {
        fatalError("Subclasses must override")
    }
}
```

Document:

- Which methods subclasses override
- Which methods they should call `super` from
- Which methods are final
- What thread or actor methods run on

If you do not want subclassing, prefer:

```swift
public final class AnalyticsClient { }
```

## Framework Access Strategy

Useful default strategy:

```text
Start private.
Raise to internal when another file in the module needs it.
Raise to package when another target in the same package needs it.
Raise to public only when external modules need it.
Raise to open only when external subclassing is a designed feature.
```

This avoids accidental API growth.

## API Evolution

Public API changes affect users.

Source-breaking changes:

- Rename public type or method
- Remove public member
- Change argument labels
- Change generic constraints
- Add requirements to public protocols
- Change a public class from open to public
- Remove public initializer

Usually safe changes:

- Add a new public overload
- Add a new public type
- Add defaulted parameters carefully
- Add internal implementation
- Change private/internal code without changing behavior

## Documentation And Examples

Public APIs deserve examples.

```swift
/// Tracks analytics events for the current app session.
public final class AnalyticsClient {
    /// Creates an analytics client.
    public init(configuration: AnalyticsConfiguration) { }
}
```

Good docs explain:

- What the API does
- Main use case
- Thread/actor expectations
- Error behavior
- Lifecycle expectations

## Modern Swift 6.x Framework Notes

Modern framework APIs often need concurrency clarity.

```swift
public protocol ProfileService: Sendable {
    func profile(id: String) async throws -> Profile
}
```

```swift
@MainActor
public final class ProfileViewModel: ObservableObject {
    @Published public private(set) var profile: Profile?
}
```

Think carefully before exposing:

- `@MainActor`
- `Sendable`
- `@Sendable`
- actor types
- nonisolated methods
- async sequences

These choices affect callers.

## iOS Framework Example

```swift
public struct LoginFeatureView: View {
    private let configuration: LoginConfiguration

    public init(configuration: LoginConfiguration) {
        self.configuration = configuration
    }

    public var body: some View {
        LoginRootView(configuration: configuration)
    }
}

public struct LoginConfiguration {
    public let title: String
    public let allowsBiometrics: Bool

    public init(title: String, allowsBiometrics: Bool = true) {
        self.title = title
        self.allowsBiometrics = allowsBiometrics
    }
}

private struct LoginRootView: View {
    let configuration: LoginConfiguration

    var body: some View {
        Text(configuration.title)
    }
}
```

The public API is small and clear.

## Senior Design Checklist

Before making something public, ask:

- Is this part of the product contract?
- Can I support this for multiple releases?
- Does it expose implementation details?
- Can callers create invalid states?
- Should this be configured through a smaller type?
- Should this be `package` instead?
- Should this class be `final`?
- Does concurrency isolation belong in the API?

## Junior Interview Answer

Framework design means exposing only the types and functions other modules need while hiding implementation details.

## Mid-Level Interview Answer

I keep framework public APIs small. I use public for supported entry points, package/internal for shared implementation, private for local details, and open only for intentional subclassing.

## Senior Interview Answer

Framework design is API contract design. Access control protects implementation, reduces coupling, and controls source compatibility. I design public APIs around stable domain concepts, avoid leaking DTOs or infrastructure, prefer final classes unless inheritance is required, and treat concurrency annotations as part of the API surface.

## Points To Remember

- Public API is a promise.
- Open API is an inheritance promise.
- Public initializers must be explicit.
- Public protocols are harder to evolve.
- Keep DTOs and mappers internal.
- Use `package` for shared package implementation.

## Practice

1. Design a public API for a Login framework.
2. Decide which types are public, package, internal, and private.
3. Explain why exposing backend DTOs is risky.
4. Convert an open class design into public final plus protocol composition.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Framework Design Basics is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Framework Design Basics in an app feature:

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

1. Write a minimal example that shows Framework Design Basics correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Framework Design Basics, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Framework Design Basics** is evaluated through this lens: access control is API economics; senior engineers decide what becomes contract and what stays replaceable. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Framework Design Basics
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

For **Framework Design Basics**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

