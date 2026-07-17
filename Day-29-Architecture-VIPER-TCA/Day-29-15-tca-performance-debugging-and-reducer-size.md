# Day 29: TCA Performance, Debugging, And Reducer Size

## Why Performance Still Matters In TCA

TCA gives structure, but it does not automatically make apps fast.

Performance issues can come from:

- Overly broad state observation
- Large reducers
- Expensive derived state
- Too many actions
- Heavy effects
- Large navigation state
- Views observing more state than needed
- Debug tools in production builds

## State Observation

Views should observe the state they need.

Bad:

```swift
struct ProductRow: View {
    let store: StoreOf<ProductsFeature>
}
```

Every row may observe too much parent state.

Better:

```swift
struct ProductRow: View {
    let row: ProductRowState
    let onTap: () -> Void
}
```

Or scope to a smaller child feature only when row behavior needs its own reducer.

## Derived State

Avoid expensive derived state recomputed frequently.

```swift
var filteredProducts: [Product] {
    products.filter { $0.name.localizedCaseInsensitiveContains(query) }
}
```

Fine for small data. For large datasets, move computation into reducer/effect when query changes.

## Reducer Size

Reducer getting too big?

Symptoms:

- Action enum has dozens of unrelated cases.
- State mixes multiple screens.
- Tests are difficult to read.
- Effects for unrelated workflows live together.
- Navigation and business logic are tangled.

Split into child features.

```swift
@Reducer
struct CheckoutFeature {
    var body: some Reducer<State, Action> {
        Scope(state: \.shipping, action: \.shipping) {
            ShippingFeature()
        }

        Scope(state: \.payment, action: \.payment) {
            PaymentFeature()
        }

        Reduce(core)
    }
}
```

## Debugging Actions

TCA encourages clear action names.

Bad:

```swift
case changed
case loaded
case tapped
```

Good:

```swift
case checkoutButtonTapped
case paymentResponse(Result<PaymentResult, PaymentError>)
case shippingAddressChanged(Address)
```

Good action names make logs, tests, and reviews useful.

## Reentrant Actions

Recent TCA releases include warnings around reentrant actions. Reentrant actions can happen when state changes synchronously cause additional sends in unsafe ways.

Senior takeaway:

- Keep reducer logic deterministic.
- Avoid sending actions from unexpected synchronous observation hooks.
- Prefer effects for follow-up work.
- Read release notes during library upgrades.

## Effect Load

Effects should be cancellable when appropriate.

```swift
.cancellable(id: SearchCancelID(), cancelInFlight: true)
```

Long-running effects should not leak or continue after feature dismissal.

## Debugging Checklist

```text
Slow screen:
Which view observes state?
Which action fires frequently?
Is derived state expensive?
Are effects duplicated?
Is cancellation missing?
Is navigation state huge?
Can reducer split?
What does Instruments show?
```

## Common Mistakes

- Passing root store everywhere.
- Computed state filters large arrays repeatedly.
- Reducer grows without composition.
- No cancellation for long effects.
- Debug logging too noisy.
- State includes non-equatable/reference-heavy objects.
- Ignoring TCA release deprecations.

## Senior Artifact

```text
Artifact: TCA Performance Review
Feature:
Observed state:
Frequent actions:
Derived state cost:
Reducer size:
Effect cancellation:
Navigation state:
Instrumentation:
Refactor plan:
```

## Interview Notes

Junior:

TCA organizes state and actions, but performance still matters.

Mid-level:

Scope state carefully, keep reducers focused, and cancel effects.

Senior:

I debug TCA performance by inspecting observed state, action frequency, derived state cost, effect lifetime, reducer size, and Instruments evidence. I split features when reducer boundaries become unclear.

## Practice

1. Refactor a large reducer into child features.
2. Remove expensive computed filtering from view observation.
3. Add cancellation to a frequent effect.
4. Explain why passing root store everywhere is harmful.
