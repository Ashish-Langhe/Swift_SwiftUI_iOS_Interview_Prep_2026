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

