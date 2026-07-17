# Day 32: Memory Graph And Leak Debugging

## What Memory Graph Is

Xcode's Memory Graph Debugger shows objects and references in your running app.

Use it to investigate:

- Retain cycles
- Leaked view controllers
- Leaked SwiftUI models
- Unexpected object growth
- Strong reference chains
- Closure capture issues
- Delegate cycles

It answers:

```text
Why is this object still alive?
```

## How To Capture

While debugging in Xcode:

```text
Debug area -> Debug Memory Graph button
```

The graph shows:

- Object nodes
- Heap allocations
- Memory regions
- Reference arrows
- Retain paths

Enable Malloc Stack in scheme diagnostics to see allocation stack traces for graph nodes.

## Leak vs High Memory

Leak:

Object should be gone, but references keep it alive.

High memory:

Object may be alive legitimately, but memory footprint is too large.

Example:

```text
Leaked ProfileViewModel after leaving screen -> leak
Image cache holds 300 MB intentionally -> high memory/cache policy issue
```

## Classic Closure Retain Cycle

Bad:

```swift
final class ProfileViewModel {
    var onUpdate: (() -> Void)?

    func start() {
        service.observeProfile { profile in
            self.profile = profile
        }
    }
}
```

If service retains the closure and closure retains `self`, the ViewModel leaks.

Better:

```swift
service.observeProfile { [weak self] profile in
    self?.profile = profile
}
```

## UIKit Delegate Cycle

Bad:

```swift
protocol ImagePickerDelegate: AnyObject {}

final class ImagePickerViewController: UIViewController {
    var delegate: ImagePickerDelegate?
}
```

If parent retains child and child strongly retains parent delegate, cycle.

Better:

```swift
weak var delegate: ImagePickerDelegate?
```

## Coordinator Cycle

Common issue:

```text
Parent coordinator -> child coordinator
Child coordinator -> view controller
View controller -> view model
View model -> coordinator
```

Fix:

- ViewModel should hold coordinator weakly if class-bound.
- Child coordinator should be removed on finish.
- Use closures with weak captures.

## SwiftUI Retain Cycle Example

```swift
@Observable
final class SearchModel {
    var results: [Product] = []
    private var task: Task<Void, Never>?

    func startPolling() {
        task = Task {
            while !Task.isCancelled {
                results = await service.products()
            }
        }
    }
}
```

The task can retain the model.

Better:

```swift
func stop() {
    task?.cancel()
    task = nil
}
```

Or tie work to SwiftUI `.task`, which cancels with view lifetime.

## Debugging Steps

```text
1. Navigate to screen
2. Perform actions
3. Navigate away
4. Capture memory graph
5. Search for expected-deallocated type
6. Inspect retain path
7. Fix strongest unexpected reference
8. Repeat until object deallocates
```

Add deinit logging temporarily:

```swift
deinit {
    print("ProfileViewModel deinit")
}
```

Use sparingly and remove after diagnosis.

## Memory Graph With Allocation Stacks

Enable:

```text
Scheme -> Run -> Diagnostics -> Malloc Stack
```

Then the memory graph can show where an object was allocated, not only why it is retained.

## Common Leak Sources

- Strong delegate
- Closure captures
- Timer retaining target
- Notification observer not removed in older APIs
- Combine `sink` retain cycles
- Task retaining owner
- Coordinator child not removed
- CADisplayLink cycles
- Caches with no eviction
- UIViewRepresentable coordinator cycles

## Senior Artifact

```text
Artifact: Leak Investigation
Leaked type:
Expected deallocation moment:
Repro steps:
Memory graph capture:
Retain path:
Root owner:
Fix:
Verification:
Remaining similar risks:
```

## Interview Notes

Junior:

Memory Graph helps find objects still alive in memory.

Mid-level:

Use it to inspect retain paths and identify cycles from closures, delegates, timers, or coordinators.

Senior:

I distinguish leaks from legitimate high memory, capture after expected deallocation, inspect retain paths with allocation stacks, fix ownership, and verify with repeated graph captures.

## Practice

1. Create and fix a closure retain cycle.
2. Debug a leaked view controller.
3. Find a coordinator child leak.
4. Explain leak vs cache growth.
