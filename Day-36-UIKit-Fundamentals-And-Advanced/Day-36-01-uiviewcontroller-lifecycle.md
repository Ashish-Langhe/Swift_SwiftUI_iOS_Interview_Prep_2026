# Day 36.1: UIViewController Lifecycle

UIKit is still extremely important for iOS interviews because many production apps are UIKit-first, UIKit-heavy, or hybrid UIKit plus SwiftUI. A senior iOS engineer should understand not only which lifecycle method runs when, but also what work belongs in each method, how containment affects callbacks, and how lifecycle decisions impact performance, networking, analytics, memory, and UI correctness.

## Mental Model

A `UIViewController` owns a view hierarchy and coordinates it with model state, navigation, presentation, layout, and system events. The lifecycle is not just a list of callbacks. It is a contract between your controller and UIKit.

The lifecycle answers questions like:

- When is the view loaded into memory?
- When is the view about to become visible?
- When is the view actually on screen?
- When should expensive work start or stop?
- When should layout-dependent code run?
- When should child controllers be added or removed?

## Core Lifecycle Order

Common first presentation flow:

```swift
final class ProfileViewController: UIViewController {
    override func loadView() {
        view = ProfileView()
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Profile"
        configureCallbacks()
        bindViewModel()
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        refreshVisibleState()
    }

    override func viewIsAppearing(_ animated: Bool) {
        super.viewIsAppearing(animated)
        updateNavigationBarAppearance()
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        analytics.trackScreen("Profile")
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        saveDraftIfNeeded()
    }

    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        stopVisibleOnlyWork()
    }
}
```

## `loadView`

`loadView` creates the root view. Override it when building UI programmatically.

Good use:

```swift
override func loadView() {
    view = ProductListView()
}
```

Avoid:

```swift
override func loadView() {
    super.loadView()
    view.addSubview(label)
}
```

If you call `super.loadView()`, UIKit creates a default blank view. For programmatic UI, assign your custom root view directly.

Senior thinking:

- Use `loadView` to clearly separate view construction from state binding.
- Avoid accessing `view` before `loadView` is ready unless you intentionally want to load the view.
- Keep business logic out of `loadView`.

## `viewDidLoad`

`viewDidLoad` is called after the view hierarchy is loaded into memory. It usually runs once for the controller instance.

Good work here:

- Add static configuration.
- Register cells.
- Configure constraints.
- Wire callbacks.
- Bind view model outputs.
- Set accessibility identifiers.

Example:

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    collectionView.register(ProductCell.self, forCellWithReuseIdentifier: ProductCell.reuseID)
    collectionView.delegate = self
    dataSource = makeDataSource()

    viewModel.onStateChange = { [weak self] state in
        self?.render(state)
    }
}
```

Be careful with:

- Starting camera/location work too early.
- Tracking screen analytics before the screen is visible.
- Assuming view sizes are final.
- Fetching data every time visibility changes.

## `viewWillAppear`

Called before the view appears. Use it for state that should update every time the screen returns.

Example:

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    cartBadgeView.count = cartStore.currentCount
}
```

Good use cases:

- Refreshing navigation bar appearance.
- Updating badges or summaries.
- Resubscribing to visible-only notifications.
- Preparing transition-related state.

Bad use cases:

- Re-registering cells.
- Rebuilding entire UI every time.
- Starting duplicate network requests without cancellation.

## `viewIsAppearing`

`viewIsAppearing(_:)` is useful because it is called while the view is being added to the hierarchy and after traits/layout environment are more reliable than early lifecycle points. Current Apple docs list it among the view-related appearance callbacks.

Example:

```swift
override func viewIsAppearing(_ animated: Bool) {
    super.viewIsAppearing(animated)
    updateForCurrentTraitCollection()
}
```

Use it for:

- Appearance work that depends on current traits.
- Updating layout-sensitive visual state before the user sees the final result.
- Coordinating navigation bar style for a transition.

## `viewDidAppear`

Called after the view is visible.

Good use:

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)

    if shouldRequestReview {
        reviewPrompter.requestReviewIfAppropriate()
    }
}
```

Use it for:

- Analytics screen tracking.
- Starting visible-only animations.
- Requesting permissions when UX requires a visible screen.
- Focusing a text field after transition.

Avoid:

- Heavy synchronous work.
- Blocking the main thread.
- Recreating constraints.

## Disappearance Callbacks

Use `viewWillDisappear` and `viewDidDisappear` to stop work that only matters while visible.

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    searchController.searchBar.resignFirstResponder()
}

override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    videoPreview.pause()
}
```

Senior detail: `viewWillDisappear` can happen because another controller is pushed, a modal appears, a tab changes, or the controller is dismissed. Check transition state before assuming the screen is gone forever.

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)

    if isMovingFromParent || isBeingDismissed {
        viewModel.cancelLongRunningWork()
    }
}
```

## Layout Lifecycle

Important callbacks:

- `viewWillLayoutSubviews`
- `viewDidLayoutSubviews`
- `traitCollectionDidChange`
- `viewSafeAreaInsetsDidChange`

Example:

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    avatarImageView.layer.cornerRadius = avatarImageView.bounds.width / 2
}
```

Do not repeatedly add constraints in `viewDidLayoutSubviews`. It can run many times.

## Memory And Deinit

Modern iOS does not commonly unload a view controller's view under memory pressure the way old iOS versions did, but controllers still need good memory hygiene.

```swift
deinit {
    NotificationCenter.default.removeObserver(self)
    print("ProfileViewController deallocated")
}
```

With block-based observers, Combine subscriptions, tasks, delegates, and closures, `deinit` is a useful verification point.

## Async Work In Lifecycle

UIKit classes are main-actor UI objects. Start async work carefully and cancel it when ownership ends.

```swift
final class OrdersViewController: UIViewController {
    private var loadTask: Task<Void, Never>?

    override func viewDidLoad() {
        super.viewDidLoad()

        loadTask = Task { [weak self] in
            guard let self else { return }
            await self.loadOrders()
        }
    }

    @MainActor
    private func loadOrders() async {
        do {
            let orders = try await service.fetchOrders()
            render(orders)
        } catch {
            renderError(error)
        }
    }

    deinit {
        loadTask?.cancel()
    }
}
```

Senior note: if the task captures `self` strongly and the service never returns, the controller may remain alive. Think in terms of ownership and cancellation.

## Child View Controller Lifecycle

Correct containment:

```swift
func embed(_ child: UIViewController, in container: UIView) {
    addChild(child)
    container.addSubview(child.view)
    child.view.translatesAutoresizingMaskIntoConstraints = false

    NSLayoutConstraint.activate([
        child.view.leadingAnchor.constraint(equalTo: container.leadingAnchor),
        child.view.trailingAnchor.constraint(equalTo: container.trailingAnchor),
        child.view.topAnchor.constraint(equalTo: container.topAnchor),
        child.view.bottomAnchor.constraint(equalTo: container.bottomAnchor)
    ])

    child.didMove(toParent: self)
}
```

Correct removal:

```swift
func remove(_ child: UIViewController) {
    child.willMove(toParent: nil)
    child.view.removeFromSuperview()
    child.removeFromParent()
}
```

Interview trap: adding a child view without calling `addChild` and `didMove` breaks appearance forwarding, rotation behavior, and containment semantics.

## Common Mistakes

- Doing layout-size work in `viewDidLoad`.
- Forgetting `super`.
- Starting duplicate network calls in `viewWillAppear`.
- Keeping strong closure cycles between view and controller.
- Updating UI from background threads.
- Forgetting child containment calls.
- Assuming `viewDidDisappear` always means dismissal.

## Junior-Level Interview Answer

`viewDidLoad` is for one-time setup after the view loads. `viewWillAppear` runs before the screen appears and can update visible state. `viewDidAppear` runs after it is on screen. Disappear methods are used to stop work or save state. For programmatic UI, `loadView` can create the root view.

## Senior-Level Interview Answer

The lifecycle is about ownership, visibility, layout readiness, and side-effect timing. I keep static UI setup in `viewDidLoad`, visible-state refresh in `viewWillAppear` or `viewIsAppearing`, analytics and permission prompts in `viewDidAppear`, and cancellation or cleanup in disappearance/deinit depending on ownership. I also distinguish temporary disappearance from removal using `isMovingFromParent` and `isBeingDismissed`, and I handle child containment explicitly.

## Points To Remember

- `viewDidLoad` is not a visibility callback.
- `viewWillAppear` can run many times.
- `viewDidLayoutSubviews` can run many times.
- `viewIsAppearing` is valuable for appearance updates with accurate environment.
- Use `deinit` to verify ownership cleanup.
- Lifecycle bugs often show up as duplicate requests, janky transitions, stale navigation bars, or memory leaks.

