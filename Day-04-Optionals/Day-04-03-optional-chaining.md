# Day 4: Optional Chaining

## Core Idea

Optional chaining lets you access properties, methods, and subscripts on an optional. If any part is nil, the whole chain returns nil.

```swift
let city = user.address?.city
```

If `address` is nil, `city` becomes nil.

## Basic Example

```swift
struct Address {
    let city: String
}

struct User {
    let address: Address?
}

let user = User(address: Address(city: "Pune"))
let city = user.address?.city
```

`city` is `String?`, not `String`.

## Method Calls

```swift
let text: String? = "Swift"
let count = text?.count
```

`count` is `Int?`.

## Chaining Multiple Levels

```swift
struct Company {
    let address: Address?
}

struct Employee {
    let company: Company?
}

let employee = Employee(company: nil)
let city = employee.company?.address?.city
```

If `company` or `address` is nil, `city` is nil.

## Optional Chaining With Nil-Coalescing

```swift
let displayCity = employee.company?.address?.city ?? "Unknown city"
```

This is common for display fallback.

## Optional Chaining With Functions

```swift
final class AnalyticsService {
    func track(_ event: String) {
        print(event)
    }
}

let analytics: AnalyticsService? = AnalyticsService()
analytics?.track("screen_view")
```

If `analytics` is nil, the method is not called.

## Assignment Through Optional Chaining

```swift
final class Profile {
    var name: String = "Guest"
}

var profile: Profile? = Profile()
profile?.name = "Aarav"
```

The assignment happens only if `profile` is not nil.

## Real iOS Use Cases

### API Response

```swift
struct ProfileResponse: Decodable {
    let user: UserResponse?
}

struct UserResponse: Decodable {
    let address: AddressResponse?
}

struct AddressResponse: Decodable {
    let city: String?
}

let city = response.user?.address?.city ?? "Unknown"
```

### UIKit Delegate

```swift
protocol LoginDelegate: AnyObject {
    func didLogin()
}

final class LoginViewController {
    weak var delegate: LoginDelegate?

    func loginCompleted() {
        delegate?.didLogin()
    }
}
```

### SwiftUI Optional State

```swift
let title = selectedUser?.name ?? "No user selected"
```

## Junior-Level Interview Answer

Optional chaining lets us access values inside optionals safely using `?.`.

## Mid-Level Interview Answer

Optional chaining returns nil if any optional in the chain is nil. It is useful for nested models and optional delegates.

## Senior-Level Interview Answer

Optional chaining is concise and safe, but long chains can hide weak domain modeling. If a chain like `order.customer?.profile?.address?.city` appears in many places, I may introduce a view model, computed property, or domain method that gives the UI a clearer value.

## Quick Interview Notes

- `?.` performs optional chaining.
- If any part is nil, the result is nil.
- Optional chaining result is usually optional.
- It works with properties, methods, and subscripts.
- Combine with `??` for display fallback.

## Practice Questions

1. What does `?.` do?
2. What is the result type of optional chaining?
3. Can optional chaining call methods?
4. What happens if the optional is nil?
5. When can long optional chains become a design smell?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Optional Chaining is not only a syntax topic. In production Swift, it affects absence modeling, safe unwrapping, nil propagation, and avoiding crash-prone assumptions. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while decoding API payloads, optional user profile fields, and view state that may not exist yet. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Optional Chaining in an app feature:

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

- Ask whether nil is a valid business state, an incomplete loading state, or a data-quality bug.
- Prefer explicit state enums when multiple optional values must stay consistent.

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

1. Write a minimal example that shows Optional Chaining correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Optional Chaining, but it shows the kind of production shape you should connect this topic to:

```swift
struct ProfileResponse: Decodable {
    let id: String
    let displayName: String?
    let avatarURL: URL?
}

struct ProfileViewState {
    let title: String
    let avatarURL: URL?

    init(response: ProfileResponse) {
        let trimmedName = response.displayName?.trimmingCharacters(in: .whitespacesAndNewlines)
        self.title = trimmedName?.isEmpty == false ? trimmedName! : "Guest"
        self.avatarURL = response.avatarURL
    }
}
```

This shows optional thinking in UI code: not every missing value is an error. Some nil values become fallback UI, while others should become validation or decoding failures.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Optional Chaining** is evaluated through this lens: optionals are absence modeling; senior engineers distinguish valid absence from corrupted data and incomplete loading. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: an optional-decision matrix showing nil-as-valid, nil-as-loading, nil-as-error, and nil-as-not-yet-requested
Topic: Optional Chaining
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine mapping API responses into view state where profile image, display name, token, or cached data may be missing. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is force unwraps, chains that silently hide bugs, and multiple optionals that should be one explicit enum state.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- What does nil mean here?
- Should this be an enum instead?
- Can the UI explain or recover from absence?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Optional Chaining**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Optional Chaining** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Safe Optional UI Fallback

```swift
struct UserProfile {
    let name: String?
    let city: String?
}

func subtitle(for profile: UserProfile) -> String {
    profile.city ?? "Location not added"
}
```

This is realistic because API fields are often optional, but the UI still needs stable text.

### Example 2: Guard Binding Before Navigation

```swift
func openProfile(userID: String?) {
    guard let userID else {
        print("Cannot open profile without user ID")
        return
    }

    print("Open profile: \(userID)")
}
```

`guard let` keeps the valid path clean and prevents accidental navigation with missing data.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Read through optional relationships safely

```swift
let city = order.shippingAddress?.city ?? "No city added"
```

Optional chaining is best when any missing link should produce nil instead of crashing.

### Why This Fits Optional Chaining

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

