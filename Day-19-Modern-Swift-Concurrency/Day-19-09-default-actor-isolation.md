# Day 19: Default Actor Isolation

## What Default Actor Isolation Means

Default actor isolation is a Swift 6.2 configuration that can infer unannotated code as isolated to a global actor, usually `MainActor`.

In SwiftPM, the setting can use `defaultIsolation(MainActor.self)`.

## Why It Matters

UI code is often main-actor code.

Without default isolation, developers may need many annotations:

```swift
@MainActor
final class AppModel { }

@MainActor
func updateUI() { }
```

Default isolation reduces this boilerplate in appropriate targets.

## Mental Model

Instead of:

```text
Everything is nonisolated unless annotated.
```

You can configure:

```text
This target's unannotated code defaults to MainActor.
```

## What It Does Not Mean

It does not mean:

- All code should run on the main actor
- Heavy work is safe on the main actor
- Data races are impossible
- Public API design no longer matters
- You never need explicit annotations

## Real iOS Example

App target:

```swift
final class AppState: ObservableObject {
    @Published var selectedTab: Tab = .home
}
```

With main actor default isolation, this can be inferred as main-actor isolated in the configured target.

Service target:

```swift
actor APIThrottle { }
```

You may not want service packages defaulting to main actor. Choose per target.

## Target-Level Thinking

Good candidates:

- App executable target
- UI feature targets
- Scripts
- Preview/demo targets

Be careful with:

- Networking packages
- Data processing packages
- Server-side code
- Shared libraries

## Common Mistakes

- Enabling main actor default everywhere
- Forgetting package consumers may use different settings
- Hiding expensive work on the main actor
- Not documenting public API isolation
- Assuming behavior is identical across toolchain settings

## Modern Swift 6.2 Notes

SwiftPM exposes default isolation configuration starting with PackageDescription 6.2. The compiler defaults to nonisolated when unspecified.

This is one of the most important Swift 6.2 concurrency updates for iOS developers.

## Junior Interview Answer

Default actor isolation lets a target infer unannotated code as isolated to a global actor like `MainActor`.

## Mid-Level Interview Answer

It reduces `@MainActor` boilerplate for UI-heavy targets, but services and CPU-heavy code still need thoughtful isolation.

## Senior Interview Answer

Default actor isolation is a target-level migration and ergonomics tool. I use it selectively for UI/application targets, keep library/service modules explicit, and review public APIs because isolation behavior is part of the client contract.

## Practice

1. Decide whether an app target should use main actor default isolation.
2. Explain why a networking package should usually not default to main actor.
3. Identify code that should be marked `@concurrent`.
4. Explain how this helps Swift 6 migration.
