# Day 28: MVVM With Combine, Closures, And Reactive Streams

## Why This Topic Matters

Not every MVVM app is pure SwiftUI Observation. Many iOS codebases use:

- UIKit + closures
- UIKit + delegates
- Combine
- RxSwift
- AsyncSequence
- SwiftUI Observation

A senior engineer can work across styles and migrate gradually.

## Closure-Based UIKit MVVM

```swift
final class LoginViewModel {
    var onStateChanged: ((LoginState) -> Void)?

    private(set) var state: LoginState = .idle {
        didSet { onStateChanged?(state) }
    }

    private let authService: AuthService

    init(authService: AuthService) {
        self.authService = authService
    }
}
```

ViewController:

```swift
viewModel.onStateChanged = { [weak self] state in
    self?.render(state)
}
```

Use `[weak self]` to avoid retain cycles.

## Combine-Based MVVM

```swift
final class SearchViewModel: ObservableObject {
    @Published var query = ""
    @Published private(set) var results: [Product] = []

    private var cancellables: Set<AnyCancellable> = []
    private let service: SearchService

    init(service: SearchService) {
        self.service = service

        $query
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .sink { [weak self] query in
                self?.search(query)
            }
            .store(in: &cancellables)
    }
}
```

Combine is still present in many production apps.

## AsyncSequence Style

Modern concurrency can replace some reactive chains.

```swift
for await query in queryStream {
    await search(query)
}
```

Use this when async sequences make the flow clearer than publishers.

## SwiftUI Observation Style

```swift
@Observable
@MainActor
final class SearchViewModel {
    var query = ""
    var results: [Product] = []
}
```

Observation is simpler for SwiftUI UI state, but Combine can still be useful for streams, legacy APIs, and publisher-based SDKs.

## When To Use Combine

Use Combine when:

- Existing codebase uses it.
- SDK exposes publishers.
- You have complex stream composition.
- You need debouncing/throttling pipelines.
- UIKit reactive binding already exists.

Prefer async/await when:

- Flow is request/response.
- Code becomes easier to read.
- Cancellation is straightforward.
- You do not need publisher composition.

## Memory Management

Combine closures can retain `self`.

```swift
publisher
    .sink { [weak self] value in
        self?.handle(value)
    }
    .store(in: &cancellables)
```

But do not use `[weak self]` blindly if the operation must complete. Understand ownership.

## Migration Strategy

Possible migration path:

1. Keep service APIs stable.
2. Convert one ViewModel from Combine to async/await or Observation.
3. Keep tests around behavior.
4. Avoid rewriting entire architecture at once.
5. Preserve cancellation/debounce semantics.

## Common Mistakes

- Mixing Combine and async/await with unclear ownership.
- Retain cycles in sinks.
- Publisher pipelines hidden in large initializers.
- No cancellation strategy.
- Migrating for fashion instead of value.
- Replacing clear Combine pipelines with messy Task code.

## Senior iOS Engineer Artifact

```text
Artifact: Reactive MVVM Review
Codebase style:
Binding mechanism:
Streams:
Cancellation:
Memory ownership:
Schedulers/actors:
Migration value:
Testing approach:
```

## Interview Notes

Junior:

ViewModels can notify views when state changes.

Mid-level:

UIKit MVVM often uses closures or Combine; SwiftUI can use Observation.

Senior:

I choose closures, Combine, async/await, or Observation based on the codebase and problem. I preserve cancellation, memory safety, scheduler/actor correctness, and test behavior during migration.

## Practice

1. Build closure-based state output for UIKit.
2. Write a Combine debounce search ViewModel.
3. Explain Combine vs async/await for search.
4. Identify a retain cycle in a sink closure.
