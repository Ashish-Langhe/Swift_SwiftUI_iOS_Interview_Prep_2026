# Day 29: VIPER Presenter, Interactor, And Router In Depth

## Presenter

The Presenter is the presentation coordinator.

It should:

- Receive View events.
- Ask Interactor for work.
- Format data into display models.
- Tell View what to render.
- Ask Router to navigate.

It should not:

- Call URLSession directly.
- Save to database directly.
- Own UIKit layout.
- Contain domain business rules.

## Presenter Example

```swift
@MainActor
final class OrdersPresenter {
    weak var view: OrdersViewInput?

    private let interactor: OrdersInteractorInput
    private let router: OrdersRouterInput

    init(interactor: OrdersInteractorInput, router: OrdersRouterInput) {
        self.interactor = interactor
        self.router = router
    }

    func viewDidLoad() {
        view?.render(.loading)

        Task {
            await loadOrders()
        }
    }

    func didTapRetry() {
        Task {
            await loadOrders()
        }
    }

    func didSelectOrder(id: Order.ID) {
        router.showOrderDetails(id: id)
    }

    private func loadOrders() async {
        do {
            let orders = try await interactor.loadOrders()
            let rows = orders.map(OrderRowDisplayModel.init)
            view?.render(rows.isEmpty ? .empty : .loaded(rows))
        } catch {
            view?.render(.failed("Unable to load orders."))
        }
    }
}
```

## Interactor

The Interactor owns use-case logic.

```swift
protocol OrdersInteractorInput {
    func loadOrders() async throws -> [Order]
    func cancelOrder(id: Order.ID) async throws
}

final class OrdersInteractor: OrdersInteractorInput {
    private let repository: OrdersRepository
    private let policy: OrderCancellationPolicy

    init(repository: OrdersRepository, policy: OrderCancellationPolicy) {
        self.repository = repository
        self.policy = policy
    }

    func loadOrders() async throws -> [Order] {
        try await repository.orders()
    }

    func cancelOrder(id: Order.ID) async throws {
        let order = try await repository.order(id: id)

        guard policy.canCancel(order) else {
            throw OrdersError.cannotCancel
        }

        try await repository.cancelOrder(id: id)
    }
}
```

The Interactor is testable without UI.

## Router

The Router owns transitions.

```swift
final class OrdersRouter: OrdersRouterInput {
    private weak var viewController: UIViewController?
    private let dependencies: OrdersDependencies

    init(viewController: UIViewController, dependencies: OrdersDependencies) {
        self.viewController = viewController
        self.dependencies = dependencies
    }

    func showOrderDetails(id: Order.ID) {
        let details = OrderDetailsAssembly.build(
            orderID: id,
            dependencies: dependencies
        )
        viewController?.navigationController?.pushViewController(details, animated: true)
    }
}
```

Router should not decide whether order cancellation is allowed. That belongs in Interactor/domain.

## Display Models

```swift
struct OrderRowDisplayModel: Equatable {
    let id: Order.ID
    let title: String
    let dateText: String
    let totalText: String
    let statusText: String

    init(order: Order) {
        id = order.id
        title = "Order #\(order.number)"
        dateText = order.createdAt.formatted(date: .abbreviated, time: .omitted)
        totalText = order.total.formatted(.currency(code: order.currencyCode))
        statusText = order.status.title
    }
}
```

Presenter creates display models from domain data.

## Interactor Output Pattern

Some VIPER codebases use Interactor output protocols.

```swift
protocol OrdersInteractorOutput: AnyObject {
    func didLoadOrders(_ orders: [Order])
    func didFailLoadingOrders(_ error: Error)
}
```

Modern Swift concurrency can simplify this with async returns. In older UIKit codebases, output protocols are common.

## Common Mistakes

- Presenter becomes business logic layer.
- Interactor knows about labels, colors, or view states.
- Router performs validation.
- Display formatting spread across View and Presenter.
- Async tasks in Presenter not managed or cancellable.
- Interactor catches all errors and hides useful domain meaning.

## Senior Artifact

```text
Artifact: VIPER Component Responsibility Review
Presenter:
Interactor:
Router:
Display models:
Domain rules:
Navigation rules:
Error flow:
Async/cancellation:
Test targets:
```

## Interview Notes

Junior:

Presenter handles view logic, Interactor handles business logic, Router handles navigation.

Mid-level:

Presenter converts domain data into display models and calls Router for screen transitions.

Senior:

I keep Presenter focused on presentation, Interactor focused on use cases, and Router focused on navigation. The moment one layer starts doing another layer's job, VIPER loses its value.

## Practice

1. Move cancellation validation from Presenter to Interactor.
2. Create an order display model in Presenter.
3. Write a Router method for details navigation.
4. Explain Interactor output protocols vs async returns.
