# Day 36.10: UIKit Navigation And Presentation

Navigation is one of the most important UIKit skills because it affects architecture, memory, deep links, user experience, and testability. A junior engineer should know how to push and present view controllers. A senior engineer should know how navigation ownership is designed, how presentations behave, how coordinators reduce coupling, and how to handle deep links without turning the app into a chain of fragile `if` statements.

## Navigation Mental Model

UIKit commonly uses these navigation containers:

- `UINavigationController`: stack-based navigation.
- `UITabBarController`: top-level app sections.
- `UISplitViewController`: master-detail and multi-column iPad layouts.
- Modal presentation: temporary focused tasks.
- Custom containers: app-specific navigation shells.

The view controller being shown should rarely decide the entire app flow. It should emit user intent. A coordinator, router, or parent controller should decide where to go.

## Push Navigation

Push when the user is moving deeper into the same task hierarchy.

```swift
final class ProductsViewController: UIViewController {
    private let coordinator: ProductsCoordinating

    init(coordinator: ProductsCoordinating) {
        self.coordinator = coordinator
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func didSelectProduct(id: Product.ID) {
        coordinator.showProductDetails(id: id)
    }
}

protocol ProductsCoordinating: AnyObject {
    func showProductDetails(id: Product.ID)
}

final class ProductsCoordinator: ProductsCoordinating {
    private let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func showProductDetails(id: Product.ID) {
        let viewModel = ProductDetailsViewModel(productID: id)
        let controller = ProductDetailsViewController(viewModel: viewModel)
        navigationController.pushViewController(controller, animated: true)
    }
}
```

Senior takeaway: the controller emits `showProductDetails`, not `push ProductDetailsViewController`.

## Modal Presentation

Present modally for focused or interrupting tasks:

- Sign in.
- Filter selection.
- Share sheet.
- Payment confirmation.
- Create/edit form.

```swift
func showFilters() {
    let controller = FiltersViewController()
    let nav = UINavigationController(rootViewController: controller)

    if let sheet = nav.sheetPresentationController {
        sheet.detents = [.medium(), .large()]
        sheet.prefersGrabberVisible = true
        sheet.prefersScrollingExpandsWhenScrolledToEdge = false
    }

    rootViewController.present(nav, animated: true)
}
```

Why wrap in a navigation controller? The modal task may need its own title, cancel/save buttons, or internal push flow.

## Presentation Styles

Common styles:

- `.automatic`: system chooses.
- `.fullScreen`: covers entire screen.
- `.pageSheet`: sheet-like presentation on supported devices.
- `.formSheet`: centered form on iPad.
- `.overFullScreen`: overlay while keeping presenting hierarchy visible.
- `.popover`: anchored contextual presentation on iPad.

Example:

```swift
let controller = SharePreviewViewController()
controller.modalPresentationStyle = .pageSheet
present(controller, animated: true)
```

Senior thinking:

- Use full screen for authentication resets or immersive flows.
- Use sheets for reversible tasks.
- Use popovers for contextual iPad interactions.
- Respect adaptive presentation on iPad and compact width.

## Dismissal

Dismiss from the owner when possible:

```swift
final class FiltersCoordinator {
    func showFilters() {
        let controller = FiltersViewController()
        controller.onCancel = { [weak self] in
            self?.root.dismiss(animated: true)
        }
        root.present(controller, animated: true)
    }
}
```

Inside a view controller, this is acceptable for local UI:

```swift
@objc private func cancelTapped() {
    dismiss(animated: true)
}
```

But for flows with consequences, prefer callback/coordinator control.

## Navigation Bar Configuration

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    title = "Orders"
    navigationItem.rightBarButtonItem = UIBarButtonItem(
        systemItem: .add,
        primaryAction: UIAction { [weak self] _ in
            self?.coordinator.showCreateOrder()
        }
    )
}
```

Large title:

```swift
navigationController?.navigationBar.prefersLargeTitles = true
navigationItem.largeTitleDisplayMode = .automatic
```

Senior note: configure appearance consistently at app boundaries, not randomly on every screen.

```swift
let appearance = UINavigationBarAppearance()
appearance.configureWithDefaultBackground()
appearance.titleTextAttributes = [.foregroundColor: UIColor.label]
appearance.largeTitleTextAttributes = [.foregroundColor: UIColor.label]

UINavigationBar.appearance().standardAppearance = appearance
UINavigationBar.appearance().scrollEdgeAppearance = appearance
```

## Transition Coordinator

Use transition coordinator to synchronize changes with navigation transitions.

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    transitionCoordinator?.animate(alongsideTransition: { _ in
        self.navigationController?.setNavigationBarHidden(false, animated: animated)
    })
}
```

Use cases:

- Coordinating nav bar visibility.
- Updating status bar style.
- Animating layout changes with a push/pop.
- Handling interactive pop cancellation.

## Custom Transitions Basics

UIKit custom transitions involve:

- `UIViewControllerTransitioningDelegate`
- `UIViewControllerAnimatedTransitioning`
- optional interactive transition controller

Simple animator:

```swift
final class FadeAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using context: UIViewControllerContextTransitioning?) -> TimeInterval {
        0.25
    }

    func animateTransition(using context: UIViewControllerContextTransitioning) {
        guard let toView = context.view(forKey: .to) else {
            context.completeTransition(false)
            return
        }

        let container = context.containerView
        toView.alpha = 0
        container.addSubview(toView)

        UIView.animate(withDuration: transitionDuration(using: context)) {
            toView.alpha = 1
        } completion: { finished in
            context.completeTransition(finished)
        }
    }
}
```

Senior caution:

- Always call `completeTransition`.
- Handle cancellation for interactive transitions.
- Keep transitions accessible and predictable.
- Avoid custom transitions for basic flows where standard UIKit behavior is better.

## Deep Link Routing

Deep links should map external input to app routes.

```swift
enum AppRoute {
    case product(id: String)
    case order(id: String)
    case settings
}

final class AppRouter {
    func route(from url: URL) -> AppRoute? {
        let components = url.pathComponents

        if components.contains("products"), let id = components.last {
            return .product(id: id)
        }

        if components.contains("settings") {
            return .settings
        }

        return nil
    }
}
```

Coordinator handles route:

```swift
func handle(_ route: AppRoute) {
    switch route {
    case .product(let id):
        selectTab(.shop)
        productsCoordinator.showProductDetails(id: id)
    case .order(let id):
        selectTab(.orders)
        ordersCoordinator.showOrder(id: id)
    case .settings:
        selectTab(.account)
        accountCoordinator.showSettings()
    }
}
```

Senior thinking:

- Validate route parameters.
- Handle authentication gates.
- Restore required tab/stack before pushing.
- Avoid pushing duplicates.
- Make routes testable without UI.

## Common Mistakes

- Pushing from cells.
- Presenting from a controller not currently visible.
- Creating navigation cycles through strong coordinator references.
- Ignoring iPad presentation behavior.
- Forgetting `completeTransition` in custom transitions.
- Treating deep links as direct view controller construction.

## Junior-Level Interview Answer

UIKit navigation usually uses `UINavigationController` for push/pop and modal presentation for focused tasks. A screen can push another view controller or present it modally, and navigation bars are configured through `navigationItem`.

## Senior-Level Interview Answer

I treat navigation as app flow, not view logic. Controllers emit intents, coordinators decide destinations, and routes model deep links. I choose push for hierarchy, modals for focused tasks, sheets/popovers based on context, and keep custom transitions limited to meaningful UX cases. I also account for iPad adaptation, interactive transition cancellation, and authentication routing.

## Points To Remember

- Push means deeper in the same hierarchy.
- Modal means focused independent task.
- Coordinators protect view controllers from app-flow knowledge.
- Deep links should route through app state, not directly instantiate random screens.
- Transition coordinator helps synchronize with navigation animations.

