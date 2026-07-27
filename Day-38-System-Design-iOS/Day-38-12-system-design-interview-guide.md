# Day 38: iOS System Design Interview Guide

This guide helps you answer system design questions from junior to senior level. Use it as a playbook for interviews.

## Common iOS System Design Questions

1. Design YouTube for iOS.
2. Design Netflix for iOS.
3. Design Amazon/Flipkart for iOS.
4. Design Instagram feed.
5. Design WhatsApp chat.
6. Design Uber ride tracking.
7. Design Swiggy/Zomato food delivery.
8. Design a banking app.
9. Design a notes app with offline sync.
10. Design an image loading library.
11. Design a video player module.
12. Design token refresh.
13. Design search with debounce and pagination.
14. Design push notification routing.
15. Design feature flags.

## Universal Answer Framework

```text
1. Clarify requirements.
2. Identify core flows.
3. Define HLD components.
4. Define iOS LLD modules.
5. Define data models and API contracts.
6. Define state management.
7. Define cache/offline/sync behavior.
8. Define security/privacy.
9. Define performance and observability.
10. Discuss tradeoffs.
```

## Junior Answer Expectations

A junior candidate should cover:

- Main screens.
- Basic APIs.
- Models.
- Loading/error states.
- Navigation.
- Basic cache.

Example:

```text
For a shopping app, I would create product list, product detail, cart, checkout, and orders screens. The app fetches products from API, decodes JSON, shows loading and error states, and lets users add items to cart.
```

## Mid-Level Answer Expectations

Mid-level should add:

- Pagination.
- Search cancellation.
- Repository layer.
- DTO/domain/view model mapping.
- Token refresh.
- Local cache.
- Unit tests.
- Analytics.

## Senior Answer Expectations

Senior should add:

- HLD and LLD.
- Offline/cache policy.
- Idempotency.
- Backward-compatible APIs.
- Feature flags.
- Observability.
- Security/privacy.
- Failure modes.
- Release strategy.
- Team/module boundaries.

## Architect Answer Expectations

Architect should add:

- Product tradeoffs.
- Multi-team ownership.
- Evolution strategy.
- Migration plan.
- Incident response.
- Experimentation.
- Data consistency.
- System constraints.
- Long-term maintainability.

## How To Design Netflix/YouTube Style App

Must mention:

- CDN for media.
- Playback metadata API.
- DRM/license service if needed.
- Adaptive bitrate streaming.
- Player state.
- Continue watching.
- Offline downloads.
- Search and recommendation.
- Playback analytics.
- Buffering/error handling.

## How To Design Amazon/Flipkart Style App

Must mention:

- Catalog.
- Search.
- Product details.
- Pricing.
- Inventory.
- Cart.
- Checkout.
- Payment.
- Idempotent order creation.
- Order tracking.
- Push notifications.
- Funnel analytics.

## How To Design Chat App

Must mention:

- Conversation list.
- Message list.
- WebSocket/realtime connection.
- Offline queue.
- Message delivery states.
- Push notifications.
- Deduplication.
- Read receipts.
- Media upload.
- End-to-end encryption if required.

## How To Design Food Delivery App

Must mention:

- Restaurant list.
- Menu.
- Cart.
- Order placement.
- Payment.
- Realtime order status.
- Delivery tracking.
- Maps/location.
- Push notifications.
- Cancellation/refund.

## iOS LLD Checklist

- Feature modules.
- View state.
- View model/store.
- Repository.
- API client.
- Local store.
- Mapper.
- Coordinator/router.
- Analytics client.
- Error model.
- Test strategy.

## HLD Checklist

- API gateway/BFF.
- Auth service.
- Domain services.
- Search service.
- Recommendation service.
- Payment service.
- Notification service.
- CDN.
- Analytics pipeline.
- Databases.
- Feature flag service.
- Observability.

## Communication Checklist

- REST/GraphQL choice.
- Pagination.
- Retry.
- Timeout.
- Cancellation.
- Token refresh.
- Idempotency.
- Error codes.
- Request IDs.
- Cache headers.

## Strong Interview Phrases

- "I would separate initial load failure from pagination failure."
- "Old app versions require backward-compatible APIs."
- "The UI should render from explicit state."
- "Index paths are not stable identity."
- "Checkout writes need idempotency."
- "Remote config should not block launch indefinitely."
- "Offline sync needs conflict resolution."
- "Analytics must not block critical user flows."
- "Cached data and source of truth are different."

## Red Flags

- No requirements clarification.
- No failure handling.
- No cache/offline discussion.
- No security.
- No observability.
- No API versioning.
- No state model.
- No test strategy.
- No mobile-specific constraints.

## Final Answer Template

```text
I will design this from the iOS app outward. First, I will clarify user flows and constraints. Then I will define the high-level services and communication paths. On the app side, I will split features into modules with view state, repositories, API clients, local stores, mappers, and coordinators. I will handle cache, offline behavior, token refresh, pagination, errors, analytics, performance, and release safety. Finally, I will call out tradeoffs and future extensions.
```

