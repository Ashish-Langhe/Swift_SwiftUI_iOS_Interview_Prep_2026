# Day 38: Feed, Search, And Recommendation Systems

Feeds, search, and recommendations appear in YouTube, Netflix, Amazon, Flipkart, Instagram, news apps, music apps, and many iOS products. On iOS, the challenge is not building the ML ranking system; it is consuming ranked data efficiently, rendering it smoothly, handling pagination, cancellation, caching, and analytics.

## Feed Requirements

User can:

- Open feed.
- Scroll.
- Pull to refresh.
- Paginate.
- Tap item.
- Hide/report item.
- Save/favorite.
- Resume where they left off.

Non-functional:

- Fast first contentful render.
- Smooth scrolling.
- Stable identity.
- No duplicates.
- Personalized content.
- Resilient to network failure.

## Feed API

```json
{
  "sections": [
    {
      "id": "recommended",
      "title": "Recommended For You",
      "items": []
    }
  ],
  "nextCursor": "abc"
}
```

## Feed State

```swift
enum FeedState {
    case loading
    case loaded(FeedContent)
    case empty
    case failed(String)
}

struct FeedContent {
    var sections: [FeedSection]
    var nextCursor: String?
    var isRefreshing: Bool
    var isLoadingMore: Bool
    var paginationError: String?
}
```

## Stable Identity

Every feed item needs stable ID.

```swift
enum FeedItem: Hashable {
    case video(VideoRow)
    case product(ProductRow)
    case banner(BannerRow)
    case loading(String)
}
```

Avoid using index as identity. Feeds reorder frequently.

## Duplicate Handling

Server should avoid duplicates, but client can defensively deduplicate.

```swift
func deduplicate<T: Identifiable>(_ items: [T]) -> [T] where T.ID: Hashable {
    var seen = Set<T.ID>()
    return items.filter { seen.insert($0.id).inserted }
}
```

## Search Requirements

Search should handle:

- Debounce.
- Cancellation.
- Suggestions.
- Recent searches.
- Empty query.
- Empty result.
- Error.
- Pagination.

## Search State

```swift
enum SearchState {
    case idle
    case suggestions([String])
    case loading(query: String)
    case results(query: String, rows: [SearchResultRow], nextCursor: String?)
    case empty(query: String)
    case failed(query: String, message: String)
}
```

## Stale Search Protection

```swift
@MainActor
final class SearchViewModel {
    private var task: Task<Void, Never>?
    private let service: SearchService

    func search(_ query: String) {
        task?.cancel()

        task = Task { [service] in
            do {
                try await Task.sleep(for: .milliseconds(300))
                let results = try await service.search(query)
                try Task.checkCancellation()
                await render(results, for: query)
            } catch {
                return
            }
        }
    }

    private func render(_ results: [SearchResult], for query: String) {}
}
```

## Recommendation System From iOS Perspective

The iOS app usually does not compute heavy recommendations. It:

- Sends user interaction events.
- Receives ranked items.
- Renders sections.
- Tracks impressions.
- Handles refresh and pagination.
- Supports experimentation.

## Impression Tracking

Impressions matter for ranking quality.

Track:

- Item ID.
- Position.
- Section ID.
- Experiment assignment.
- Timestamp.
- Visibility duration if needed.

Senior note: do not fire impression when cell is created. Fire when item is actually visible enough.

## Experimentation

Feed experiments may change:

- Ranking algorithm.
- Section order.
- Card design.
- Page size.
- Autoplay behavior.

iOS needs:

- Remote config.
- Feature flags.
- Analytics dimensions.
- Safe fallback.
- Kill switch.

## Feed Caching

Cache:

- Last successful feed.
- Images.
- Recent searches.
- User preferences.

Be careful caching:

- Personalized sensitive content.
- Expired promotions.
- Price/inventory.

## Pull To Refresh vs Pagination

Pull to refresh:

- Usually resets cursor.
- Replaces or merges top content.
- Should preserve user context when possible.

Pagination:

- Uses next cursor.
- Appends items.
- Failure should not destroy existing feed.

## Common Feed Bugs

- Duplicate items.
- Jumpy scroll after refresh.
- Stale search results.
- Analytics double-counting impressions.
- Cell image mismatch due to reuse.
- Pagination triggered multiple times.
- Empty state shown while refreshing existing content.

## Interview Answer Example

```text
For a personalized feed, I would use cursor pagination, stable item IDs, diffable snapshots, cached last feed for fast startup, and separate states for initial load, refresh, pagination, and failure. Search would use debounce and cancellation to avoid stale results. Recommendation ranking stays backend-side, while iOS tracks impressions and interactions reliably but asynchronously.
```

