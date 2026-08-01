# Day 39: Performance, Memory, Battery, And UX Reliability

In MSD, performance is not only frames per second. It includes launch time, screen load time, scrolling smoothness, memory footprint, battery usage, network efficiency, responsiveness, and user-perceived reliability.

## Performance Targets

Common targets:

- Fast cold launch.
- Home screen usable quickly.
- Smooth scrolling.
- No main-thread blocking.
- Bounded memory growth.
- Minimal duplicate API calls.
- Responsive interactions.
- Graceful degraded states.

Senior answer:

```text
I would define success metrics before optimizing: cold launch time, p95 screen load latency, scroll hitch rate, memory footprint, crash-free sessions, and API failure rate by app version.
```

## Launch Performance

Avoid doing too much at startup.

Do at launch:

- Initialize lightweight app container.
- Restore session state.
- Load minimal remote config if cached.
- Show cached shell quickly.

Defer:

- Heavy database migrations if possible.
- Large analytics setup.
- Non-critical SDK initialization.
- Full feed fetch before first render.

Example:

```text
For an e-commerce app, I would show the shell and cached home sections quickly, then refresh personalized content asynchronously.
```

## Main-Thread Work

Main thread should handle UI work. Heavy work should move away from it.

Risky tasks:

- JSON decoding of very large payloads.
- Image decoding/resizing.
- Database queries on main thread.
- Synchronous file I/O.
- Complex layout recalculation.

SwiftUI example:

```swift
func load() {
    Task {
        do {
            let products = try await repository.products()
            await MainActor.run {
                self.state = .loaded(products)
            }
        } catch {
            await MainActor.run {
                self.state = .failed
            }
        }
    }
}
```

## Scrolling Performance

For feeds and lists:

- Paginate.
- Prefetch carefully.
- Use image resizing.
- Avoid unstable identifiers.
- Avoid expensive work in cell/body rendering.
- Cache layout-relevant data.
- Track impressions without blocking UI.

Bad:

```swift
var body: some View {
    ForEach(posts) { post in
        Text(expensiveFormatting(post.date))
    }
}
```

Better:

```swift
struct FeedRowViewModel: Identifiable, Equatable {
    let id: String
    let title: String
    let formattedDate: String
    let imageURL: URL?
}
```

## Memory Management

Common mobile memory risks:

- Retain cycles in closures.
- Huge image caches.
- Video players retained after leaving screen.
- Map views holding resources.
- Large decoded JSON arrays.
- Unbounded local caches.

Design choices:

- Use `weak self` where closures outlive owner.
- Release player/map resources when screen disappears.
- Use cache eviction.
- Downsample images.
- Stream large data instead of loading all at once.

## Battery

Battery is often forgotten in MSD answers.

Battery-heavy operations:

- GPS.
- Camera.
- Bluetooth.
- Constant polling.
- WebSocket reconnect loops.
- Background uploads.
- Video playback.

Senior example:

```text
For Uber tracking, I would use high-frequency location updates only during active trip states. Outside active tracking, I would reduce accuracy or stop updates to protect battery.
```

## UX Reliability

Reliability means users understand what is happening.

Good states:

- Loading.
- Loaded fresh.
- Loaded stale.
- Empty.
- Failed retryable.
- Failed final.
- Offline.
- Syncing.
- Pending local change.

Example:

```swift
enum OrderActionState: Equatable {
    case idle
    case submitting
    case pendingOffline
    case confirmed
    case failed(message: String)
}
```

## Performance Tradeoffs

### Cache More

Pros:

- Faster app.
- Better offline experience.
- Less network.

Cons:

- Stale data.
- Storage usage.
- Privacy risk.
- Cache invalidation complexity.

### Fetch Fresh Every Time

Pros:

- Simpler correctness.
- Less local storage complexity.

Cons:

- Slower.
- Poor offline UX.
- More backend load.

Senior decision:

```text
I would cache read-heavy, non-sensitive data like feed items and product details, but fetch sensitive or correctness-critical data like bank balances from the server and clearly mark refresh time.
```

## Interview Notes

- Always define performance metrics.
- Mention Instruments, Time Profiler, Allocations, Memory Graph, and network profiling.
- Battery belongs in mobile design.
- Smooth UI requires data shaping, caching, and cancellation.
- Senior answers connect performance decisions to user experience.
