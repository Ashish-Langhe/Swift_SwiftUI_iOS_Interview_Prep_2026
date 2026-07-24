# Day 37: Solved iOS Architecture Coding Scenarios

These problems are closer to what I would ask for mid, senior, and architect-level iOS interviews. They mix logical coding with real app concerns: state, identity, caching, async work, memory, and testability.

## Scenario 1: Product List View State

### Prompt

Model a product list screen that supports:

- Initial loading.
- Loaded products.
- Empty state.
- Error state.
- Pull-to-refresh.
- Pagination.
- Pagination error.

### Solution

```swift
struct ProductRow: Hashable, Identifiable {
    let id: String
    let title: String
    let price: String
    let isFavorite: Bool
}

enum ProductListState: Equatable {
    case idle
    case loading
    case loaded(ProductListContent)
    case empty
    case failed(message: String)
}

struct ProductListContent: Equatable {
    var rows: [ProductRow]
    var isRefreshing: Bool
    var isLoadingNextPage: Bool
    var paginationErrorMessage: String?
    var canLoadMore: Bool
}
```

### Why This Is Good

Initial failure and pagination failure are different. Initial failure may show a full-screen error. Pagination failure should usually keep existing rows and show a footer retry.

### Senior Discussion

A weak answer uses multiple booleans:

```swift
var isLoading = false
var isEmpty = false
var error: String?
var rows: [ProductRow] = []
```

This allows invalid states like `isLoading == true`, `isEmpty == true`, and `rows` not empty. A stronger enum makes invalid states harder.

## Scenario 2: DTO To Domain To View Model Mapping

### Prompt

Given an API product DTO with optional fields, map it into a safe UI row.

### Solution

```swift
struct ProductDTO: Decodable {
    let id: String?
    let name: String?
    let price: Double?
    let currencyCode: String?
    let isAvailable: Bool?
}

struct Product {
    let id: String
    let name: String
    let price: Decimal
    let currencyCode: String
    let isAvailable: Bool
}

struct ProductRow: Hashable, Identifiable {
    let id: String
    let title: String
    let subtitle: String
}

enum ProductMappingError: Error {
    case missingID
    case missingName
}

func mapDTOToDomain(_ dto: ProductDTO) throws -> Product {
    guard let id = dto.id, !id.isEmpty else {
        throw ProductMappingError.missingID
    }

    guard let name = dto.name, !name.isEmpty else {
        throw ProductMappingError.missingName
    }

    return Product(
        id: id,
        name: name,
        price: Decimal(dto.price ?? 0),
        currencyCode: dto.currencyCode ?? "USD",
        isAvailable: dto.isAvailable ?? false
    )
}

func mapDomainToRow(_ product: Product) -> ProductRow {
    ProductRow(
        id: product.id,
        title: product.name,
        subtitle: product.isAvailable ? "Available" : "Unavailable"
    )
}
```

### Senior Discussion

Do not pass raw DTOs directly to UI. DTOs represent network shape. Domain models represent app meaning. Row models represent display.

## Scenario 3: Merge Cached And Remote Products

### Prompt

Remote products arrive from API. Local cache contains favorite state. Merge them so remote data is fresh but local favorite state is preserved.

### Solution

```swift
struct RemoteProduct {
    let id: String
    let name: String
    let price: Decimal
}

struct CachedProduct {
    let id: String
    let isFavorite: Bool
}

struct ProductDisplayModel: Hashable, Identifiable {
    let id: String
    let name: String
    let price: Decimal
    let isFavorite: Bool
}

func mergeProducts(
    remote: [RemoteProduct],
    cached: [CachedProduct]
) -> [ProductDisplayModel] {
    let cachedByID = Dictionary(uniqueKeysWithValues: cached.map { ($0.id, $0) })

    return remote.map { product in
        ProductDisplayModel(
            id: product.id,
            name: product.name,
            price: product.price,
            isFavorite: cachedByID[product.id]?.isFavorite ?? false
        )
    }
}
```

### Complexity

- Time: O(n + m)
- Space: O(m)

### Senior Discussion

Clarify:

- Should locally favorite products missing from remote still appear?
- Does remote deletion remove local favorite state?
- Should order follow remote ranking or local order?
- Should favorite changes sync to server?

This is the kind of question where product behavior matters as much as code.

## Scenario 4: Search With Cancellation

### Prompt

Build view-model logic for search that cancels stale requests.

### Solution

```swift
@MainActor
final class ProductSearchViewModel {
    enum State: Equatable {
        case idle
        case loading(query: String)
        case loaded(query: String, rows: [ProductRow])
        case empty(query: String)
        case failed(query: String, message: String)
    }

    private let service: ProductSearchService
    private var searchTask: Task<Void, Never>?
    private(set) var state: State = .idle

    init(service: ProductSearchService) {
        self.service = service
    }

    func search(query: String) {
        searchTask?.cancel()

        let trimmed = query.trimmingCharacters(in: .whitespacesAndNewlines)
        guard !trimmed.isEmpty else {
            state = .idle
            return
        }

        state = .loading(query: trimmed)

        searchTask = Task { [service] in
            do {
                try await Task.sleep(for: .milliseconds(300))
                let products = try await service.searchProducts(query: trimmed)
                try Task.checkCancellation()

                let rows = products.map(ProductRow.init)
                state = rows.isEmpty
                    ? .empty(query: trimmed)
                    : .loaded(query: trimmed, rows: rows)
            } catch is CancellationError {
                return
            } catch {
                state = .failed(query: trimmed, message: "Search failed.")
            }
        }
    }

    deinit {
        searchTask?.cancel()
    }
}
```

### Senior Discussion

This tests:

- Cancellation.
- Debounce.
- Main actor state.
- Empty query behavior.
- Stale result prevention.

Ask whether search should show cached results while loading. Ask whether errors should clear previous results or keep them.

## Scenario 5: Actor-Isolated Memory Cache

### Prompt

Design a simple thread-safe cache for images or decoded data.

### Solution

```swift
actor MemoryCache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]

    func value(for key: Key) -> Value? {
        storage[key]
    }

    func setValue(_ value: Value, for key: Key) {
        storage[key] = value
    }

    func removeValue(for key: Key) {
        storage.removeValue(forKey: key)
    }

    func removeAll() {
        storage.removeAll()
    }
}
```

### Usage

```swift
let cache = MemoryCache<URL, Data>()

func loadData(from url: URL) async throws -> Data {
    if let cached = await cache.value(for: url) {
        return cached
    }

    let (data, _) = try await URLSession.shared.data(from: url)
    await cache.setValue(data, for: url)
    return data
}
```

### Senior Discussion

This cache is thread-safe but not production-complete. A real cache may need:

- Cost limits.
- Expiration.
- Eviction.
- Memory warning handling.
- Disk persistence.
- Duplicate in-flight request coalescing.

## Scenario 6: Token Refresh Coordination

### Prompt

Multiple requests receive unauthorized at the same time. Ensure only one refresh request happens.

### Solution

```swift
actor AuthTokenStore {
    private var token: String?
    private var refreshTask: Task<String, Error>?
    private let service: AuthService

    init(service: AuthService) {
        self.service = service
    }

    func currentToken() -> String? {
        token
    }

    func refreshToken() async throws -> String {
        if let refreshTask {
            return try await refreshTask.value
        }

        let task = Task { [service] in
            try await service.refreshToken()
        }

        refreshTask = task

        do {
            let newToken = try await task.value
            token = newToken
            refreshTask = nil
            return newToken
        } catch {
            refreshTask = nil
            throw error
        }
    }
}
```

### Senior Discussion

This tests actor isolation and duplicate-work prevention. Discuss:

- What if refresh fails?
- Should user be logged out?
- Should queued requests retry once only?
- Should refresh token be stored in Keychain?
- How do we prevent infinite 401 loops?

## Scenario 7: Diffable Row Identity

### Prompt

Create row models for a profile screen with mixed row types.

### Solution

```swift
enum ProfileSection: Hashable {
    case header
    case account
    case support
}

enum ProfileRow: Hashable {
    case userHeader(id: String, name: String, email: String)
    case menu(id: String, title: String, iconName: String)
    case logout(id: String)

    var stableID: String {
        switch self {
        case .userHeader(let id, _, _):
            return "header-\(id)"
        case .menu(let id, _, _):
            return "menu-\(id)"
        case .logout(let id):
            return "logout-\(id)"
        }
    }

    func hash(into hasher: inout Hasher) {
        hasher.combine(stableID)
    }

    static func == (lhs: ProfileRow, rhs: ProfileRow) -> Bool {
        lhs.stableID == rhs.stableID
    }
}
```

### Senior Discussion

Identity-only equality helps diffable understand stable rows, but content updates may need `reconfigureItems`. If equality includes all display fields, content changes may look like delete/insert. Know the tradeoff.

## Scenario 8: Repository Cache-Then-Network

### Prompt

Design repository behavior that emits cached products first, then fresh remote products.

### Solution

```swift
protocol ProductLocalStore {
    func loadProducts() async throws -> [Product]
    func saveProducts(_ products: [Product]) async throws
}

protocol ProductRemoteService {
    func fetchProducts() async throws -> [Product]
}

enum ProductRepositoryEvent {
    case cached([Product])
    case fresh([Product])
}

struct ProductRepository {
    let localStore: ProductLocalStore
    let remoteService: ProductRemoteService

    func loadProducts() -> AsyncThrowingStream<ProductRepositoryEvent, Error> {
        AsyncThrowingStream { continuation in
            Task {
                do {
                    let cached = try await localStore.loadProducts()
                    if !cached.isEmpty {
                        continuation.yield(.cached(cached))
                    }

                    let fresh = try await remoteService.fetchProducts()
                    try await localStore.saveProducts(fresh)
                    continuation.yield(.fresh(fresh))
                    continuation.finish()
                } catch {
                    continuation.finish(throwing: error)
                }
            }
        }
    }
}
```

### Senior Discussion

Clarify whether network failure after cached data should:

- Show cached data with a non-blocking error.
- Replace screen with error.
- Mark data as stale.

Often, the best UX is cached content plus a small "Could not refresh" message.

## Scenario 9: Pagination Logic

### Prompt

Prevent duplicate pagination requests and stop when there are no more pages.

### Solution

```swift
@MainActor
final class PaginationController<Item> {
    private(set) var items: [Item] = []
    private var nextCursor: String?
    private var isLoading = false
    private var hasLoadedFirstPage = false

    func loadNextPage(
        fetch: (String?) async throws -> Page<Item>
    ) async throws {
        guard !isLoading else { return }
        guard !hasLoadedFirstPage || nextCursor != nil else { return }

        isLoading = true
        defer { isLoading = false }

        let page = try await fetch(nextCursor)
        items.append(contentsOf: page.items)
        nextCursor = page.nextCursor
        hasLoadedFirstPage = true
    }
}

struct Page<Item> {
    let items: [Item]
    let nextCursor: String?
}
```

### Senior Discussion

Pagination bugs often come from duplicate requests, wrong cursor state, and replacing old items accidentally. Discuss refresh separately because refresh should usually reset cursor and replace items.

## Scenario 10: Retain Cycle In Cell Callback

### Prompt

Fix the memory/logic problem:

```swift
cell.onTap = {
    self.openDetails(indexPath: indexPath)
}
```

### Solution

```swift
let itemID = row.id
cell.onTap = { [weak self] in
    self?.openDetails(id: itemID)
}
```

### Senior Discussion

There are two bugs:

1. Strong `self` capture can retain the view controller.
2. Captured `indexPath` can become stale after insert, delete, filter, sort, or diffable updates.

Stable IDs are safer than index paths.

## Scenario 11: Error Mapping

### Prompt

Convert low-level errors into user-facing messages.

### Solution

```swift
enum APIError: Error {
    case offline
    case unauthorized
    case server(statusCode: Int)
    case decoding
    case unknown
}

enum UserFacingError: Equatable {
    case message(String)
    case requiresLogin
}

func mapError(_ error: APIError) -> UserFacingError {
    switch error {
    case .offline:
        return .message("You appear to be offline.")
    case .unauthorized:
        return .requiresLogin
    case .server:
        return .message("Something went wrong. Please try again.")
    case .decoding:
        return .message("We could not read the latest data.")
    case .unknown:
        return .message("Unexpected error occurred.")
    }
}
```

### Senior Discussion

Do not expose backend status codes directly to users. Keep technical error details in logs/analytics, and map to useful user-facing recovery paths.

## Scenario 12: Testable Date Grouping

### Prompt

Group transactions by day. Make it testable.

### Solution

```swift
struct Transaction {
    let id: String
    let date: Date
    let amount: Decimal
}

func groupTransactionsByDay(
    _ transactions: [Transaction],
    calendar: Calendar
) -> [Date: [Transaction]] {
    Dictionary(grouping: transactions) { transaction in
        calendar.startOfDay(for: transaction.date)
    }
}
```

### Senior Discussion

Inject `Calendar` instead of using `Calendar.current` directly. Tests can use a fixed calendar/time zone, preventing flaky date bugs.

