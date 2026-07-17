# Day 16: Debugging Memory Issues

## Types Of Memory Issues

Common iOS memory issues:

- Memory leaks
- Retain cycles
- Excessive memory growth
- Large image/data retention
- Caches that never evict
- Tasks/subscriptions that outlive screens
- Accessing deallocated objects through unsafe references

## Basic Vs Advanced Memory Problems

Basic ARC issue:

```text
Object does not deallocate because something still strongly references it.
```

Advanced memory issue:

```text
Object deallocates, but memory still grows because images, caches, tasks, or buffers are retained elsewhere.
```

Do not assume every memory issue is one retain cycle.

## First Debugging Artifact: Deinit Logs

Add a `deinit` log to confirm release.

```swift
final class ProfileViewModel {
    deinit {
        print("ProfileViewModel deallocated")
    }
}
```

If `deinit` does not print after dismissal, investigate ownership.

## Deinit Log Template Artifact

```swift
deinit {
    print("DEINIT:", String(describing: Self.self))
}
```

For view controllers:

```swift
deinit {
    print("DEINIT ProfileViewController")
}
```

For view models:

```swift
deinit {
    print("DEINIT ProfileViewModel")
}
```

## Xcode Memory Graph

Use Xcode's Memory Graph Debugger to inspect object references.

Checklist:

1. Run the app.
2. Navigate to the screen.
3. Dismiss the screen.
4. Open Memory Graph.
5. Search for the view controller/view model.
6. Inspect strong reference paths.

What to look for:

- Strong reference chain back to the object
- Closure retaining owner
- Timer or subscription retaining owner
- Unexpected singleton/cache references
- Objects that remain after screen dismissal

## Instruments

Use Instruments for deeper analysis:

- Leaks instrument
- Allocations instrument
- Time Profiler when memory growth causes performance issues

Suggested workflow:

```text
Allocations:
1. Record baseline.
2. Navigate repeatedly through flow.
3. Observe persistent growth.
4. Mark generation.
5. Inspect objects that survive.

Leaks:
1. Run leak detection.
2. Reproduce lifecycle.
3. Inspect leaked object graph.
```

## Debugging Retain Cycles

Look for:

- Strong delegate
- Closure property capturing `self`
- Parent-child strong cycle
- Timer retaining target
- Notification observer not removed
- Combine subscription retaining owner
- Task retaining owner

## Debugging Large Memory Growth

Look for:

- Images not downsampled
- Large arrays retained by cache
- `Data` stored longer than needed
- Repeated decoding without release
- Autorelease-heavy Objective-C APIs

Image-specific checklist:

```text
[ ] Are large images downsampled before display?
[ ] Are original full-size images retained unnecessarily?
[ ] Is image cache bounded?
[ ] Are images released when screen disappears?
[ ] Are repeated thumbnails regenerated instead of cached appropriately?
```

## Modern Swift 6.x Notes

Swift 6.2 introduced opt-in strict memory safety diagnostics for unsafe constructs. This helps identify use of unsafe APIs such as unsafe pointers or `unowned(unsafe)`. This is different from ARC leak debugging, but both belong to memory safety.

Swift 6.2 also introduced `Span`, which offers safe access to contiguous memory and helps avoid pointer-style memory bugs in systems-level code.

## Memory Debugging Checklist Artifact

Use this checklist during app review:

```text
Memory Review Checklist

[ ] Does the screen's view model deinit after dismissal?
[ ] Are delegates weak?
[ ] Do stored closures capture self weakly when needed?
[ ] Are timers invalidated?
[ ] Are tasks cancelled when no longer needed?
[ ] Are Combine subscriptions released?
[ ] Are image/data caches bounded?
[ ] Are large images downsampled?
[ ] Are notification observers cleaned up?
[ ] Are unsafe APIs avoided or isolated?
```

## Ownership Graph Artifact

Before fixing, draw the expected ownership graph.

```text
ProfileViewController
  -> ProfileViewModel
      -> ProfileService

ProfileService
  -> URLSession

ProfileViewModel
  x should not strongly retain ProfileViewController
```

Then compare this with Memory Graph.

## Interview Debugging Answer

Junior:

I would add `deinit` logs and check whether objects are released.

Mid-level:

I would use Xcode Memory Graph to find strong reference paths and Instruments Leaks/Allocations to observe memory growth.

Senior:

I would reproduce the lifecycle, define the expected ownership graph, verify deallocation with `deinit`, inspect retain paths with Memory Graph, and confirm with Instruments. Then I would fix ownership at the source, not just add weak everywhere. I would also review tasks, subscriptions, caches, and large resources.

## Common Mistakes

- Assuming every memory issue is a retain cycle.
- Adding weak references randomly.
- Forgetting caches and large resources.
- Ignoring tasks/subscriptions.
- Not reproducing the lifecycle before debugging.

## Interview Progression

Basic:

Use `deinit` logs to check whether objects are released.

Intermediate:

Use Memory Graph and Instruments to inspect retain paths, leaks, and allocations.

Advanced:

Debugging memory means proving ownership. I reproduce the lifecycle, define expected ownership, capture evidence, inspect retain paths, fix the incorrect owner, and retest. For memory growth, I also inspect caches, images, tasks, subscriptions, and large data buffers.

## Practice

1. Add deinit to a view model.
2. Create and fix a delegate cycle.
3. Use Memory Graph to inspect references.
4. Explain Leaks vs Allocations instruments.
