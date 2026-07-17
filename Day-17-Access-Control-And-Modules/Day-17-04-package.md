# Day 17: `package`

## What `package` Means

`package` allows a declaration to be used by any target inside the same Swift package, but not by code outside that package.

It sits between `internal` and `public`.

```text
private < fileprivate < internal < package < public < open
```

```swift
package struct FeatureFlag {
    package let name: String
    package let isEnabled: Bool
}
```

Targets in the same package can use `FeatureFlag`. External package users cannot.

## Why `package` Exists

Before `package`, teams often had an awkward choice:

- Use `internal`: visible only inside one target.
- Use `public`: visible to everyone importing the package.

But real packages often have multiple targets that need to share implementation.

```text
MyAppFeatures package
  LoginFeature target
  ProfileFeature target
  SharedFeatureSupport target
  FeatureTestingSupport target
```

Some APIs should be shared across these targets but hidden from app/framework consumers.

That is what `package` is for.

## Beginner Mental Model

Use `package` when you want:

```text
This is shared inside the package, but it is not public product API.
```

## Swift Package Example

Package layout:

```text
Sources/
  Networking/
    APIClient.swift
  NetworkingSupport/
    RequestBuilder.swift
  AppFeature/
    FeatureService.swift
```

`RequestBuilder` can be package-visible:

```swift
package struct RequestBuilder {
    package func makeRequest(path: String) -> URLRequest {
        URLRequest(url: URL(string: "https://api.example.com\(path)")!)
    }
}
```

Other targets in the same package can use it, but outside apps cannot.

## Real iOS Use Cases

Use `package` for:

- Shared feature infrastructure inside one package
- Internal design system primitives
- Test support utilities used by multiple package targets
- Mock factories shared across package tests
- Shared routing contracts not intended for external consumers
- DTO mappers used by multiple targets in the package

```swift
package enum AnalyticsEventName {
    package static let checkoutStarted = "checkout_started"
    package static let checkoutCompleted = "checkout_completed"
}
```

Your feature targets can reuse event names without making them part of your public SDK.

## `package` vs `internal`

`internal` is target-level.

`package` is package-level.

```text
Same target: internal works
Different target, same package: package works
Different package: public/open required
```

Example:

```swift
// Target: DesignSystemCore
package struct SpacingScale {
    package let small: CGFloat = 8
    package let medium: CGFloat = 16
}
```

Another target in the same package can use `SpacingScale`.

An app importing the published design system product cannot use it unless it is `public`.

## `package` vs `public`

Use `public` for supported API.

Use `package` for shared implementation.

```swift
public struct PrimaryButton: View {
    private let title: String

    public init(_ title: String) {
        self.title = title
    }

    public var body: some View {
        Text(title)
            .padding(SpacingScale.defaultPadding)
    }
}

package enum SpacingScale {
    package static let defaultPadding: CGFloat = 16
}
```

The button is API. The spacing implementation can remain package-only.

## Package Access And Testing

`package` is useful for test support targets.

```swift
package struct UserFactory {
    package static func makeUser(id: String = "1") -> User {
        User(id: id, name: "Test User")
    }
}
```

Multiple test targets inside the package can reuse this without making it public.

This avoids polluting production API just for tests.

## Framework Design With `package`

If you ship a framework built from a package, `package` helps keep the external API small.

```text
Public API:
  LoginView
  LoginConfiguration

Package API:
  LoginValidator
  LoginAnalyticsMapper
  LoginPreviewData
```

This is cleaner than making everything public.

## Advanced Rules And Constraints

Access levels compose with the types they mention.

A `public` declaration should not expose a `package` type in its signature.

```swift
package struct InternalConfiguration { }

public struct LoginView {
    // Invalid design: public API exposes package-only type.
    // public init(configuration: InternalConfiguration) { }
}
```

Better:

```swift
public struct LoginConfiguration {
    public let allowsBiometrics: Bool

    public init(allowsBiometrics: Bool) {
        self.allowsBiometrics = allowsBiometrics
    }
}
```

## Modern Swift 6.x Notes

`package` was introduced before Swift 6 and remains important in modern Swift package design.

Current practical relevance:

- SwiftPM-heavy apps benefit from package-level sharing.
- Large iOS codebases can avoid making support APIs public.
- Package access works well with multi-target packages.
- Concurrency annotations on public APIs should be intentional, so keeping helper APIs package-only reduces long-term compatibility pressure.

## Common Mistakes

Mistake:

Making helper APIs public because another target needs them.

Better:

Use `package` if both targets are inside the same package.

Mistake:

Using `package` in a single-target app and expecting a benefit.

Better:

Use `internal` or `private` unless you actually have package-level target sharing.

## Junior Interview Answer

`package` means a declaration is accessible across targets in the same Swift package, but not outside the package.

## Mid-Level Interview Answer

`package` is useful when a Swift package has multiple targets that need to share code without exposing that code as public API.

## Senior Interview Answer

`package` is an API design tool for multi-target Swift packages. It prevents public API pollution while allowing shared internal infrastructure across package targets. I use it for support types, test factories, shared feature infrastructure, and implementation details that cross target boundaries but should not become external contract.

## Points To Remember

- `package` is broader than `internal`.
- `package` is narrower than `public`.
- It is mainly useful in Swift Package Manager packages.
- It helps avoid accidental public API.
- Public APIs should not expose package-only types.

## Practice

1. Design a package with `Networking`, `FeatureA`, and `FeatureSupport` targets.
2. Mark shared implementation as `package` and external API as `public`.
3. Explain why `package` is better than `public` for test factories.
4. Identify a public helper that should really be package-only.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

package is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying package in an app feature:

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

1. Write a minimal example that shows package correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use package, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **package** is evaluated through this lens: access control is API economics; senior engineers decide what becomes contract and what stays replaceable. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: package
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

For **package**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

