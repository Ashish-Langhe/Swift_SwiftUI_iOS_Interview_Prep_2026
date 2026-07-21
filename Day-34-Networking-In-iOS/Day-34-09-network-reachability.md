# Day 34: Network Reachability

## What Reachability Means

Reachability tells you about current network path status.

Modern iOS uses Network framework path monitoring.

Important: reachability is advisory. It does not guarantee your next request will succeed.

## NWPathMonitor

```swift
import Network

@Observable
@MainActor
final class NetworkMonitor {
    var isConnected = false
    var isExpensive = false
    var isConstrained = false

    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")

    func start() {
        monitor.pathUpdateHandler = { [weak self] path in
            Task { @MainActor in
                self?.isConnected = path.status == .satisfied
                self?.isExpensive = path.isExpensive
                self?.isConstrained = path.isConstrained
            }
        }

        monitor.start(queue: queue)
    }

    func stop() {
        monitor.cancel()
    }
}
```

`isExpensive` often means cellular or hotspot. `isConstrained` can indicate Low Data Mode.

## Reachability Is Not Permission To Request

Bad:

```swift
guard networkMonitor.isConnected else {
    showOfflineMessage()
    return
}

try await api.load()
```

The network can change after the check.

Better:

- Let requests fail.
- Use reachability for UI hints.
- Use reachability for offline mode decisions.
- Use URLSession waits/retry policies.

## Offline UI

```swift
if !networkMonitor.isConnected {
    OfflineBanner()
}
```

Good offline behavior:

- Show cached data.
- Disable actions that require online.
- Queue safe writes if supported.
- Offer retry.
- Avoid alert spam.

## Low Data Mode

```swift
if networkMonitor.isConstrained {
    imageQuality = .thumbnail
}
```

Use Low Data Mode signals to reduce:

- Video auto-play
- Large image downloads
- Background sync
- Preloading

## waitsForConnectivity vs NWPathMonitor

`waitsForConnectivity`:

- URLSession behavior for connection establishment.
- Lets task wait for sufficient connectivity.

`NWPathMonitor`:

- App-level network path observation.
- Useful for UI and policy decisions.

They solve different problems.

## Retry After Path Change

```swift
func pathBecameAvailable() {
    Task {
        await syncEngine.resumePendingWork()
    }
}
```

Be careful:

- Debounce path changes.
- Do not trigger huge sync immediately.
- Respect battery/data constraints.

## Senior iOS Engineer Perspective

Ask:

- Is reachability used for UI hint or request blocking?
- Is offline cache available?
- Is Low Data Mode respected?
- Are expensive networks handled?
- Are path changes debounced?
- Is retry policy still needed?
- Is URLSession configured appropriately?

## Common Mistakes

- Treating reachability as truth.
- Blocking every request when offline flag says no.
- No request-level error handling.
- Ignoring Low Data Mode.
- Alert spam on path changes.
- Not cancelling monitor.
- Doing UI updates off main actor.

## Interview Notes

Junior:

Reachability checks whether network appears available.

Mid-level:

Use `NWPathMonitor` for network path changes and UI hints, but still handle request failures.

Senior:

I treat reachability as advisory. I combine path monitoring, request-level errors, caching, retry policy, Low Data Mode, and URLSession configuration.

## Practice

1. Build a `NetworkMonitor` with `NWPathMonitor`.
2. Show an offline banner.
3. Respect Low Data Mode for image quality.
4. Explain why reachability cannot guarantee success.
