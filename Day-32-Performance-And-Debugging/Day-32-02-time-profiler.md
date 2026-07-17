# Day 32: Time Profiler

## What Time Profiler Is

Time Profiler samples your app's call stacks to show where CPU time is spent.

Use it for:

- Slow taps
- CPU spikes
- Slow launch
- Janky scrolling
- Expensive calculations
- JSON parsing cost
- Image processing cost
- Date/number formatting hotspots

It answers:

```text
What code is using CPU during this slow moment?
```

## Sampling Mental Model

Time Profiler samples stacks at intervals. If a function appears often, it likely consumed meaningful CPU time.

This is not line-by-line tracing. It is statistical profiling.

## Important Views

Call Tree:

- Shows functions and stack relationships.

Top Functions:

- Surfaces heavy functions quickly.

Heaviest Stack Trace:

- Helps find hot call paths.

Timeline:

- Lets you select the exact slow interval.

## First Steps

1. Reproduce the slow action.
2. Stop recording.
3. Select the exact time interval of the issue.
4. Inspect top functions and call tree.
5. Hide system libraries only when needed.
6. Look for your app code.
7. Verify with another run after changes.

## Common Call Tree Filters

Useful options:

```text
Separate by Thread
Invert Call Tree
Hide System Libraries
Flatten Recursion
```

Use `Invert Call Tree` to see leaf hot functions.

Use `Separate by Thread` to understand main-thread vs background work.

## Example: Expensive Sorting In Body

Problem:

```swift
var body: some View {
    List(products.sorted { rank($0) > rank($1) }) { product in
        ProductRow(product: product)
    }
}
```

Time Profiler may show:

```text
rank(_:)
sorted(by:)
ProductsView.body
```

Fix:

```swift
@Observable
@MainActor
final class ProductsModel {
    private(set) var visibleProducts: [ProductRowViewModel] = []

    func update(products: [Product]) async {
        let rows = await ProductMapper.rows(from: products)
        visibleProducts = rows.sorted { $0.rank > $1.rank }
    }
}
```

Move expensive work out of repeated view updates.

## Example: DateFormatter Hotspot

Bad:

```swift
struct OrderRow: View {
    let order: Order

    var body: some View {
        Text(DateFormatter.localizedString(
            from: order.date,
            dateStyle: .medium,
            timeStyle: .none
        ))
    }
}
```

If thousands of rows format dates repeatedly, CPU can spike.

Better:

```swift
struct OrderRowViewModel: Identifiable {
    let id: Order.ID
    let dateText: String
}
```

Format once during mapping or use efficient formatting strategies.

## Main Thread In Time Profiler

If the main thread is busy during a hang, inspect what it is doing.

Common main-thread offenders:

- JSON decoding
- Image resizing
- File writes
- Sorting large arrays
- Synchronous database queries
- Heavy SwiftUI body work
- Date/number formatting
- Regex over large data

## Async Code In Time Profiler

Async code can hop between threads. Swift 6.2 and current Xcode tooling improve async stepping and task context visibility, but in Time Profiler you still need to inspect call stacks and executor tracks carefully.

Use:

- Swift Concurrency instrument
- Time Profiler
- System Trace
- Task names
- Signposts

## Senior Time Profiler Checklist

```text
Symptom:
Selected interval:
Main thread busy?
Top app functions:
Hot stack:
Algorithm issue:
Repeated work:
Background work needed:
Expected improvement:
Verified after fix:
```

## Common Mistakes

- Looking at whole trace instead of problem interval.
- Blaming the top system function without checking caller.
- Optimizing cold code.
- Ignoring main-thread context.
- Not re-profiling after fix.
- Forgetting debug builds distort results.

## Interview Notes

Junior:

Time Profiler shows where CPU time is spent.

Mid-level:

Select the slow interval, inspect call trees, and identify expensive app functions.

Senior:

I use Time Profiler with precise intervals, call tree filtering, signposts, and before/after comparisons to prove CPU bottlenecks and verify fixes.

## Practice

1. Find an expensive function in Time Profiler.
2. Move date formatting out of a row body.
3. Compare main-thread and background-thread samples.
4. Explain inverted call tree.
