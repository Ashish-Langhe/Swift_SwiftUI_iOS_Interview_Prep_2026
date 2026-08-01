# Day 39: Data Modeling, Cache, Offline, And Sync

Data design is one of the most important MSD topics. Mobile apps must balance speed, correctness, offline behavior, storage limits, privacy, and old app compatibility.

## Data Model Types

Use separate models for separate responsibilities.

```text
API DTO -> Domain Model -> View State
```

### API DTO

Matches backend contract.

```swift
struct ProductDTO: Decodable {
    let id: String
    let title: String
    let priceCents: Int
    let currency: String
    let imageURL: URL?
}
```

### Domain Model

Represents app meaning.

```swift
struct Product: Identifiable, Equatable, Sendable {
    let id: String
    let title: String
    let price: Money
    let imageURL: URL?
}

struct Money: Equatable, Sendable {
    let minorUnits: Int
    let currencyCode: String
}
```

### View State

Represents screen state.

```swift
enum ProductDetailState: Equatable {
    case loading
    case loaded(Product)
    case offline(Product)
    case failed(message: String)
}
```

Senior reason:

```text
DTOs protect network compatibility. Domain models protect app logic. View state protects UI clarity.
```

## Cache Strategies

### Memory Cache

Use for short-lived objects:

- Images.
- Parsed feed sections.
- Recent search suggestions.

Pros:

- Very fast.

Cons:

- Lost when app exits.
- Must react to memory pressure.

### Disk Cache

Use for:

- Feed snapshots.
- Product details.
- Profile data.
- Image bytes.
- Offline content.

Pros:

- Survives app restart.

Cons:

- Needs eviction policy.
- Sensitive data must be protected or avoided.

### Database

Use SQLite, Core Data, SwiftData, or another local database when:

- Data is relational or query-heavy.
- Offline edits are needed.
- Pagination needs merging.
- Search/filtering is local.

## Stale-While-Revalidate

This is a common mobile pattern:

1. Show cached data immediately.
2. Fetch fresh data in background.
3. Update UI when new data arrives.
4. Mark stale state if refresh fails.

Example:

```swift
@MainActor
func loadHome() {
    Task {
        if let cached = await repository.cachedHome() {
            state = .loaded(cached, freshness: .stale)
        } else {
            state = .loading
        }

        do {
            let fresh = try await repository.fetchHome()
            state = .loaded(fresh, freshness: .fresh)
        } catch {
            if state.isLoaded == false {
                state = .failed("Could not load home.")
            }
        }
    }
}
```

## Offline Write Queue

Offline writes should be designed carefully.

Good candidates:

- Draft notes.
- Likes.
- Saves/bookmarks.
- Chat messages.
- Analytics events.
- Non-critical preferences.

Bad candidates:

- Final payment capture.
- Banking transfer approval.
- Inventory reservation.
- Fraud-sensitive actions.

Queue item example:

```swift
struct PendingMutation: Codable, Identifiable, Sendable {
    let id: UUID
    let kind: MutationKind
    let payload: Data
    let idempotencyKey: String
    var attemptCount: Int
    let createdAt: Date
}

enum MutationKind: String, Codable, Sendable {
    case likePost
    case sendMessage
    case updateProfile
}
```

## Idempotency

Retries can duplicate writes unless the server can identify repeated attempts.

Use an idempotency key for operations like:

- Place order.
- Send message.
- Create payment intent.
- Submit form.

Example:

```swift
struct SendMessageRequest: Encodable {
    let conversationID: String
    let clientMessageID: UUID
    let text: String
}
```

The server treats the same `clientMessageID` as the same message.

## Conflict Resolution

Conflicts happen when:

- Multiple devices edit the same data.
- Offline edits sync later.
- Server state changes while client is stale.

Strategies:

- Last write wins: simple but may lose data.
- Server wins: safe for critical domains.
- Client merge: useful for documents or drafts.
- Manual conflict UI: useful when correctness matters.
- CRDT-like models: useful for collaborative editing, but complex.

Senior example:

```text
For a notes app, I may allow local edits offline and merge by updated field or show conflict UI. For banking, I will not resolve money movement on the client; server state wins.
```

## Pagination And Cache Merging

For feeds:

- Use cursor-based pagination.
- Store item IDs in order.
- Deduplicate by stable ID.
- Keep item data separate from feed ordering.
- Handle deleted or hidden items.

Simple model:

```swift
struct FeedPage: Decodable {
    let items: [FeedItemDTO]
    let nextCursor: String?
}

struct CachedFeed {
    var orderedIDs: [String]
    var itemsByID: [String: FeedItem]
    var nextCursor: String?
}
```

## Security And Privacy In Local Data

Do not cache everything.

Avoid storing:

- Raw access tokens outside Keychain.
- CVV, OTP, PIN.
- Highly sensitive banking details unless required and protected.
- Unredacted PII in logs.

Use:

- Keychain for secrets.
- File protection for sensitive files.
- Encryption where required.
- Data retention policies.

## Interview Notes

- Cache is a product decision, not only a technical detail.
- Offline mode must define which actions are allowed.
- Idempotency is a senior-level keyword for retries.
- Separate DTO, domain, and view state.
- Always discuss stale data and conflict resolution.
