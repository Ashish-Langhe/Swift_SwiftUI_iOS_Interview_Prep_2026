# Day 32: Allocations And Memory Footprint

## What Allocations Shows

The Allocations instrument tracks heap and anonymous virtual memory allocations.

Use it to answer:

```text
What is allocating memory?
When is memory allocated?
Does memory return after leaving a feature?
Which allocation category is growing?
```

Apple recommends Allocations for tracking memory categories, individual allocations, allocation times, and responsible code.

## Memory Footprint vs Allocations

Allocation:

Memory requested by your app.

Footprint:

Memory the system charges your app for, including dirty resident/compressed pages.

An app can allocate virtual memory without immediately increasing physical footprint. iOS memory pressure cares about footprint.

## Memory Gauge

Xcode's memory report shows current memory and high-water memory.

Important simulator caveat:

The simulator does not receive iOS-style memory warnings/jetsam in the same way. Green simulator memory does not prove device-safe memory.

Always test memory-sensitive flows on device.

## Allocations Workflow

```text
1. Launch app in Instruments Allocations
2. Navigate to stable baseline
3. Mark Generation
4. Perform feature action
5. Mark Generation
6. Navigate away
7. Mark Generation
8. Inspect allocations between generations
9. Check whether memory returns
```

Generations isolate feature-specific memory.

## Example: Image Memory Growth

Problem:

```swift
@Observable
final class GalleryModel {
    var images: [UIImage] = []
}
```

Loading many full-resolution images can explode memory.

Better:

```swift
struct ImageReference: Identifiable {
    let id: UUID
    let thumbnailURL: URL
}
```

Use thumbnail decoding, caching, and avoid storing many full-size `UIImage`s in state.

## Autorelease Pools

In loops that create many temporary Objective-C objects, autorelease pools can help.

```swift
for url in imageURLs {
    autoreleasepool {
        let image = UIImage(contentsOfFile: url.path)
        process(image)
    }
}
```

Use after measuring. Swift value code often does not need this, but UIKit/Foundation image/file processing may.

## Caches

Caches must have limits.

```swift
let cache = NSCache<NSURL, UIImage>()
cache.countLimit = 100
cache.totalCostLimit = 50 * 1024 * 1024
```

Cache growth is not always a leak, but unlimited cache growth is still a production problem.

## Large Allocations

Investigate:

- Images
- Data buffers
- JSON payloads
- Video/audio buffers
- Swift arrays
- Cached view models
- Database result sets
- PDF rendering

## Reducing Memory

Techniques:

- Load thumbnails, not full images.
- Stream large files.
- Paginate data.
- Clear caches on memory warning.
- Avoid duplicate decoded images.
- Use value types carefully.
- Avoid storing huge arrays in UI state.
- Release feature state when flow ends.

## Common Mistakes

- Looking only for leaks, ignoring large legitimate allocations.
- Profiling memory only on simulator.
- No generation marks.
- Storing full-size images in arrays.
- Unlimited caches.
- Loading entire files into memory.
- Holding onto old screen state after navigation.

## Senior Artifact

```text
Artifact: Allocation Investigation
Feature:
Device:
Baseline memory:
Peak memory:
Generation marks:
Largest categories:
Largest allocation sites:
Memory returns after exit:
Fix:
Verification:
```

## Interview Notes

Junior:

Allocations shows memory allocated by the app.

Mid-level:

Use generations to isolate allocations during a feature and check whether memory returns.

Senior:

I analyze memory footprint, allocation categories, generation deltas, cache policy, image/data buffers, and device behavior. I distinguish leaks from legitimate but excessive memory.

## Practice

1. Use generation marks around opening/closing a gallery.
2. Add limits to an image cache.
3. Replace full-size images with thumbnails.
4. Explain footprint vs allocation.
