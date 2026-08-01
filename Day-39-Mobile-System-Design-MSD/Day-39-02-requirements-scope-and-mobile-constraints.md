# Day 39: Requirements, Scope, And Mobile Constraints

The first five minutes of an MSD interview decide the quality of the rest of the answer. If you start designing immediately, you may solve the wrong problem. A senior iOS engineer first narrows scope, names assumptions, and identifies mobile constraints that affect the architecture.

## Functional Requirements

Functional requirements describe what the app must do.

For a YouTube-like app:

- Show home feed.
- Search videos.
- Play video.
- Show comments.
- Save watch progress.
- Download videos for offline viewing.
- Send notifications.

For a banking app:

- Login securely.
- Show accounts and balances.
- View transactions.
- Transfer money.
- Approve high-risk actions with biometrics or OTP.
- Show receipts.
- Handle session timeout.

Good interview phrasing:

```text
I will scope the first version to the consumer iOS app. I will cover the happy path, offline or poor-network behavior, security, performance, and release strategy. I will exclude admin tooling unless you want me to include it.
```

## Non-Functional Requirements

Non-functional requirements decide design quality.

Ask:

- What latency is acceptable?
- Does the app need offline mode?
- Is realtime required?
- How many users or sessions are expected?
- Are payments or sensitive data involved?
- Is the app global?
- Which iOS versions are supported?
- Are accessibility and localization required?
- Is the backend already available?

Example:

```text
For a chat app, latency and reliability matter more than perfect global ordering. For a banking app, correctness, auditability, and security matter more than aggressive UI speed.
```

## Mobile Constraints That Change Design

Mobile apps are different from web clients and backend systems.

### Network Is Unstable

Users move between Wi-Fi, cellular, tunnels, elevators, and offline areas.

Design impact:

- Use clear loading and stale states.
- Retry only safe operations.
- Add request timeouts.
- Cache read-heavy data.
- Queue offline writes only when business rules allow it.

### App Can Be Killed

iOS can terminate the app in the background.

Design impact:

- Persist important in-flight state.
- Do not rely on long-running background work.
- Make uploads/downloads resumable where possible.
- Reconcile state on next launch.

### Battery Matters

Realtime does not mean constant polling.

Design impact:

- Prefer push for rare events.
- Use WebSocket only for active realtime sessions.
- Throttle location updates.
- Batch analytics.
- Avoid unnecessary background refresh.

### Memory Is Limited

Large feeds, images, videos, and maps can create memory pressure.

Design impact:

- Use image downsampling.
- Paginate.
- Release invisible resources.
- Avoid keeping full decoded images in memory.
- Watch retain cycles in long-lived services.

### Old App Versions Stay Alive

Backend can deploy daily. Mobile apps may remain old for months.

Design impact:

- Maintain backward-compatible APIs.
- Use optional fields safely.
- Version critical contracts.
- Use feature flags and remote config.
- Avoid server changes that break old clients.

### Permissions Are User-Controlled

Location, camera, photos, notifications, contacts, and biometrics can be denied.

Design impact:

- Model denied, restricted, notDetermined, and authorized states.
- Provide graceful fallback flows.
- Avoid asking permission before user intent is clear.

## Scope Template

Use this during interviews:

```text
I will design:
- Primary user app flow
- Core screen architecture
- API and data model
- Local persistence and cache
- Offline/retry behavior
- Security and privacy
- Performance and observability
- Rollout and testing

I will not deeply design:
- Admin dashboards
- Full backend database schema
- ML ranking internals
- Infrastructure scaling beyond what affects mobile contracts
```

## Example: Clarifying Instagram Feed

Ask:

- Is this home feed only or also profile/explore?
- Are posts image-only or video too?
- Do we need likes, comments, saves, stories?
- Is offline feed browsing required?
- Is ranking server-driven?
- Should new posts appear live?

Good scoped answer:

```text
I will design the home feed with image/video posts, pagination, like/save actions, comments count, pull-to-refresh, cached feed on launch, and server-driven ranking. I will keep story creation and explore out of scope.
```

## Example: Clarifying Uber Ride Tracking

Ask:

- Rider app only or driver app too?
- Do we include booking or only active trip tracking?
- How realtime should driver location be?
- What happens when location permission is denied?
- Do we need fare updates and route changes?

Senior framing:

```text
For active ride tracking, I will combine an initial REST snapshot with realtime location updates. I will throttle UI map updates and fall back to polling if the realtime channel fails.
```

## Common Mistakes

- Designing backend tables before user flows.
- Ignoring offline or poor-network behavior.
- Saying "WebSocket" for every realtime problem.
- Storing sensitive data casually.
- Forgetting old app versions.
- Treating push notifications as guaranteed delivery.
- Putting retry logic directly in view models.
- Missing accessibility, localization, and observability.

## Interview Notes

- Requirements are not a formality. They shape everything.
- If the interviewer gives vague scope, define assumptions out loud.
- Say what you are excluding.
- Always include mobile constraints.
- Senior engineers are judged by boundaries and tradeoffs.
