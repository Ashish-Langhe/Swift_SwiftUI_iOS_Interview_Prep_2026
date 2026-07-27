# Day 38: Requirements, Scale, And API Contracts

Good system design starts with requirements. A weak answer jumps directly into classes or services. A strong iOS system design answer first clarifies product behavior, scale, data freshness, offline expectations, and API contracts.

## Functional Requirements

Functional requirements describe what users can do.

Example for a YouTube-like app:

- Browse home feed.
- Search videos.
- Play video.
- Like, comment, subscribe.
- Continue watching.
- Download video for offline use.
- Receive notifications.

Example for an Amazon/Flipkart-like app:

- Browse catalog.
- Search products.
- View product details.
- Add to cart.
- Apply offers.
- Checkout.
- Track orders.
- Return/refund.

## Non-Functional Requirements

Non-functional requirements describe quality.

For iOS apps:

- Fast launch.
- Smooth scrolling.
- Low crash rate.
- Low memory usage.
- Offline tolerance.
- Secure storage.
- Accessibility.
- Privacy compliance.
- Battery efficiency.
- API backward compatibility.
- Low network usage.

## Scale Questions

Ask:

- How many daily active users?
- How many requests per user per day?
- How large is each payload?
- How often does content change?
- What regions are supported?
- Is content personalized?
- Is offline mode required?
- Are there real-time updates?
- What is acceptable latency?

## iOS-Specific Constraints

iOS apps are constrained by:

- Network variability.
- Background execution limits.
- Battery.
- Memory.
- App size.
- App Review.
- Privacy requirements.
- Device differences.
- OS version compatibility.
- Release cycles.

Unlike backend services, once an app version is released, old clients may exist for months or years.

## API Contract Design

API contracts should be:

- Versioned.
- Backward compatible.
- Stable.
- Paginated.
- Typed.
- Error-aware.
- Cache-aware.

Example product list response:

```json
{
  "items": [
    {
      "id": "p_123",
      "title": "Wireless Keyboard",
      "price": {
        "amount": "129.00",
        "currency": "USD"
      },
      "imageURL": "https://cdn.example.com/p_123.png",
      "availability": "in_stock"
    }
  ],
  "nextCursor": "cursor_abc",
  "serverTime": "2026-07-27T10:00:00Z"
}
```

## API Versioning

Common approaches:

- URL versioning: `/v1/products`
- Header versioning: `API-Version: 2026-07`
- Capability flags: server returns features supported by client.

Senior note: mobile API changes must support old app versions. Do not remove fields suddenly. Add fields safely.

## Error Contract

Bad error:

```json
{
  "error": "Something wrong"
}
```

Better error:

```json
{
  "code": "PAYMENT_CARD_DECLINED",
  "message": "The card was declined.",
  "retryable": false,
  "requestID": "req_123"
}
```

iOS mapping:

```swift
struct APIErrorResponse: Decodable {
    let code: String
    let message: String
    let retryable: Bool
    let requestID: String?
}
```

## Pagination Contract

Prefer cursor pagination for feeds:

```json
{
  "items": [],
  "nextCursor": "eyJwYWdlIjoyfQ=="
}
```

Why cursor pagination:

- Handles inserted/deleted content better than page number.
- Works well for personalized feeds.
- Avoids duplicate/missing items in fast-changing lists.

Page number can be okay for stable catalog pages, but cursor is often safer.

## Idempotency

For actions like payment or order creation, use idempotency keys.

```http
POST /orders
Idempotency-Key: 7C2E-1234
```

Why:

- User taps twice.
- Network retries.
- App crashes after request.
- Server receives duplicate requests.

The server should return the same result for the same idempotency key.

## Data Freshness

Different data has different freshness needs:

| Data | Freshness Need | Cache Strategy |
|---|---|---|
| Product price | High | short cache, server validation |
| Product images | Medium | CDN + disk cache |
| User profile | Medium | cache then refresh |
| Auth token | High security | Keychain, expiration |
| Video metadata | Medium | cache with TTL |
| Cart | High consistency | server source of truth |

## iOS DTO, Domain, View Model Contract

Do not use API DTO directly in UI.

```swift
struct ProductDTO: Decodable {
    let id: String
    let title: String?
    let imageURL: URL?
}

struct Product {
    let id: String
    let title: String
    let imageURL: URL?
}

struct ProductRow: Hashable, Identifiable {
    let id: String
    let titleText: String
    let imageURL: URL?
}
```

This gives the app control over defaults and display behavior.

## Interview Answer Example

```text
I would first clarify whether the feed must work offline, whether it is personalized, and whether order matters. Then I would design a cursor-paginated API with stable item IDs, typed errors, request IDs, and cache headers. On iOS, I would decode DTOs, map to domain models, then map to row models for UI. I would keep API versioning backward compatible because old app versions remain in the wild.
```

## Common Mistakes

- Not asking requirements.
- Ignoring old app versions.
- Designing APIs with no pagination.
- No typed error model.
- No request ID for debugging.
- No idempotency for write APIs.
- UI depends directly on backend response shape.
- Same cache policy for all data.

