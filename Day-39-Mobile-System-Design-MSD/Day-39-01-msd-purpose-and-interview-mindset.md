# Day 39: MSD Purpose And Interview Mindset

MSD means Mobile System Design. For an iOS engineer, MSD is the ability to design a real mobile product experience end to end: user flows, screens, state ownership, app architecture, APIs, offline behavior, sync, security, performance, observability, releases, and how the mobile client works with backend systems.

Traditional system design often starts from servers, databases, queues, load balancers, and distributed storage. MSD starts from the device and expands outward. The strongest iOS answers still mention backend architecture, but they stay anchored in mobile realities: app lifecycle, weak networks, old app versions, battery, memory, push notifications, permissions, app startup, accessibility, privacy, and App Store release constraints.

## What Interviewers Actually Test

In a junior interview, MSD usually tests whether you can break a feature into screens, models, APIs, and simple persistence.

In a mid-level interview, it tests whether you can design app modules, state flow, networking, caching, error handling, and testable boundaries.

In a senior interview, it tests judgment:

- Can you clarify scope before designing?
- Can you avoid overengineering while still planning for scale?
- Can you explain tradeoffs between REST, GraphQL, BFF, WebSocket, push notifications, and local sync?
- Can you handle bad network, app termination, stale data, auth expiry, duplicate requests, and old app versions?
- Can you define contracts between product, iOS, backend, QA, security, analytics, and release teams?
- Can you make decisions that reduce operational risk?

## MSD Answer Structure

Use this flow for almost every mobile system design problem:

1. Clarify product scope.
2. Define users and core flows.
3. State mobile constraints.
4. Propose high-level architecture.
5. Design iOS modules.
6. Define API contracts and data models.
7. Explain local data, cache, offline, and sync.
8. Cover concurrency and state updates.
9. Cover security, privacy, performance, observability, and release.
10. Summarize tradeoffs and future extensions.

Example opening:

```text
I will design this from the mobile client outward. First I will clarify the main user flows and scale assumptions, then I will describe the iOS architecture, API contracts, local storage, caching, sync, realtime behavior, security, performance, analytics, and rollout strategy. I will call out tradeoffs as we go.
```

That opening tells the interviewer you can drive the round.

## MSD vs iOS Architecture

iOS architecture is about the internal structure of the app: MVVM, TCA, VIPER, modules, services, repositories, navigation, state, and tests.

MSD is larger. It includes iOS architecture but also includes backend communication, product constraints, operational behavior, offline guarantees, release compatibility, observability, and reliability.

Example:

```text
Designing a chat screen with MVVM is architecture.
Designing WhatsApp for iOS is MSD.
```

For WhatsApp MSD, you discuss message models, local database, optimistic sending, delivery states, retry queues, encryption, push notifications, background sync, media upload, pagination, search, crash recovery, and how the UI stays responsive.

## Senior Mental Model

A senior iOS engineer thinks in terms of ownership and failure.

Ownership questions:

- Who owns the source of truth?
- Which module owns navigation?
- Which module owns local persistence?
- Which layer translates DTOs into domain models?
- Which service owns retries?
- Which state is view-local and which is app-global?
- Which decisions belong on the server and which belong on the client?

Failure questions:

- What happens when the API succeeds but image loading fails?
- What happens when auth expires during checkout?
- What happens when the user kills the app while an upload is in progress?
- What happens when two devices edit the same object?
- What happens when the app receives a push for data it does not have locally?
- What happens when the server deploys a new response field but older clients are still installed?

## Realistic iOS Example

Prompt:

```text
Design a food delivery order tracking screen.
```

Weak answer:

```text
I will call an API and show the order status. I will use MVVM.
```

Strong MSD answer:

```text
I will model order tracking as a state machine: placed, accepted, preparing, pickedUp, nearby, delivered, cancelled. The screen loads an initial order snapshot through REST, then subscribes to lightweight realtime updates through WebSocket or server-driven push depending on scale and battery needs. I will store the last known order locally so the user can reopen the app and see the latest known state while refresh happens. Map updates will be throttled to avoid battery drain. UI state will distinguish loading, stale, live, failed, and completed. Auth refresh and retry will live in the networking layer, not inside the view model.
```

## What To Draw

In an MSD round, draw three levels:

```text
Product flow:
Launch -> Login -> Home -> Detail -> Action -> Confirmation

System flow:
iOS App -> Mobile BFF -> Domain Services -> DB / Cache / Queue / Notifications

iOS module flow:
View -> ViewModel/Reducer -> UseCase -> Repository -> API Client / Local Store
```

For senior-level answers, add:

```text
Telemetry -> Crash/Logs/Metrics
Remote Config -> Feature Flags
Push Provider -> Notification Router
Local Store -> Offline Queue -> Sync Engine
```

## Decision Vocabulary

Use precise language:

- "Source of truth" for ownership.
- "Optimistic update" when UI updates before server confirmation.
- "Idempotency key" for safe retries.
- "Backward-compatible contract" for old app versions.
- "Stale-while-revalidate" for fast cached UI plus refresh.
- "Pagination cursor" for stable feed loading.
- "Eventual consistency" for offline sync.
- "MainActor boundary" for UI updates.
- "Repository boundary" for hiding local/network details.
- "Feature flag" for gradual rollout.

## Junior To Senior Progression

Junior:

- Can explain screens, models, APIs, and simple states.
- Knows where to call the API.
- Handles loading, success, and failure.

Mid-level:

- Separates view, state, use cases, repositories, and API clients.
- Adds caching, pagination, retries, and tests.
- Understands threading and main-thread updates.

Senior:

- Designs for old app versions, rollout, observability, security, offline, background limits, cost, product evolution, and operational failure.
- Explains why a simpler design is enough for v1 and where it can evolve.

## Interview Notes

- Start with requirements, not code.
- Speak in flows before classes.
- Mention mobile constraints early.
- Draw client architecture and backend touchpoints.
- Discuss failure cases naturally.
- Do not over-index on backend internals unless asked.
- Tie every technical choice to user experience, reliability, or team velocity.

## Points To Remember

- MSD is not only HLD. It includes LLD, data flow, device constraints, releases, and production behavior.
- A strong answer is structured, not memorized.
- Senior answers show tradeoffs, not just components.
- iOS-specific depth is your advantage. Use it.
