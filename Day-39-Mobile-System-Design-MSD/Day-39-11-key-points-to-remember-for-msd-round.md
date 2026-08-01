# Day 39: Key Points To Remember For MSD Round

This file is a senior iOS developer's last-minute MSD checklist. Read it before a mobile system design round when you want to sound structured, practical, and production-minded.

The goal is not to dump every possible component. The goal is to show that you can design a mobile product that works under real constraints: slow network, app termination, stale data, retries, auth expiry, memory pressure, battery cost, old app versions, observability gaps, and release risk.

## 1. Start From The Mobile Client

In MSD, begin from the iOS app and expand outward.

Strong opening:

```text
I will design this from the iOS client outward: user flows, app modules, state ownership, API contracts, local cache/offline behavior, realtime updates, security, performance, observability, and rollout strategy.
```

Why it matters:

- It makes your answer mobile-specific.
- It prevents you from sounding like a backend-only system design answer.
- It gives the interviewer a map of your thinking.

Use case:

```text
For Instagram feed, I will first define feed loading, pagination, media rendering, like/save actions, comments entry point, cached feed on launch, and refresh behavior before discussing ranking services or databases.
```

## 2. Clarify Scope Before Designing

Always ask what is included.

Ask:

- Which user role are we designing for?
- Is this iPhone only or iPad/widgets/watch too?
- Is offline support required?
- Is realtime required?
- Is the domain sensitive: payments, banking, health, identity?
- Are we designing v1 or large-scale production?
- What iOS versions are supported?

Senior phrasing:

```text
I will scope this to the consumer iOS app and the core user flow. I will mention backend touchpoints where they affect the mobile contract, but I will not deeply design admin tooling unless needed.
```

## 3. Define The Core User Flows

Do not jump directly into classes.

Write 3-5 flows:

```text
Launch -> Session restore -> Cached home -> Fresh refresh
Search -> Debounce -> Results -> Pagination -> Detail
Action -> Optimistic UI -> API -> Confirm/Rollback
Push -> Auth gate -> Deep link router -> Target screen
Offline edit -> Local queue -> Retry -> Reconcile
```

Why it matters:

- Flows reveal screens.
- Screens reveal state.
- State reveals modules.
- Modules reveal APIs.
- APIs reveal failure cases.

## 4. Mention Mobile Constraints Early

Mobile constraints are the heart of MSD.

Remember:

- Network is unreliable.
- App can be backgrounded or killed.
- Push is not guaranteed.
- Background work is limited.
- Battery and memory matter.
- Old app versions remain active.
- Permissions can be denied.
- App Store release is slower than backend release.

Senior line:

```text
Because mobile clients cannot be upgraded instantly, I would design backward-compatible API contracts and use feature flags for risky rollout.
```

## 5. Separate HLD And LLD Clearly

HLD explains the ecosystem.

```text
iOS App -> Mobile BFF/API Gateway -> Domain Services
iOS App -> CDN for media
Push Provider -> iOS App
Remote Config -> iOS App
iOS App -> Analytics/Crash/Logs
```

LLD explains app internals.

```text
View -> ViewModel/Reducer -> UseCase -> Repository -> API Client / Local Store
```

Senior tip:

```text
When moving from HLD to LLD, explicitly say: "Now I will zoom into the iOS app internals."
```

## 6. Use Source Of Truth Language

Senior MSD answers must identify who owns truth.

Examples:

- Banking balance: server source of truth.
- Product catalog: server source of truth, client cache for speed.
- Draft note: local source of truth until synced.
- Chat message before send: local pending source, server confirms final state.
- Feature flags: remote config service source of truth.
- UI selection state: local view/feature state.

Use case:

```text
For cart, I may show optimistic local updates for responsiveness, but server remains source of truth because price, inventory, offers, and eligibility can change.
```

## 7. Model UI State Explicitly

Avoid only `isLoading` and `error`.

Better:

```swift
enum FeedState: Equatable {
    case idle
    case loading
    case loaded(items: [FeedItem], freshness: Freshness)
    case empty
    case offline(items: [FeedItem])
    case failed(message: String)
}

enum Freshness: Equatable {
    case fresh
    case stale(lastUpdated: Date)
}
```

Why it matters:

- Real apps have stale, offline, empty, partial, and retrying states.
- Explicit state reduces UI bugs.
- It gives better interview discussion around user experience.

## 8. Separate DTO, Domain, And View Models

Do not leak API DTOs everywhere.

```text
DTO: API response shape
Domain model: app business meaning
View model/state: UI rendering shape
```

Example:

```swift
struct TransactionDTO: Decodable {
    let id: String
    let amount: Int
    let currency: String
    let status: String
}

struct Transaction: Identifiable, Equatable, Sendable {
    let id: String
    let amount: Money
    let status: TransactionStatus
}

enum TransactionStatus: Equatable, Sendable {
    case pending
    case completed
    case failed
    case unknown(String)
}
```

Senior reason:

```text
DTOs protect network compatibility. Domain models protect app logic. View state protects UI clarity.
```

## 9. Always Discuss Cache Policy

Cache is not just "save data locally." Define what, where, when, and for how long.

Questions:

- What data can be cached?
- Is cached data sensitive?
- How stale can it be?
- What invalidates it?
- Is the cache memory-only, disk, or database?
- Does user logout clear it?

Examples:

- Product images: memory and disk cache.
- Home feed: disk snapshot plus refresh.
- Bank balance: cautious cache with last-updated timestamp or no cache depending policy.
- Auth token: Keychain only.
- OTP/CVV/PIN: never cache.

## 10. Use Stale-While-Revalidate For Read-Heavy Screens

Pattern:

1. Show cached data fast.
2. Mark it stale if needed.
3. Fetch fresh data.
4. Replace cache and UI.
5. Keep usable UI if refresh fails.

Use cases:

- Feed.
- Product detail.
- Profile.
- Search suggestions.
- Order history.

Senior phrasing:

```text
For read-heavy non-sensitive data, I would prefer stale-while-revalidate because it improves perceived performance while still converging to fresh server data.
```

## 11. Be Careful With Offline Writes

Offline writes are not always allowed.

Good candidates:

- Draft notes.
- Draft messages.
- Likes/bookmarks.
- Analytics.
- Profile edits if conflict rules are clear.

Poor candidates:

- Money transfer.
- Payment capture.
- Inventory reservation.
- High-risk identity changes.

Solution:

```text
For safe writes, I would queue mutations locally with idempotency keys and retry when network returns. For critical writes, I would require server confirmation before showing final success.
```

## 12. Mention Idempotency For Retries

Retries can create duplicate writes.

Use idempotency keys for:

- Send message.
- Place order.
- Create booking.
- Submit payment intent.
- Upload completion.

Example:

```swift
struct PlaceOrderRequest: Encodable {
    let cartID: String
    let addressID: String
    let paymentMethodID: String
    let idempotencyKey: String
}
```

Senior point:

```text
The retry should be safe even if the first request reached the server but the response was lost.
```

## 13. Choose Realtime Technology Intentionally

Do not say WebSocket for everything.

Use push when:

- App may be backgrounded.
- Event frequency is low.
- User needs notification.

Use WebSocket when:

- App is foregrounded.
- Low latency matters.
- Updates are frequent.

Use polling when:

- Simplicity is enough.
- Realtime is not strict.
- Backend does not support streaming.

Use case:

```text
For Uber tracking, I would use REST for initial trip snapshot, WebSocket for active tracking, push for background status changes, and polling fallback if realtime fails.
```

## 14. Respect iOS Background Limits

Do not claim the app can run forever in the background.

Use:

- Background URLSession for uploads/downloads.
- BGTaskScheduler for opportunistic refresh.
- Silent push as a hint, not guarantee.
- Local persistence to resume after termination.

Senior line:

```text
I would design background sync as resumable because iOS may delay or terminate background execution.
```

## 15. Protect The Main Thread

Mention main-thread safety in every performance-heavy design.

Move off main:

- JSON decoding for large payloads.
- Image downsampling.
- Database queries.
- File I/O.
- Expensive formatting.

Keep on main:

- UI state updates.
- Navigation.
- View rendering.

Swift concurrency note:

```swift
@MainActor
final class FeedViewModel: ObservableObject {
    @Published private(set) var state: FeedState = .idle
}
```

Senior add-on:

```text
Shared mutable services like cache writers or sync queues can be actor-isolated to avoid data races under Swift concurrency.
```

## 16. Include Performance Metrics

Do not say "make it fast." Define measurable targets.

Metrics:

- Cold launch time.
- Warm launch time.
- Screen load p95.
- API latency p95.
- Scroll hitch rate.
- Memory footprint.
- Crash-free sessions.
- Battery-heavy operation duration.
- Image cache hit rate.
- Sync queue failure rate.

Use case:

```text
For a feed app, I would monitor time to first content, pagination latency, image load failures, scroll hitching, memory warnings, and impression event batching.
```

## 17. Include Security And Privacy

Minimum senior coverage:

- Keychain for tokens.
- ATS enabled.
- Certificate pinning for high-risk apps when operationally ready.
- Biometric gate for sensitive local access.
- Server-side authorization.
- PII redaction in logs.
- Privacy manifests and permission purpose strings.
- Clear local data on logout where appropriate.

Banking answer:

```text
The app can cache the last known balance only if policy allows it, protected and clearly marked with last updated time. Transaction approval remains server-authoritative and may require step-up authentication.
```

## 18. Observability Is A Senior Differentiator

Mention:

- Crash reporting.
- Non-fatal errors.
- API latency.
- Screen load events.
- Feature flag state.
- App version.
- OS version.
- Device class.
- Network type.
- Request IDs.

Good event shape:

```text
event: order_tracking_failed
properties: app_version, os_version, trip_state, network_type, error_family, request_id
```

Avoid:

- Raw tokens.
- OTP.
- Card details.
- Private messages.
- Exact location history unless required and consented.

## 19. Explain Rollout Strategy

Mobile release risk is real.

Use:

- Dark launch.
- Feature flags.
- Remote config.
- Internal rollout.
- Phased rollout.
- Kill switch.
- Backward-compatible APIs.
- Monitoring by app version.

Senior answer:

```text
I would ship the new flow behind a feature flag, enable it for internal users, then small percentages, and watch crash rate, conversion, API errors, and support tickets before full rollout.
```

## 20. Mention Testing Strategy

Testing is part of design.

Cover:

- Unit tests for view models/reducers.
- Repository tests with mocked API/local store.
- API contract tests.
- Snapshot/UI tests for critical screens.
- Offline/retry tests.
- Deep link tests.
- Performance tests for launch or scrolling.
- Security tests for sensitive flows.

Senior note:

```text
I would test state machines directly because they capture most of the app's correctness.
```

## 21. Use State Machines For Complex Flows

Useful for:

- Checkout.
- Ride tracking.
- Order delivery.
- Upload.
- Authentication.
- Banking transfer.

Example:

```swift
enum CheckoutState: Equatable {
    case reviewingCart
    case validatingInventory
    case selectingPayment
    case authorizingPayment
    case placingOrder
    case confirmed(orderID: String)
    case failed(reason: String)
}
```

Why it helps:

- Prevents invalid transitions.
- Makes testing easier.
- Makes interview answers clearer.

## 22. Prepare Common Tradeoff Lines

REST vs GraphQL:

```text
REST is simpler and cache-friendly. GraphQL reduces over-fetching for dynamic screens but needs stronger schema and caching discipline.
```

BFF vs direct services:

```text
BFF gives mobile-optimized contracts and protects old clients, but adds another backend layer to own.
```

Cache vs fresh fetch:

```text
Cache improves speed and offline UX, but creates stale-data and invalidation complexity.
```

Optimistic UI vs server confirmation:

```text
Optimistic UI improves responsiveness for low-risk actions, but critical actions need server confirmation before final success.
```

WebSocket vs push:

```text
WebSocket is better for active low-latency sessions. Push is better for background event notification but is not guaranteed.
```

## 23. Common MSD Prompt Solutions

### Design Instagram Feed

Key solution points:

- Server-ranked cursor feed.
- Local feed snapshot.
- CDN media.
- Image/video prefetch with cancellation.
- Optimistic like/save.
- Impression analytics batching.
- Pull-to-refresh.
- Pagination deduplication.
- Stale-while-revalidate.

### Design WhatsApp Chat

Key solution points:

- Local message database.
- Client message IDs.
- Optimistic sending.
- Delivery/read states.
- WebSocket foreground.
- Push background.
- Media upload pipeline.
- Pagination from latest.
- Encryption discussion if in scope.

### Design Amazon Checkout

Key solution points:

- Cart server source of truth.
- Optimistic cart changes allowed.
- Price/inventory revalidation before order.
- Payment authorization server-driven.
- Idempotency key for place order.
- Order state machine.
- Feature-flagged checkout rollout.

### Design Uber Tracking

Key solution points:

- Initial trip snapshot through REST.
- WebSocket active tracking.
- Push status changes.
- Poll fallback.
- Throttled map updates.
- Battery-aware location policy.
- Trip state machine.
- Local last known trip state.

### Design Banking Transfer

Key solution points:

- Strong auth.
- Keychain token storage.
- Biometric/OTP step-up.
- Server-authoritative balance and transfer state.
- No unsafe offline transfer.
- Idempotency for submit.
- Audit logs.
- Privacy-safe analytics.
- Session timeout.

## 24. Final Two-Minute Summary Template

Use this at the end:

```text
To summarize, I designed the app from the mobile client outward. The iOS app has clear feature modules, explicit state, repository boundaries, local cache where safe, and server-authoritative handling for critical operations. The system uses a BFF/API layer for mobile contracts, CDN for media, push/WebSocket/polling depending on realtime needs, and feature flags for rollout. I covered offline behavior, idempotent retries, security, privacy, performance, observability, and old app compatibility. The main tradeoff is balancing freshness, correctness, battery, and user-perceived speed.
```

## 25. Final Checklist Before You Stop Speaking

Make sure you said:

- Requirements and assumptions.
- Core flows.
- Mobile constraints.
- HLD.
- iOS LLD.
- API contract.
- Cache/offline/sync.
- Realtime choice.
- Security/privacy.
- Performance/memory/battery.
- Observability.
- Testing.
- Release strategy.
- Tradeoffs.

## 26. Understand What The Interviewer Is Scoring

For senior iOS roles, the interviewer is usually not checking whether you know every backend buzzword. They are checking whether you can make reliable product decisions under mobile constraints.

They want to hear:

- Clear scoping.
- Practical user flows.
- Ownership boundaries.
- Strong iOS architecture.
- Correctness for critical actions.
- Caching and offline judgment.
- Performance awareness.
- Security and privacy discipline.
- Rollout and observability thinking.
- Mature tradeoff explanations.

Weak senior signal:

```text
I will use MVVM, URLSession, Core Data, WebSocket, and push notification.
```

Strong senior signal:

```text
I will use MVVM or reducer-style state depending on feature complexity. The repository will hide network and local store decisions. Reads can use stale-while-revalidate, but writes need domain-specific rules. For chat, optimistic writes are acceptable with client IDs. For banking transfer, final success must be server-confirmed.
```

## 27. Convert Requirements Into Architecture Decisions

Every requirement should map to a design choice.

| Requirement | Mobile Design Decision |
| --- | --- |
| Fast first screen | Cache snapshot, skeleton UI, defer non-critical work |
| Offline browsing | Disk cache or database, freshness indicator |
| Offline editing | Mutation queue, idempotency key, sync engine |
| Realtime tracking | REST snapshot plus WebSocket/polling/push blend |
| Sensitive data | Keychain, protected storage, redacted logs |
| Old app support | Backward-compatible API, optional decoding, feature flags |
| Large media | CDN, downsampling, cancellation, prefetch budget |
| High reliability | State machine, retry policy, observability |
| Global app | Localization, regional config, latency-aware APIs |
| Experimentation | Remote config, analytics, phased rollout |

Use this sentence:

```text
This requirement affects the client architecture because it changes state ownership, caching policy, and failure recovery.
```

## 28. Know When To Use BFF In MSD

A mobile BFF is valuable when one screen needs data from multiple backend services or when mobile apps need stable, versioned contracts.

Use BFF for:

- Home screen composition.
- Feed cards with heterogeneous layouts.
- Checkout summary.
- Banking dashboard.
- Personalized marketplace home.
- App-version-specific response shaping.

Avoid BFF when:

- The app is small.
- The domain API already matches screen needs.
- The team cannot own the extra service.
- It becomes a place for random business logic.

Senior answer:

```text
I would use a mobile BFF for the home/dashboard surface because it allows server-driven composition and protects the app from backend service fragmentation. For simple resource screens, direct REST endpoints may be enough.
```

## 29. Design For Old App Versions

Old app compatibility is a major mobile system design topic.

Practical rules:

- Never remove required fields suddenly.
- Add optional fields.
- Use unknown enum fallback.
- Version breaking API changes.
- Keep old endpoints until usage drops.
- Gate new features by app version.
- Use capability negotiation for risky features.

Swift example:

```swift
struct HomeResponse: Decodable {
    let sections: [HomeSectionDTO]
    let minimumSupportedAppVersion: String?
    let capabilities: [String]
}
```

Senior explanation:

```text
Backend can deploy instantly, but iOS clients remain in the wild. I would avoid server contracts that assume all clients upgraded, especially for checkout, auth, and dashboard flows.
```

## 30. Design API Contracts For Mobile Screens

A good mobile API is not always a pure database-resource API. Sometimes it should be screen-shaped.

For feed:

```json
{
  "items": [],
  "nextCursor": "abc",
  "trackingToken": "feed-session-123",
  "generatedAt": "2026-08-02T10:00:00Z"
}
```

For checkout:

```json
{
  "cartId": "cart_123",
  "items": [],
  "priceSummary": {},
  "availablePaymentMethods": [],
  "warnings": ["PRICE_CHANGED"],
  "idempotencyKeyRequired": true
}
```

For ride tracking:

```json
{
  "tripId": "trip_123",
  "state": "driver_arriving",
  "driver": {},
  "vehicle": {},
  "route": {},
  "lastKnownLocation": {},
  "realtimeChannel": {}
}
```

Senior tip:

```text
Screen-shaped APIs reduce app orchestration and improve latency, but they should remain versioned and not leak backend service complexity.
```

## 31. Use Error Taxonomy, Not Generic Errors

Senior iOS engineers model error families.

Useful categories:

- Network unavailable.
- Timeout.
- Unauthorized.
- Forbidden.
- Validation failed.
- Conflict.
- Rate limited.
- Server unavailable.
- Decoding/contract issue.
- Operation not allowed offline.

Example:

```swift
enum AppError: Error, Equatable {
    case offline
    case timeout
    case unauthorized
    case forbidden
    case validation(message: String)
    case conflict(message: String)
    case serverUnavailable
    case contractMismatch
    case unknown
}
```

Why it matters:

- Offline can show cached content.
- Unauthorized can trigger refresh/login.
- Validation can show field errors.
- Conflict can trigger reconciliation.
- Server unavailable can retry with backoff.

## 32. Explain Optimistic UI Carefully

Optimistic UI is useful, but it is not universally correct.

Good optimistic actions:

- Like post.
- Save item.
- Add non-critical reaction.
- Send chat message with pending state.
- Toggle local preference.

Risky optimistic actions:

- Payment success.
- Bank transfer completion.
- Inventory reservation.
- Identity verification.
- Security setting changes.

Pattern:

```text
UI updates optimistically -> local pending state -> API request -> confirmed or rollback.
```

Swift state example:

```swift
enum LikeState: Equatable {
    case notLiked
    case liked
    case updating(previous: Bool)
    case failed(previous: Bool, message: String)
}
```

Senior line:

```text
Optimistic UI should improve perceived speed without lying about final business correctness.
```

## 33. Design Deep Links And Push Routing

Deep links are often missed in MSD rounds.

Flow:

```text
URL/Push -> Parse -> Validate -> Auth Gate -> Fetch Required Data -> Navigate -> Track Result
```

Cases to handle:

- User logged out.
- Target object deleted.
- User lacks permission.
- App cold launched.
- App already on another flow.
- Required data not cached.
- Link version is old.

Example route:

```swift
enum DeepLinkRoute: Equatable {
    case product(id: String)
    case order(id: String)
    case chat(conversationID: String)
    case transferApproval(id: String)
}
```

Senior explanation:

```text
Deep links should not directly mutate navigation from random services. I would route through an app-level coordinator or root state so auth, permissions, and data loading are handled consistently.
```

## 34. Plan For App Lifecycle

Mention what happens across lifecycle events.

On launch:

- Restore session.
- Load cached critical state.
- Resume pending tasks.
- Refresh remote config.
- Reconcile sync queue.

On foreground:

- Refresh stale data.
- Reconnect realtime if needed.
- Re-check permissions.
- Flush pending analytics.

On background:

- Persist important state.
- Pause expensive updates.
- Close or downgrade realtime connections.
- Schedule background work if appropriate.

On termination:

- Assume in-memory state is gone.
- Recover from local durable state next launch.

Senior answer:

```text
I would persist pending mutations and last known screen-critical state because iOS can terminate the app before async work completes.
```

## 35. Mention Accessibility And Localization In Design

MSD is not only networking.

For production iOS:

- Dynamic Type support.
- VoiceOver labels.
- Reduced motion.
- Color contrast.
- Localized strings.
- Right-to-left layout.
- Locale-aware currency/date/number formatting.

Use case:

```text
For checkout and banking, currency formatting must be locale-aware and accessible. Error messages should be localized and announced correctly for assistive technologies.
```

Senior signal:

```text
I would include accessibility and localization in acceptance criteria, not treat them as final polish.
```

## 36. Design Image And Media Loading Like A System

For feed, e-commerce, video, and social apps, media loading is a system design problem.

Key pieces:

- CDN URLs.
- Size variants.
- Memory cache.
- Disk cache.
- Request coalescing.
- Cancellation.
- Priority.
- Downsampling.
- Placeholder/error image.
- Prefetch budget.
- Memory warning handling.

Example:

```text
For a product grid, I would request thumbnail-sized images, cancel image tasks when cells are reused, downsample off the main thread, and avoid prefetching too far on cellular networks.
```

Senior tradeoff:

```text
Aggressive prefetch improves smoothness but increases network, battery, and memory cost, so it should be bounded and adaptive.
```

## 37. Design Sync As A State Machine

Offline sync should not be hand-waved.

Sync states:

```swift
enum SyncState: Equatable {
    case idle
    case pending(count: Int)
    case syncing(current: Int, total: Int)
    case failedRetryable(reason: String)
    case blockedRequiresUserAction(reason: String)
}
```

Mutation fields:

- Local mutation ID.
- Idempotency key.
- Entity ID.
- Operation type.
- Payload.
- Created time.
- Attempt count.
- Last error.

Senior decision:

```text
I would make the sync engine own retries and ordering. Feature view models should only observe sync state, not manually replay network calls.
```

## 38. Know Data Store Selection

Choose storage intentionally.

| Storage | Use For | Avoid For |
| --- | --- | --- |
| UserDefaults | Small preferences, flags | Secrets, large data, relational data |
| Keychain | Tokens, credentials, sensitive keys | Large objects, frequent writes |
| FileManager | Files, downloads, media | Query-heavy relational data |
| SQLite/Core Data/SwiftData | Offline domain data, queries, sync | Tiny settings |
| Memory cache | Short-lived speed | Durable state |

Senior example:

```text
For chat, I would use a local database because messages need pagination, search, offline access, and delivery-state updates. UserDefaults would be the wrong storage choice.
```

## 39. Prepare Domain-Specific Correctness Rules

Different apps optimize for different things.

Feed:

- Perceived speed and smooth scrolling.
- Eventual consistency is acceptable.

Chat:

- Local durability, ordering, delivery states.
- Eventual consistency is acceptable, but message loss is not.

E-commerce:

- Responsive browse.
- Server confirmation for price, inventory, and payment.

Ride tracking:

- Low-latency updates.
- Battery-aware location and map rendering.

Banking:

- Correctness, auditability, privacy, and security.
- Offline final actions usually not allowed.

Use this line:

```text
I would tune the architecture based on domain correctness. A like and a bank transfer should not have the same consistency model.
```

## 40. Add A Small LLD Example When Asked For Depth

If the interviewer asks for implementation depth, show one focused slice.

Example: order tracking repository.

```swift
protocol OrderTrackingRepository {
    func cachedOrder(id: Order.ID) async -> Order?
    func fetchOrder(id: Order.ID) async throws -> Order
    func observeOrderUpdates(id: Order.ID) -> AsyncStream<OrderUpdate>
}

@MainActor
final class OrderTrackingViewModel: ObservableObject {
    @Published private(set) var state: OrderTrackingState = .loading

    private let repository: OrderTrackingRepository
    private var updatesTask: Task<Void, Never>?

    init(repository: OrderTrackingRepository) {
        self.repository = repository
    }

    func load(id: Order.ID) {
        updatesTask?.cancel()

        updatesTask = Task {
            if let cached = await repository.cachedOrder(id: id) {
                state = .loaded(cached, freshness: .stale)
            }

            do {
                let fresh = try await repository.fetchOrder(id: id)
                state = .loaded(fresh, freshness: .fresh)

                for await update in repository.observeOrderUpdates(id: id) {
                    guard !Task.isCancelled else { return }
                    state = state.applying(update)
                }
            } catch {
                state = .failed("Unable to track this order right now.")
            }
        }
    }

    func stop() {
        updatesTask?.cancel()
        updatesTask = nil
    }
}
```

Explain:

- Cached state improves launch speed.
- Fetch gives authoritative snapshot.
- Async stream handles realtime updates.
- Task cancellation protects lifecycle.
- UI state stays main-actor isolated.

## 41. Senior-Level "When Not To Use" Answers

Interviewers like restraint.

Do not use WebSocket when:

- Updates are rare.
- Push or polling is enough.
- Battery matters more than latency.

Do not use Core Data/SwiftData when:

- The data is tiny.
- No querying/offline behavior exists.
- A simple file or memory cache is enough.

Do not use GraphQL when:

- Team lacks schema governance.
- Caching must be very simple.
- REST endpoints already match mobile screens.

Do not use optimistic UI when:

- The action is irreversible.
- User trust depends on confirmed correctness.
- Regulatory or financial rules apply.

Do not cache sensitive data when:

- The UX benefit is small.
- Policy forbids it.
- The data can become harmful if stale.

## 42. Senior Answer Templates For Tradeoffs

Use these short templates.

For correctness:

```text
I would keep the server authoritative because this action affects global business state.
```

For UX speed:

```text
I would show cached data first because this screen is read-heavy and can tolerate short-lived staleness.
```

For offline:

```text
I would support offline for drafts and low-risk mutations, but I would block final critical actions until server confirmation.
```

For rollout:

```text
I would ship the code dark and enable it gradually with metrics by app version and feature flag.
```

For observability:

```text
I would add request IDs and feature flag state to logs so backend and iOS failures can be correlated.
```

## 43. How To Recover If You Get Stuck

If you freeze, return to the framework:

```text
Let me structure this by flow, data, communication, local state, and failure handling.
```

Then say:

- What is the primary user flow?
- What data is needed?
- Is data fresh, cached, or realtime?
- Who owns source of truth?
- What can fail?
- How does the app recover?

This is often enough to regain control.

## 44. What Makes An Answer Sound Senior

Senior answers:

- Explain why each choice exists.
- Say what they are not designing.
- Keep user experience connected to system behavior.
- Discuss mobile lifecycle.
- Mention old app versions.
- Avoid unsafe offline claims.
- Add observability and rollout.
- Give tradeoffs with domain context.

Example:

```text
For banking, I would accept slower refresh if it improves correctness and auditability. For feed, I would accept brief staleness to improve launch speed.
```

## 45. What Makes An Answer Sound Junior

Avoid:

- Listing technologies without decisions.
- Saying "I will call API" for everything.
- Ignoring failure states.
- Ignoring lifecycle and background limits.
- Treating cache as always safe.
- Treating push as guaranteed.
- Forgetting old app versions.
- Letting view models own networking, persistence, analytics, and navigation all together.

Better phrasing:

```text
The view model owns screen state. The repository owns data access. The sync engine owns retries. The server owns final correctness for critical actions.
```

## Points To Remember

- MSD is not backend-only system design.
- The iOS app is part of the distributed system.
- Mobile constraints should appear early in your answer.
- Use state machines for complex flows.
- Use idempotency for safe retries.
- Treat push as a signal, not guaranteed truth.
- Server owns critical business correctness.
- Cache improves UX but creates stale-data risk.
- Observability and rollout make your answer senior-level.
- End with a crisp tradeoff summary.
