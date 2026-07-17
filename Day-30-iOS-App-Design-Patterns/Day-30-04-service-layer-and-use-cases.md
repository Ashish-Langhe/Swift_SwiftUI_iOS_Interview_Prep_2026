# Day 30: Service Layer And Use Cases

## What Service Layer Means

A service layer contains operations that interact with external systems or perform business workflows.

Examples:

- `AuthService`
- `PaymentService`
- `LocationService`
- `NotificationService`
- `AnalyticsClient`
- `ImageUploadService`

Service methods usually perform effects:

- Network calls
- Disk writes
- Keychain access
- Location permission
- Push registration

## Basic Service

```swift
protocol AuthService {
    func signIn(email: String, password: String) async throws -> UserSession
    func signOut() async throws
}
```

Live:

```swift
struct LiveAuthService: AuthService {
    let client: HTTPClient

    func signIn(email: String, password: String) async throws -> UserSession {
        let request = SignInRequest(email: email, password: password)
        let response: SignInResponse = try await client.post("/signin", body: request)
        return try UserSession(response: response)
    }

    func signOut() async throws {
        try await client.post("/signout", body: EmptyBody())
    }
}
```

## Service Layer vs Use Case

Service:

- External capability.
- Often technical boundary.
- Auth, payment, analytics, network, storage.

Use Case:

- Business operation.
- Coordinates services/repositories.
- Represents user/business intent.

Example use case:

```swift
protocol CheckoutUseCase {
    func execute(cart: Cart, paymentMethod: PaymentMethod) async throws -> Order
}
```

Implementation:

```swift
struct LiveCheckoutUseCase: CheckoutUseCase {
    let paymentService: PaymentService
    let orderRepository: OrderRepository
    let analytics: AnalyticsClient

    func execute(cart: Cart, paymentMethod: PaymentMethod) async throws -> Order {
        let payment = try await paymentService.charge(
            amount: cart.total,
            method: paymentMethod
        )

        let order = try await orderRepository.createOrder(
            cart: cart,
            paymentID: payment.id
        )

        analytics.track("checkout_success")
        return order
    }
}
```

## ViewModel Uses Use Case

```swift
@Observable
@MainActor
final class CheckoutViewModel {
    var state: CheckoutState = .editing

    private let checkoutUseCase: CheckoutUseCase

    init(checkoutUseCase: CheckoutUseCase) {
        self.checkoutUseCase = checkoutUseCase
    }

    func submit(cart: Cart, paymentMethod: PaymentMethod) async {
        state = .submitting

        do {
            let order = try await checkoutUseCase.execute(
                cart: cart,
                paymentMethod: paymentMethod
            )
            state = .completed(order.id)
        } catch {
            state = .failed("Checkout failed.")
        }
    }
}
```

## When To Add Use Cases

Add use cases when:

- One action coordinates multiple services.
- Business workflow is important.
- Operation is reused.
- Transaction/order matters.
- ViewModel is becoming too large.

Skip when:

- ViewModel calls one simple service.
- Use case only forwards a method.
- Feature is tiny.

## Error Mapping

Services can throw technical errors:

```swift
enum NetworkError: Error {
    case offline
    case unauthorized
    case server
}
```

ViewModel maps to display:

```swift
state = .failed("Check your connection and try again.")
```

Use cases can map domain errors:

```swift
enum CheckoutError: Error {
    case paymentDeclined
    case cartChanged
    case unavailableItem
}
```

## Common Mistakes

- ViewModels doing multi-service workflows directly.
- Use cases that just forward one method.
- Services returning UI display strings.
- Analytics scattered everywhere.
- Payment/retry/idempotency not modeled.
- Service layer becomes a dumping ground.

## Senior Artifact

```text
Artifact: Service/Use Case Review
User action:
Services involved:
Use case needed:
Transaction risk:
Retry/idempotency:
Error model:
ViewModel role:
Tests:
```

## Interview Notes

Junior:

Services perform operations like network or auth.

Mid-level:

Use cases coordinate services for business actions.

Senior:

I keep technical capabilities in services and business workflows in use cases. I add use cases when coordination, transaction order, reuse, or testing value justifies the extra layer.

## Practice

1. Create an `AuthService`.
2. Create a checkout use case.
3. Move multi-service code out of a ViewModel.
4. Explain when a use case is overengineering.
