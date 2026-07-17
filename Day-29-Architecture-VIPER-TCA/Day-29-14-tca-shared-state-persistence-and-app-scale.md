# Day 29: TCA Shared State, Persistence, And App Scale

## Why Shared State Is Hard

Large apps need shared state:

- Session
- User profile
- Feature flags
- Cart
- Settings
- Cached data
- Authentication status

TCA encourages explicit ownership. Shared state should be deliberate, not global by accident.

## Parent-Owned Shared State

Root state can own shared state and pass scoped pieces to children.

```swift
@ObservableState
struct AppState: Equatable {
    var session: SessionState
    var products: ProductsFeature.State
    var cart: CartFeature.State
}
```

Parent reducer composes children.

This is explicit, but root can grow too large if abused.

## Dependency-Owned Shared State

Some shared data belongs behind a dependency.

```swift
struct SessionClient {
    var currentSession: @Sendable () async -> UserSession?
    var signOut: @Sendable () async -> Void
}
```

Feature asks dependency when needed.

This is useful for data that is persisted or external.

## Persistence

Persistence is a side effect.

```swift
struct SettingsClient {
    var loadTheme: @Sendable () async -> AppTheme
    var saveTheme: @Sendable (AppTheme) async -> Void
}
```

Reducer:

```swift
case .task:
    return .run { send in
        await send(.themeLoaded(settingsClient.loadTheme()))
    }

case .themeChanged(let theme):
    state.theme = theme
    return .run { _ in
        await settingsClient.saveTheme(theme)
    }
```

## Shared Cart Example

Cart state may need to be visible in:

- Product list
- Product details
- Checkout
- Tab badge

Options:

1. Root owns cart state and scopes it.
2. Cart feature owns cart and others send delegate actions.
3. Cart client persists/loads cart state.

Senior decision depends on consistency, persistence, and update frequency.

## Delegate Actions

Child can report events to parent.

```swift
enum Action {
    case addToCartButtonTapped(Product.ID)
    case delegate(Delegate)

    enum Delegate: Equatable {
        case addToCart(Product.ID)
    }
}
```

Reducer:

```swift
case .addToCartButtonTapped(let id):
    return .send(.delegate(.addToCart(id)))
```

Parent handles:

```swift
case .productDetails(.delegate(.addToCart(let id))):
    state.cart.items.append(CartItem(productID: id))
    return .none
```

This keeps child decoupled from parent state shape.

## App Scale Foldering

```text
AppFeature/
ProductsFeature/
ProductDetailsFeature/
CartFeature/
CheckoutFeature/
SharedModels/
Dependencies/
DesignSystem/
```

For very large apps, use Swift packages to enforce module boundaries.

## Common Mistakes

- One root state with everything.
- Child features directly mutate unrelated parent state.
- Shared mutable dependency with no tests.
- Persistence hidden in views.
- Delegate actions too broad.
- Circular feature dependencies.

## Senior Artifact

```text
Artifact: TCA Shared State Review
Shared state:
Owner:
Readers:
Writers:
Persistence:
Delegate actions:
Dependency clients:
Module boundaries:
Testing strategy:
Risks:
```

## Interview Notes

Junior:

Shared state is data multiple features need.

Mid-level:

In TCA, shared state can be parent-owned, dependency-backed, or communicated through delegate actions.

Senior:

I make shared state ownership explicit. I avoid global root bloat, use delegate actions to decouple children, and place persistence behind dependencies so shared behavior remains testable.

## Practice

1. Model cart shared across product details and checkout.
2. Add a delegate action from child to parent.
3. Create a settings persistence client.
4. Decide whether session belongs in root state or dependency.
