# Day 39: MSD Senior Interview Playbook

This file is a practical playbook for clearing mobile system design rounds as an iOS engineer. Use it when you need to convert a vague prompt into a structured senior-level answer.

## The 35-Minute MSD Round

### Minute 0-3: Clarify

Say:

```text
Before designing, I will clarify scope: users, core flows, offline needs, realtime needs, sensitive data, scale, and supported iOS versions.
```

Ask 4-6 questions, then make assumptions.

### Minute 3-7: User Flows

List flows:

```text
Launch -> Auth/session -> Home
Search -> Results -> Detail
Action -> Local state -> Server confirmation
Push/deep link -> Auth gate -> Target screen
Offline action -> Queue -> Sync -> Reconcile
```

### Minute 7-12: HLD

Draw:

```text
iOS App -> Mobile BFF/API Gateway -> Domain Services
iOS App -> CDN
Push Provider -> iOS App
iOS App -> Analytics/Crash/Logs
Remote Config -> iOS App
```

Explain:

- Why BFF or direct API.
- Which data comes from CDN.
- Which events use push/WebSocket/polling.
- What is cached locally.

### Minute 12-20: iOS LLD

Draw:

```text
View -> ViewModel/Reducer -> UseCase -> Repository -> API Client / Local Store
```

Mention:

- DTO/domain/view state separation.
- Dependency injection.
- MainActor UI updates.
- Actor or serial queue for shared mutable services.
- Feature module boundaries.
- Test strategy.

### Minute 20-27: Data, Offline, Realtime

Cover:

- Cache type.
- Pagination.
- Offline queue.
- Idempotency.
- Conflict resolution.
- Realtime channel.
- Background limitations.

### Minute 27-32: Security, Performance, Observability

Cover:

- Keychain.
- Auth refresh.
- Privacy-safe logging.
- Startup and screen latency.
- Memory/battery.
- Crash and metric tracking.
- Feature flags and phased rollout.

### Minute 32-35: Tradeoffs And Summary

End with:

```text
The main tradeoff is between freshness, correctness, and mobile reliability. For this design I would keep the server authoritative for critical state, cache read-heavy data for speed, use idempotent retries for safe writes, and operate the rollout with feature flags and observability.
```

## Generic Answer Template

```text
I will design [app/feature] for iOS.

Scope:
- [core flow 1]
- [core flow 2]
- [core flow 3]

Assumptions:
- [online/offline]
- [realtime/polling]
- [sensitive/non-sensitive data]
- [scale]

HLD:
- iOS app talks to [BFF/API gateway].
- Media/static assets use [CDN].
- Auth handled by [auth service].
- Realtime handled by [push/WebSocket/polling].
- Observability through [crash/logs/metrics/analytics].

iOS LLD:
- View -> ViewModel/Reducer -> UseCase -> Repository -> API/Local Store.
- DTOs are mapped to domain models.
- UI state is modeled explicitly.
- Dependencies are injected for testing.

Data:
- [cache strategy]
- [pagination]
- [offline rules]
- [sync/conflict strategy]

Security/performance:
- Keychain for secrets.
- Main thread protected.
- Images/media optimized.
- Feature flags and phased rollout.

Tradeoffs:
- [choice A] gives [benefit] but costs [risk].
- I choose [decision] because [reason].
```

## Senior Decision Matrix

| Problem | Simple Choice | Senior Choice |
| --- | --- | --- |
| Home feed | Fetch every time | Cached snapshot + refresh + cursor pagination |
| Chat send | Call API and append | Local optimistic message + client ID + retry + reconcile |
| Payments | Retry on failure | Idempotency key + server-authoritative state + no unsafe retry |
| Realtime | Always WebSocket | WebSocket only for active live flows, push/poll fallback |
| Images | Load raw URL | Downsample, cache, cancel, prefetch carefully |
| Auth | Add token header | Keychain + refresh coordination + logout recovery |
| Release | Ship feature live | Dark launch + feature flag + phased rollout |
| Errors | Show alert | Typed error states + retry + stale content fallback |

## Red Flags In Answers

Avoid saying:

- "The app will always stay connected."
- "Push will update everything reliably."
- "I will store all data locally."
- "I will use singleton for all services."
- "The client will validate payments."
- "I will retry all failed requests."
- "We do not need analytics for design."
- "Offline is easy, just cache it."

Better:

```text
Offline is a product and correctness decision. I would allow offline drafts and low-risk actions, but not final payment or banking transfer approval.
```

## Example Mini Answer: Design Search

```text
I will design search with debounced input, cancellable requests, recent searches cached locally, server-side ranking, cursor pagination, and empty/error states. The search view model owns query and UI state. Repository hides API and cache. Requests are cancelled when the query changes. The API returns results plus next cursor and tracking metadata. I would log search latency and zero-result rate, but not raw sensitive queries if the domain is private.
```

## Example Mini Answer: Design Image Loader

```text
I would build an image loading service with memory cache, disk cache, request coalescing, cancellation, downsampling, and priority support. Cells request images through stable URLs. When a cell is reused, the task is cancelled. The service avoids decoding huge images on the main thread. Cache policy respects HTTP headers where possible and storage limits are enforced.
```

## Example Mini Answer: Design Offline Notes

```text
Notes can be created and edited offline because the product can tolerate eventual consistency. I would store notes locally with updatedAt, version, dirty flag, and client mutation ID. Sync uploads pending mutations when network returns. If the same note changed on another device, I would either merge field-level changes or show a conflict screen depending on product expectations.
```

## Interview Questions To Practice

- Design Instagram feed for iOS.
- Design WhatsApp chat for iOS.
- Design YouTube video playback for iOS.
- Design Amazon checkout for iOS.
- Design Uber live tracking for iOS.
- Design offline notes sync.
- Design image loading and caching.
- Design app-wide analytics.
- Design deep link routing.
- Design a secure banking transfer flow.
- Design a notification system for a marketplace app.
- Design search with suggestions and pagination.

## Final Senior Checklist

Before ending your answer, confirm you covered:

- Requirements.
- HLD.
- LLD.
- API contracts.
- Local storage.
- Cache/offline/sync.
- Realtime/push/background.
- Security/privacy.
- Performance/memory/battery.
- Observability.
- Testing.
- Release strategy.
- Tradeoffs.

## Points To Remember

- MSD is a conversation. Drive it calmly.
- Draw the app first, then backend touchpoints.
- Mobile constraints are not edge cases; they are the system.
- Senior answers explain why, not only what.
- A good summary at the end can rescue a messy middle.
