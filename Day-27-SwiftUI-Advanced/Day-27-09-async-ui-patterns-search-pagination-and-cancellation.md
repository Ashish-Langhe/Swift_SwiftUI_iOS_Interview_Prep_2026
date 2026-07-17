# Day 27: Async UI Patterns, Search, Pagination, And Cancellation

## Why Async UI Patterns Are Advanced

Real SwiftUI apps need more than one `await service.load()`.

Common advanced flows:

- Search with debounce
- Pagination
- Pull to refresh
- Retry
- Cancellation
- Optimistic updates
- Offline fallback
- Race prevention

## Search With Cancellation

```swift
@Observable
@MainActor
final class SearchModel {
    var query = ""
    var results: [Product] = []
    var isSearching = false
    var errorMessage: String?

    @ObservationIgnored
    private let service: SearchService

    @ObservationIgnored
    private var searchTask: Task<Void, Never>?

    init(service: SearchService) {
        self.service = service
    }

    func queryChanged(_ query: String) {
        self.query = query
        searchTask?.cancel()

        searchTask = Task {
            try? await Task.sleep(for: .milliseconds(300))
            guard !Task.isCancelled else { return }
            await search()
        }
    }

    private func search() async {
        guard !query.isEmpty else {
            results = []
            return
        }

        isSearching = true
        defer { isSearching = false }

        do {
            results = try await service.search(query)
        } catch is CancellationError {
            return
        } catch {
            errorMessage = "Search failed."
        }
    }
}
```

## Pagination

```swift
@Observable
@MainActor
final class FeedModel {
    var posts: [Post] = []
    var isLoadingPage = false
    var canLoadMore = true

    private var nextPage: String?
    private let service: FeedService

    init(service: FeedService) {
        self.service = service
    }

    func loadNextPageIfNeeded(currentPost: Post?) async {
        guard let currentPost else { return }
        guard currentPost.id == posts.last?.id else { return }
        await loadNextPage()
    }

    func loadNextPage() async {
        guard !isLoadingPage, canLoadMore else { return }

        isLoadingPage = true
        defer { isLoadingPage = false }

        do {
            let page = try await service.page(after: nextPage)
            posts += page.items
            nextPage = page.nextCursor
            canLoadMore = page.nextCursor != nil
        } catch {
            // expose retry state
        }
    }
}
```

View:

```swift
List(model.posts) { post in
    PostRow(post: post)
        .task {
            await model.loadNextPageIfNeeded(currentPost: post)
        }
}
```

## Retry

```swift
func retry() async {
    errorMessage = nil
    await load()
}
```

For production networking, retry policy should be deliberate:

- Which errors are retryable?
- How many attempts?
- Is there backoff?
- Can user cancel?
- Does retry duplicate writes?

## Optimistic Updates

```swift
func favorite(_ product: Product) async {
    guard let index = products.firstIndex(where: { $0.id == product.id }) else {
        return
    }

    products[index].isFavorite.toggle()
    let newValue = products[index].isFavorite

    do {
        try await service.setFavorite(newValue, productID: product.id)
    } catch {
        products[index].isFavorite.toggle()
        errorMessage = "Could not update favorite."
    }
}
```

Optimistic updates improve perceived speed but need rollback.

## Race Conditions

If two searches finish out of order, stale results can overwrite fresh results.

Use cancellation, request IDs, or actor-isolated services.

```swift
let requestQuery = query
let response = try await service.search(requestQuery)

guard requestQuery == query else {
    return
}

results = response
```

## Common Mistakes

- No cancellation for search.
- Multiple pagination calls at once.
- Retry duplicate writes.
- No rollback for optimistic updates.
- Loading state stuck after errors.
- UI state mutated from wrong actor.
- Error messages too technical.

## Senior iOS Engineer Perspective

Senior async UI design asks:

- What cancels this work?
- Can responses arrive out of order?
- Is loading state idempotent?
- Are writes retry-safe?
- Is pagination guarded?
- Is optimistic state rollback correct?
- Are errors recoverable?

## Interview Notes

Junior:

Use `async/await` to load data and update UI.

Mid-level:

Handle loading, error, retry, refresh, and cancellation states.

Senior:

I design async UI as a state machine with cancellation, race prevention, retry policy, pagination guards, optimistic rollback, and main-actor isolation.

## Practice

1. Add debounced search with cancellation.
2. Build pagination that avoids duplicate loads.
3. Add optimistic favorite with rollback.
4. Explain stale response prevention.
