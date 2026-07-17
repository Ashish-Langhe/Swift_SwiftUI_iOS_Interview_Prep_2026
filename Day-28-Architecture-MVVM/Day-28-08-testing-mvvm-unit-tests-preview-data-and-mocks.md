# Day 28: Testing MVVM, Unit Tests, Preview Data, And Mocks

## Why MVVM Is Testable

MVVM makes UI behavior testable because ViewModels are plain Swift objects with injected dependencies.

You can test:

- Initial state
- Loading success
- Loading failure
- Validation
- Retry
- Navigation output
- Side effects
- Formatting
- Optimistic rollback

## Test Stub

```swift
struct StubProductRepository: ProductRepository {
    let result: Result<[Product], Error>

    func products() async throws -> [Product] {
        try result.get()
    }
}
```

## Success Test

```swift
@Test
func load_success_setsLoadedRows() async {
    let repository = StubProductRepository(result: .success([.sample]))
    let viewModel = await ProductsViewModel(repository: repository)

    await viewModel.load()

    await #expect(viewModel.state == .loaded([
        ProductRowViewModel(product: .sample)
    ]))
}
```

## Failure Test

```swift
@Test
func load_failure_setsDisplayError() async {
    let repository = StubProductRepository(result: .failure(NetworkError.offline))
    let viewModel = await ProductsViewModel(repository: repository)

    await viewModel.load()

    await #expect(viewModel.state == .failed(DisplayError(
        title: "Unable to Load Products",
        message: "Check your connection and try again.",
        retryTitle: "Retry"
    )))
}
```

## Validation Test

```swift
@Test
func canSubmit_requiresValidEmailAndPassword() async {
    let viewModel = await LoginViewModel(authService: StubAuthService.success)

    await viewModel.update(email: "ashish@example.com")
    await viewModel.update(password: "12345678")

    await #expect(viewModel.canSubmit)
}
```

## Side Effect Spy

```swift
final class AnalyticsSpy: AnalyticsClient {
    private(set) var events: [String] = []

    func track(_ event: String) {
        events.append(event)
    }
}
```

Test:

```swift
@Test
func signIn_success_tracksAnalytics() async {
    let analytics = AnalyticsSpy()
    let viewModel = LoginViewModel(
        authService: StubAuthService.success,
        analytics: analytics
    )

    await viewModel.signIn()

    #expect(analytics.events == ["sign_in_success"])
}
```

## Preview Data

Previews should use deterministic models.

```swift
extension ProductsViewModel {
    static var previewLoaded: ProductsViewModel {
        let viewModel = ProductsViewModel(
            repository: StubProductRepository(result: .success([.sample, .longName]))
        )
        viewModel.state = .loaded([
            ProductRowViewModel(product: .sample),
            ProductRowViewModel(product: .longName)
        ])
        return viewModel
    }
}
```

Preview:

```swift
#Preview("Loaded") {
    ProductsScreen(viewModel: .previewLoaded)
}
```

## Mock vs Stub vs Spy

Stub:

Returns controlled data.

Spy:

Records calls for assertions.

Mock:

Pre-programmed expectation object. Use carefully; mocks can make tests brittle.

Senior preference: use simple stubs and spies unless interaction verification is truly needed.

## What Not To Test

Avoid testing:

- SwiftUI body internals.
- Private implementation details.
- Exact order of unimportant calls.
- Framework behavior.
- Every computed property when one behavior test covers it.

Test behavior and state transitions.

## Common Mistakes

- ViewModel creates live service, impossible to test.
- Tests hit real network.
- Assertions depend on private internals.
- No failure tests.
- No cancellation/race tests for search.
- Preview data too clean.
- Mock-heavy tests that break during refactor.

## Senior iOS Engineer Artifact

```text
Artifact: MVVM Test Plan
Feature:
Initial state:
Success states:
Failure states:
Validation:
Retry:
Cancellation:
Navigation:
Side effects:
Preview states:
Test doubles:
Untested risk:
```

## Interview Notes

Junior:

MVVM makes logic easier to test outside the UI.

Mid-level:

Inject stubs into ViewModels and test state after actions.

Senior:

I test ViewModels through behavior: state transitions, error mapping, validation, side effects, cancellation, and navigation outputs. I use simple stubs/spies and keep previews deterministic.

## Practice

1. Write success and failure tests for a loading ViewModel.
2. Add a spy for analytics.
3. Create preview-loaded and preview-error ViewModels.
4. Explain why tests should avoid SwiftUI body internals.
