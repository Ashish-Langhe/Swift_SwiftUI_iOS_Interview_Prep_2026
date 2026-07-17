# Day 27: Production Hardening, Testing, And Architecture Review

## Why This Is Advanced

Advanced SwiftUI is not only API knowledge. It is shipping reliable UI.

Production hardening includes:

- Preview coverage
- Unit tests
- UI tests
- Accessibility checks
- Performance profiling
- State restoration
- Error recovery
- Offline behavior
- Localization
- Analytics correctness
- Architecture boundaries

## Feature Review Checklist

```text
Feature:
Primary user flow:
State model:
Navigation model:
Async loading:
Error recovery:
Empty state:
Offline behavior:
Accessibility:
Dynamic type:
Localization:
Performance risk:
Testing:
Analytics:
Security/privacy:
Known tradeoffs:
```

Senior engineers make tradeoffs visible.

## Testing Observable Models

```swift
@Test
func load_success_setsLoadedState() async {
    let service = StubProductService(result: .success([.sample]))
    let model = await ProductsModel(service: service)

    await model.load()

    await #expect(model.state == .loaded([.sample]))
}
```

Test state transitions. Avoid testing SwiftUI implementation details when model tests are enough.

## Testing Navigation State

```swift
@Test
func openProduct_pushesProductRoute() async {
    let router = await AppRouter()

    await router.openProduct(Product.ID(rawValue: "123"))

    await #expect(router.homePath == [.product(Product.ID(rawValue: "123"))])
}
```

Typed route state is testable.

## Preview Audit

Important screens should have previews for:

- Loading
- Empty
- Error
- Loaded with realistic data
- Long text
- Large dynamic type
- Dark mode
- RTL or alternate locale where possible

```swift
#Preview("Error") {
    ProductsScreen(model: .previewFailed)
}
```

## Accessibility Audit

Checklist:

- Icon-only buttons have labels.
- Custom gestures have alternate actions.
- Dynamic type does not clip.
- Hit targets are large enough.
- Color is not the only signal.
- Reduce motion is respected.
- VoiceOver reads rows logically.

## Performance Audit

Checklist:

- No expensive work in `body`.
- Stable IDs.
- Lazy containers for large content.
- Images are sized and cached.
- Scroll offset updates are not global.
- Representables update idempotently.
- Instruments verifies fixes.

## Error Recovery

Good error UI gives the user a next action.

```swift
ContentUnavailableView {
    Label("Unable to Load Products", systemImage: "wifi.exclamationmark")
} description: {
    Text("Check your connection and try again.")
} actions: {
    Button("Retry") {
        Task { await model.reload() }
    }
}
```

Avoid dead-end error states.

## Analytics Correctness

Track user intent, not noisy render events.

Weak:

```swift
.onAppear {
    analytics.track("row_appeared")
}
```

Better:

```swift
Button("Checkout") {
    analytics.track("checkout_tapped")
    beginCheckout()
}
```

If screen impressions matter, dedupe them intentionally.

## Architecture Smells

Watch for:

- God `AppModel`.
- ViewModels created in `body`.
- Environment used for everything.
- Network calls in views.
- Presentation state split across many booleans.
- Full models used as routes.
- No test seams.
- Previews requiring live services.

## Senior iOS Engineer Perspective

Senior review is about predictability:

- Can the feature fail gracefully?
- Can it be tested without the app?
- Can design review all states?
- Can accessibility users complete the task?
- Can performance be measured?
- Are dependencies explicit?
- Is state ownership clear?

## Interview Notes

Junior:

Testing and previews help verify SwiftUI screens.

Mid-level:

Test view models, preview UI states, and cover error/loading flows.

Senior:

I harden SwiftUI features with state-machine thinking, testable models, route-state tests, preview matrices, accessibility audits, measured performance, and explicit dependency boundaries.

## Practice

1. Create a feature review checklist for checkout.
2. Add tests for loading success and failure.
3. Add previews for every state.
4. Audit a SwiftUI feature for architecture smells.
