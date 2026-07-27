# Day 38: iOS System Design Mindset

iOS system design is not only backend architecture. In mobile interviews, system design means designing the full product experience across the iOS app, backend services, APIs, caching, offline behavior, security, analytics, releases, observability, performance, and failure handling.

For a junior engineer, system design means understanding screens, APIs, models, and basic app flow. For a senior engineer, it means making product and engineering tradeoffs under real-world constraints.

## What iOS System Design Includes

An iOS system design answer should cover:

- Product requirements.
- User flows.
- Client architecture.
- Backend/service boundaries.
- API contracts.
- Local storage.
- Cache strategy.
- Offline behavior.
- Sync strategy.
- Authentication.
- Security and privacy.
- Push notifications.
- Deep links.
- Analytics and observability.
- Performance.
- Scalability.
- Failure handling.
- Release and experimentation.

## HLD vs LLD In iOS

### High-Level Design

HLD answers:

- What are the major components?
- How does the app communicate with backend services?
- What services exist?
- What data flows between them?
- How does the system scale?
- Where are caches?
- How do we handle failure?

Example components:

```text
iOS App
API Gateway
Auth Service
Catalog Service
Recommendation Service
Search Service
Order Service
Payment Service
Notification Service
Analytics Pipeline
CDN
Local Cache
```

### Low-Level Design

LLD answers:

- What modules/classes/protocols exist in the iOS app?
- What is the state model?
- What is the screen architecture?
- How does data move from API to UI?
- How are errors modeled?
- How are dependencies injected?
- How is the feature tested?

Example iOS LLD:

```text
ProductListView
ProductListViewModel
ProductRepository
ProductAPIClient
ProductLocalStore
ProductDTO
Product
ProductRow
ProductCoordinator
```

## System Design Interview Flow

Use this flow:

1. Clarify requirements.
2. Define users and main flows.
3. Define scale and constraints.
4. Draw high-level architecture.
5. Define APIs and data models.
6. Define iOS app architecture.
7. Discuss caching/offline/sync.
8. Discuss security/privacy.
9. Discuss observability/performance.
10. Discuss tradeoffs and future extensions.

## Junior-Level Thinking

A junior answer should explain:

- Screens needed.
- APIs needed.
- Basic data models.
- How data appears in UI.
- Basic error/loading/empty states.

Example:

```text
For an e-commerce product list, I need a product list screen, product detail screen, cart screen, and checkout screen. The app fetches products from API, decodes JSON into models, shows loading and error states, and stores cart locally or on server depending on login state.
```

## Senior-Level Thinking

A senior answer should explain:

- Ownership boundaries.
- API versioning.
- Cache policy.
- Offline behavior.
- Concurrency and cancellation.
- Token refresh.
- Pagination.
- Analytics.
- Feature flags.
- Performance budgets.
- Test strategy.
- Failure modes.

Example:

```text
For a product list, I would show cached products immediately, refresh from network, preserve scroll position, cancel stale searches, paginate with cursor, separate first-page failure from pagination failure, and emit analytics for impressions and taps without blocking UI.
```

## Architect-Level Thinking

An architect answer also includes:

- How teams can own modules independently.
- How contracts evolve.
- How experiments are rolled out.
- How incidents are detected.
- How app releases coordinate with backend changes.
- How privacy and compliance affect design.
- How data consistency is managed.

## iOS-Specific Design Questions

Always ask:

- What happens on poor network?
- Should the app work offline?
- What data can be cached?
- What data must never be cached?
- What happens after logout?
- How do we handle token expiry?
- Can multiple devices modify same data?
- How do we avoid stale UI?
- What is the stable identity for list rows?
- How do we measure performance and crashes?

## Common System Design Mistakes

- Designing only backend and ignoring the app.
- Ignoring offline behavior.
- Ignoring token refresh.
- Ignoring pagination.
- Ignoring app version compatibility.
- Returning raw DTOs to UI.
- Treating all errors the same.
- Forgetting analytics and observability.
- Not discussing security.
- Not distinguishing cache from source of truth.

## Senior iOS System Design Template

Use this answer shape:

```text
Requirement:
What are we building and for whom?

Core flows:
What are the main user actions?

HLD:
What services and client components exist?

LLD:
What iOS modules/classes/models are needed?

Data:
What API contracts, local models, and view models exist?

State:
How does UI represent loading, success, empty, failure, refresh, pagination?

Offline/cache:
What is stored locally and when is it invalidated?

Security:
How are auth, secrets, sensitive data, and privacy handled?

Performance:
How do we keep UI fast?

Observability:
How do we measure errors, latency, crashes, and user flows?

Tradeoffs:
What alternatives did we reject and why?
```

