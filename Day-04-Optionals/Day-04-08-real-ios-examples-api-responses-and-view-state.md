# Day 4: Real iOS Examples With API Responses And View State

## What You Will Learn

- How optionals appear in API models
- How optionals affect UI view state
- How to avoid force unwraps in real app code
- How to choose between optional, default value, error, and enum state
- Junior to senior-level reasoning with iOS examples

## Example 1: API Response With Missing Fields

```swift
struct UserResponse: Decodable {
    let id: String
    let name: String
    let avatarURL: URL?
    let bio: String?
}
```

This model says:

- `id` is required
- `name` is required
- `avatarURL` may be missing
- `bio` may be missing

## Displaying API Data

```swift
func displayBio(from response: UserResponse) -> String {
    response.bio ?? "No bio available"
}
```

This fallback is fine if it is only for display.

But for required data:

```swift
guard !response.id.isEmpty else {
    throw APIError.missingRequiredField("id")
}
```

## Example 2: Mapping API Model To View Model

```swift
struct UserProfileViewModel {
    let name: String
    let subtitle: String
    let avatarURL: URL?
}

func makeViewModel(from response: UserResponse) -> UserProfileViewModel {
    UserProfileViewModel(
        name: response.name,
        subtitle: response.bio ?? "No bio available",
        avatarURL: response.avatarURL
    )
}
```

The view model gives the UI a display-ready `subtitle`.

## Example 3: View State With Optional

Simple optional state:

```swift
var selectedUser: User?
```

This is good when there are only two states:

- No selected user
- Selected user

But if the screen has loading, loaded, empty, and error states, use an enum.

```swift
enum ProfileViewState {
    case idle
    case loading
    case loaded(UserProfileViewModel)
    case empty
    case failed(String)
}
```

## Example 4: SwiftUI Optional Rendering

```swift
import SwiftUI

struct AvatarView: View {
    let avatarURL: URL?

    var body: some View {
        if let avatarURL {
            Text("Load image from \(avatarURL.absoluteString)")
        } else {
            Image(systemName: "person.circle")
        }
    }
}
```

This is safe because the UI handles both states.

## Example 5: Optional Delegate In UIKit

```swift
protocol PaymentDelegate: AnyObject {
    func paymentDidComplete()
}

final class PaymentViewController {
    weak var delegate: PaymentDelegate?

    func completePayment() {
        delegate?.paymentDidComplete()
    }
}
```

Delegates are often optional because the object may or may not have a listener.

## Example 6: Text Field Input

```swift
func submit(emailText: String?) {
    guard let email = emailText?.trimmingCharacters(in: .whitespaces),
          !email.isEmpty,
          email.contains("@") else {
        print("Invalid email")
        return
    }

    print("Submit \(email)")
}
```

This combines optional chaining, optional binding, and validation.

## Example 7: Avoiding Optional Explosion

Bad model:

```swift
struct CheckoutState {
    var cartId: String?
    var paymentId: String?
    var errorMessage: String?
    var isLoading: Bool
}
```

This can represent invalid combinations:

- loading with error
- payment without cart
- error and success together

Better:

```swift
enum CheckoutState {
    case empty
    case loading(cartId: String)
    case ready(cartId: String)
    case paid(paymentId: String)
    case failed(message: String)
}
```

Senior point:

Sometimes many optionals mean you need a state enum.

## Junior-Level Interview Answer

Optionals are common in iOS because API data, selected values, images, delegates, and user input may be missing.

## Mid-Level Interview Answer

I use optionals in API models when the backend field may be missing. I unwrap them safely with `if let`, `guard let`, optional chaining, or nil-coalescing depending on the use case.

## Senior-Level Interview Answer

In iOS architecture, optionals should reflect real domain absence. For UI display, nil-coalescing is often fine. For required API fields, missing data should become a decoding, validation, or mapping error. For screen state, too many optionals often indicate the need for an enum-based state machine.

## Quick Interview Notes

- API optional fields should match backend reality.
- View models can convert optionals into display-ready values.
- Optional delegates are common in UIKit.
- SwiftUI can render optional state with `if let`.
- Too many optional properties may signal poor state modeling.

## Practice Questions

1. Why are API fields often optional?
2. When should missing API data become an error?
3. How do optionals appear in SwiftUI state?
4. Why are delegates often optional?
5. When should you replace optionals with an enum state?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Real iOS Examples With API Responses And View State is not only a syntax topic. In production Swift, it affects absence modeling, safe unwrapping, nil propagation, and avoiding crash-prone assumptions. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Real iOS Examples With API Responses And View State in an app feature:

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

1. Write a minimal example that shows Real iOS Examples With API Responses And View State correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Real iOS Examples With API Responses And View State, but it shows the kind of production shape you should connect this topic to:

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

