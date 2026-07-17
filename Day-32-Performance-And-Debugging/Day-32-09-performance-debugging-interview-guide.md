# Day 32: Performance And Debugging Interview Guide

## One-Minute Senior Answer

Performance debugging is a measurement workflow. I reproduce the issue, capture a baseline trace, choose the right instrument, identify the hot interval, fix one bottleneck, and verify with a comparison trace. For CPU I use Time Profiler. For memory growth and leaks I use Allocations, Memory Graph, Leaks, and memory reports. For SwiftUI I inspect long body updates and frequent update causes. For concurrency I inspect task scheduling, actor contention, task context, and Main Actor pressure. I care about evidence, not guesses.

## Tool Map

```text
Instruments:
Overall profiling environment

Time Profiler:
CPU hotspots and call stacks

Memory Graph:
Retain cycles and leaked objects

Allocations:
Heap/VM allocations and memory growth

SwiftUI Instrument:
Long/frequent view updates

Swift Concurrency Instrument:
Tasks, actors, executor/thread behavior

System Trace:
Thread scheduling and blocking

Power Profiler:
Battery/thermal impact
```

## Junior Questions

What is Instruments?

Apple's profiling tool for performance, memory, and responsiveness.

What is Time Profiler?

An instrument that samples call stacks to show where CPU time is spent.

What is Memory Graph?

An Xcode debugger view that shows objects and references to find retain cycles.

## Mid-Level Questions

How do you debug a memory leak?

Reproduce the flow, leave the screen, capture memory graph, search for objects that should be gone, inspect retain paths, fix ownership, and verify deallocation.

How do you debug janky scrolling?

Profile with SwiftUI/Time Profiler, select hitch intervals, inspect long row/body updates, unstable IDs, image loading, main-thread work, and update frequency.

How do you debug high allocations?

Use Allocations with generation marks around the feature, inspect allocation categories and stack traces, then reduce large buffers/caches/duplicates.

## Senior Questions

How do you avoid optimizing the wrong thing?

I write reproducible steps, capture baseline traces, select the exact bad interval, identify hot app code, make one change, and compare traces after the fix.

How do you distinguish leak from cache growth?

A leak is retained unintentionally after expected deallocation. Cache growth may be intentional but still needs limits and eviction. Memory Graph helps leaks; Allocations helps growth.

How do you handle Main Actor performance?

I inspect whether CPU/blocking work is running on Main Actor, move heavy work to explicit concurrency boundaries, and keep only UI mutation on Main Actor.

How do Swift 6.2 async debugging improvements help?

They improve async stepping, task context visibility, backtraces, and named task support, making concurrent code easier to follow in LLDB and profiling tools.

## Senior Artifact: Performance Report

```text
Issue:
User-visible symptom:
Device/OS:
Build:
Repro steps:
Baseline trace:
Instrument:
Finding:
Root cause:
Fix:
Verification trace:
Improvement:
Remaining risk:
```

## Real Scenario: Product Gallery

Symptoms:

- Scrolling hitches.
- Memory grows after opening details repeatedly.
- Thumbnail generation blocks UI.

Investigation:

- SwiftUI instrument shows long row updates.
- Time Profiler shows image resizing on Main Actor.
- Allocations shows full-size images retained.
- Memory Graph shows details model retained by task.

Fixes:

- Precompute row view models.
- Generate thumbnails off Main Actor.
- Store image references instead of full images.
- Cancel task on disappear.
- Add cache limits.

Verification:

- Comparison trace shows reduced main-thread spikes.
- Allocations return after leaving feature.
- Memory Graph no longer shows leaked model.

## Common Interview Traps

- Saying "use Instruments" without workflow.
- Not knowing which instrument to use.
- Confusing leak with high memory.
- Ignoring simulator/device differences.
- Forgetting Main Actor inheritance.
- Optimizing without re-measuring.
- No SwiftUI-specific performance answer.
- No cancellation/async debugging answer.

## Strong Closing Answer

Senior performance work is disciplined: reproduce, measure, diagnose, fix, verify. The best engineers can explain not only what they changed, but which trace proved the change mattered.

## Practice Prompts

1. Explain how you debug a SwiftUI scrolling hitch.
2. Explain how you find a retain cycle.
3. Explain how you use Allocations generation marks.
4. Explain async task debugging in Swift 6.2.
5. Explain copy-on-write performance risk.
6. Write a performance investigation report.
