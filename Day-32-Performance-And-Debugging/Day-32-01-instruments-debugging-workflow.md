# Day 32: Instruments Debugging Workflow

## What Instruments Is

Instruments is Apple's performance analysis tool for profiling apps.

Use it to investigate:

- CPU usage
- Hangs
- Hitches
- Memory growth
- Leaks
- Allocations
- SwiftUI view update cost
- Swift concurrency scheduling
- File/network/system blocking
- Energy usage

Instruments is not just a "slow app" tool. It is the senior engineer's evidence tool.

## Core Workflow

```text
1. Reproduce
2. Record baseline
3. Identify bottleneck
4. Form hypothesis
5. Make one fix
6. Record again
7. Compare runs
8. Keep or revert change based on evidence
```

Do not optimize from vibes. Optimize from traces.

## Product Profile Entry

In Xcode:

```text
Product -> Profile
```

Shortcut:

```text
Command-I
```

Choose an Instruments template based on the symptom.

## Which Instrument To Start With

```text
App hangs during tap:
Time Profiler, System Trace, Swift Concurrency

Scrolling is janky:
SwiftUI, Time Profiler, Core Animation, System Trace

Memory grows:
Allocations, Leaks, Memory Graph Debugger

App killed in background:
Allocations, VM Tracker, memory report, jetsam reports

Async tasks stuck:
Swift Concurrency instrument, Time Profiler, debugger task context

Battery drain:
Power Profiler, Energy Log
```

## Repro Script

Before profiling, write down exact steps.

```text
Scenario: Product search scroll hitch
Device: iPhone 16 Pro
Build: Release/Profile
Steps:
1. Launch app
2. Login with test account
3. Open Products tab
4. Type "keyboard"
5. Scroll from top to bottom twice
Expected problem:
Scroll drops frames after images load
```

If you cannot reproduce consistently, your trace will be noisy.

## Release-Like Builds

Debug builds can distort performance. For serious profiling, use a release/profile configuration with symbols.

Debug builds are useful for diagnosis and assertions. Release-like builds are better for performance truth.

## Add Signposts

Signposts help connect code to trace intervals.

```swift
import os

private let logger = Logger(subsystem: "com.example.shop", category: "Products")

func loadProducts() async {
    logger.info("Products load started")
    defer { logger.info("Products load finished") }

    // fetch and map products
}
```

For detailed intervals, use `OSSignposter`.

```swift
import os

let signposter = OSSignposter(subsystem: "com.example.shop", category: "Checkout")

func saveOrder() async throws {
    let state = signposter.beginInterval("Save Order")
    defer { signposter.endInterval("Save Order", state) }

    try await repository.saveOrder()
}
```

Signposts make traces much easier to read.

## Baseline And Comparison

Always preserve a baseline run.

```text
Baseline:
Search scroll hitch: 420 ms main-thread spike

Fix:
Move image resizing off main actor

Comparison:
Main-thread spike reduced to 40 ms
Scroll hitch gone
```

Recent Instruments workflows make run comparisons and top-function analysis more useful. Senior engineers use this to prove that a fix worked.

## Common Debugging Mistakes

- Profiling debug builds only.
- Changing multiple things before re-measuring.
- Looking only at average CPU instead of specific hitches.
- Ignoring device differences.
- Profiling simulator memory as if it matches iPhone memory pressure.
- Optimizing code that is not on the hot path.
- No baseline trace.
- No reproduction script.

## Senior iOS Engineer Artifact

```text
Artifact: Performance Investigation Report
Issue:
Device/OS:
Build configuration:
Repro steps:
Baseline trace:
Symptom:
Primary instrument:
Finding:
Hypothesis:
Fix:
Verification trace:
Result:
Remaining risk:
```

## Interview Notes

Junior:

Instruments helps profile performance and memory.

Mid-level:

Choose the instrument based on the symptom and compare before/after traces.

Senior:

I use a repeatable profile-fix-verify workflow with release-like builds, signposts, baseline traces, targeted instruments, and run comparisons. I do not optimize without measured evidence.

## Practice

1. Write a repro script for a slow search screen.
2. Add signposts around a network-to-render pipeline.
3. Capture baseline and fixed traces.
4. Explain why profiling debug builds can mislead.
