# Day 38: Offline, Cache, And Sync Design

Offline behavior is one of the biggest differences between mobile and web system design. iOS apps must handle poor network, airplane mode, app backgrounding, partial data, stale cache, and multi-device conflicts.

## Cache vs Source Of Truth

Cache:

- Improves speed.
- May be stale.
- Can be deleted.
- Should have invalidation policy.

Source of truth:

- Authoritative.
- Usually backend for shared data.
- Sometimes local for drafts or offline-first apps.

## Cache Types

### Memory Cache

Use for:

- Images.
- Recently decoded data.
- Short-lived objects.

### Disk Cache

Use for:

- Images.
- API responses.
- Offline content.
- Downloaded media.

### Database

Use for:

- Structured offline data.
- Sync metadata.
- Searchable local data.
- Relationships.

### Keychain

Use for:

- Tokens.
- Credentials.
- Sensitive secrets.

## Cache Policy Examples

| Data | Policy |
|---|---|
| Thumbnails | CDN + disk cache |
| Product list | cache then network |
| Product price | short TTL, refresh before checkout |
| Auth token | Keychain |
| User draft | local source until submitted |
| Video download | offline store with expiry |

## Cache-Then-Network

Flow:

1. Load cached data.
2. Render immediately.
3. Fetch remote data.
4. Update cache.
5. Render fresh data.
6. If network fails, keep cached data with warning.

```swift
enum LoadEvent<Value> {
    case cached(Value)
    case fresh(Value)
}
```

## Offline Queue

For offline actions:

- Save action locally.
- Mark pending.
- Retry when network returns.
- Resolve conflict.
- Show sync status.

Example:

```swift
struct PendingAction: Codable, Identifiable {
    let id: String
    let type: ActionType
    let payload: Data
    let createdAt: Date
    var attemptCount: Int
}
```

## Sync States

```swift
enum SyncState: Equatable {
    case synced
    case pending
    case syncing
    case failed(message: String)
    case conflicted
}
```

## Conflict Resolution

Strategies:

- Last write wins.
- Server wins.
- Client wins.
- Manual resolution.
- Field-level merge.

Example:

```text
Notes app:
- Title changed on device A.
- Body changed on device B.
- Field-level merge may preserve both.

Payment app:
- Server wins. Never trust local conflict resolution for money.
```

## Offline-First Design

Offline-first means:

- Local database is primary for UI.
- Writes go local first.
- Sync pushes changes later.
- UI shows sync state.

Good for:

- Notes.
- Tasks.
- Drafts.
- Field apps.

Risky for:

- Payments.
- Inventory.
- Trading.
- Checkout.

## Cache Invalidation

Invalidate by:

- TTL.
- ETag.
- App version.
- User logout.
- Manual refresh.
- Push notification.
- Server version.

## Logout Cleanup

On logout:

- Clear auth tokens.
- Clear sensitive cache.
- Cancel in-flight requests.
- Reset user-specific database.
- Clear pending uploads if they belong to user.
- Reset analytics user identity.

## Background Sync

iOS background execution is limited. Design as best-effort.

Use cases:

- Upload pending drafts.
- Refresh small data.
- Finish downloads.

Senior note: never design critical correctness assuming unlimited background execution.

## Interview Answer Example

```text
I would decide which data is cache and which is source of truth. For product browsing, I can use cache-then-network. For cart and checkout, server remains source of truth. For notes, local database can be source first with offline queue and conflict resolution. Every cached item needs invalidation, logout cleanup, and sync state.
```

## Common Mistakes

- Calling every local store "offline support."
- No invalidation policy.
- No logout cleanup.
- Assuming background sync always runs.
- Same conflict strategy for all domains.
- Hiding sync failures from user.

