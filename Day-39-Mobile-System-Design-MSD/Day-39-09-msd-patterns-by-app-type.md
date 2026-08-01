# Day 39: MSD Patterns By App Type

Most MSD prompts are variations of a few app families. If you understand the patterns, you can answer new problems calmly.

## Feed Apps

Examples:

- Instagram feed.
- LinkedIn feed.
- News app.
- Product recommendations.

Core design:

- Server-ranked feed.
- Cursor pagination.
- Local feed cache.
- Image/video CDN.
- Optimistic likes/saves.
- Impression analytics.
- Pull-to-refresh.
- Prefetch next page.

Tradeoffs:

- Server ranking gives consistent product control.
- Client ranking is limited and can become stale.
- Aggressive prefetch improves speed but costs network and battery.

Senior answer:

```text
I would store feed ordering separately from post objects so updates to one post do not require rewriting the entire feed.
```

## Chat Apps

Examples:

- WhatsApp.
- Slack.
- In-app support chat.

Core design:

- Local message database.
- Optimistic send with client message ID.
- Delivery states.
- WebSocket for active chat.
- Push for background messages.
- Media upload pipeline.
- Pagination from latest messages backward.
- Encryption if required.

Message states:

```swift
enum MessageStatus: Equatable {
    case draft
    case sending
    case sent
    case delivered
    case read
    case failedRetryable
}
```

Senior tradeoff:

```text
For chat, local database is not optional at scale. It gives fast open, offline drafts, search, and recovery after app termination.
```

## E-Commerce Apps

Examples:

- Amazon.
- Flipkart.
- Grocery delivery.

Core design:

- Home composition through BFF.
- Product details with cache.
- Search and filters.
- Cart server source of truth.
- Checkout state machine.
- Payment flow.
- Inventory revalidation.
- Order tracking.

Senior detail:

```text
I would allow optimistic cart UI for responsiveness, but checkout must revalidate price, inventory, address, offers, and payment eligibility on the server.
```

## Video Streaming Apps

Examples:

- YouTube.
- Netflix.
- Course streaming app.

Core design:

- Feed/search metadata from API.
- Playback from CDN.
- Adaptive bitrate streaming.
- Watch progress sync.
- Offline downloads where licensed.
- Recommendation surfaces.
- Player lifecycle management.
- Error fallback for network changes.

Senior tradeoff:

```text
The app should not proxy video bytes through the API service. Metadata comes from API/BFF; media comes from CDN.
```

## Ride Tracking Apps

Examples:

- Uber.
- Ola.
- Delivery tracking.

Core design:

- Initial trip snapshot.
- Realtime driver/location updates.
- Map rendering.
- Location permissions.
- Push for status changes.
- Polling fallback.
- Battery-aware update frequency.
- Trip state machine.

Senior detail:

```text
Location frequency should depend on trip state. Searching for driver, active pickup, active trip, and completed trip do not need the same update policy.
```

## Banking Apps

Core design:

- Secure login.
- Device binding if required.
- Keychain token storage.
- Biometric unlock.
- Server-authoritative balances.
- Transaction history pagination.
- High-risk action confirmation.
- Audit logs.
- Privacy-safe analytics.
- Session timeout.

Senior tradeoff:

```text
Banking should optimize for correctness and trust before speed. Cache cautiously and display last updated time when showing sensitive financial data.
```

## Marketplace Apps

Examples:

- Airbnb.
- Swiggy.
- Food delivery.
- Ride-hailing.

Core design:

- Search/discovery.
- Availability.
- Cart/booking.
- Payments.
- Order/trip state.
- Notifications.
- Location.
- Support/refund flows.

Senior detail:

```text
Availability and price must be revalidated before final confirmation because inventory can change between browse and checkout.
```

## Social/Profile Apps

Core design:

- Profile cache.
- Follow/unfollow.
- Posts grid.
- Privacy rules.
- Blocking/reporting.
- Media upload.
- Push notifications.

Senior note:

```text
Privacy and blocking rules must be enforced on the server. Client hiding improves UX but cannot be the security boundary.
```

## Interview Notes

- Identify the app family quickly.
- Reuse known patterns, then customize.
- Mention one state machine for complex flows.
- Mention one cache strategy.
- Mention one failure mode and recovery.
- Mention one release/observability plan.
