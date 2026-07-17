# Day 29: VIPER Testing, Use Cases, And Tradeoffs

## Why VIPER Is Testable

VIPER separates responsibilities into small units:

- Presenter tests for UI state decisions.
- Interactor tests for business use cases.
- Router tests for navigation requests.
- Assembly tests for wiring if needed.

## Presenter Test

```swift
final class OrdersViewSpy: OrdersViewInput {
    private(set) var states: [OrdersViewState] = []

    func render(_ state: OrdersViewState) {
        states.append(state)
    }
}
```

Stub interactor:

```swift
struct StubOrdersInteractor: OrdersInteractorInput {
    let result: Result<[Order], Error>

    func loadOrders() async throws -> [Order] {
        try result.get()
    }
}
```

Test:

```swift
@Test
func viewDidLoad_success_rendersLoadedRows() async {
    let view = OrdersViewSpy()
    let presenter = await OrdersPresenter(
        interactor: StubOrdersInteractor(result: .success([.sample])),
        router: OrdersRouterSpy()
    )

    await presenter.setView(view)
    await presenter.viewDidLoad()

    #expect(view.states.contains(.loaded([OrderRowDisplayModel(order: .sample)])))
}
```

## Interactor Test

```swift
@Test
func cancelOrder_whenPolicyRejects_throws() async {
    let interactor = OrdersInteractor(
        repository: StubOrdersRepository(order: .shipped),
        policy: OrderCancellationPolicy()
    )

    await #expect(throws: OrdersError.cannotCancel) {
        try await interactor.cancelOrder(id: .sample)
    }
}
```

Interactors should be easy to test because they do not know UI.

## Router Test

Router tests are often lighter. You can spy on navigation controller or assert it builds the right module.

```swift
final class OrdersRouterSpy: OrdersRouterInput {
    private(set) var shownOrderID: Order.ID?

    func showOrderDetails(id: Order.ID) {
        shownOrderID = id
    }
}
```

Presenter navigation test:

```swift
@Test
func didSelectOrder_routesToDetails() {
    let router = OrdersRouterSpy()
    let presenter = OrdersPresenter(interactor: interactor, router: router)

    presenter.didSelectOrder(id: .sample)

    #expect(router.shownOrderID == .sample)
}
```

## VIPER Strengths

- Very explicit boundaries.
- Strong UIKit fit.
- Testable use cases.
- Navigation isolation.
- Team/module scalability.
- Lean view controllers.
- Clear ownership for complex features.

## VIPER Costs

- More files.
- More protocols.
- More boilerplate.
- Slower for simple screens.
- Can feel unnatural in SwiftUI.
- Risk of ceremony over value.
- More wiring/assembly code.

## Where VIPER Fits Best

Good:

- Banking flows
- Enterprise apps
- Healthcare apps
- Large UIKit apps
- Complex navigation
- Teams with strict module ownership

Weak:

- Tiny apps
- Simple SwiftUI screens
- Rapid prototypes
- Highly dynamic UI experiments

## Senior Decision

VIPER is a tradeoff:

```text
More structure, more testability, more boilerplate.
```

Use it when structure pays rent.

## Senior Artifact

```text
Artifact: VIPER Test Strategy
Module:
Presenter tests:
Interactor tests:
Router tests:
Assembly risk:
Mocks/stubs/spies:
Async behavior:
Navigation behavior:
Untested risk:
```

## Common Mistakes

- Testing only Presenter and ignoring Interactor rules.
- Using real network in Interactor tests.
- Writing fragile assembly tests.
- Too many mocks.
- No tests despite choosing VIPER for testability.
- Ignoring memory cycles.

## Interview Notes

Junior:

VIPER is testable because each part has one job.

Mid-level:

Presenter tests use view/router spies; Interactor tests use repository stubs.

Senior:

I choose VIPER when module boundaries and testability justify boilerplate. I test use cases and presentation decisions, avoid over-mocking, and keep router/assembly tests proportional to risk.

## Practice

1. Write a Presenter test with a view spy.
2. Write an Interactor test for a business rule.
3. Compare VIPER cost for a small settings screen.
4. Explain when VIPER's boilerplate is worth it.
