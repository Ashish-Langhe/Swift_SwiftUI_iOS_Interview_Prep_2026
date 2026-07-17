# Day 27: SwiftUI Advanced Interview Guide

## One-Minute Senior Answer

Advanced SwiftUI is about controlling complexity. I use custom layout when built-in containers cannot express reusable placement rules, preferences and anchors for measured parent-child coordination, typed route state for navigation and deep links, Instruments for performance, representables for UIKit boundaries, Canvas and TimelineView for custom drawing, and explicit dependency/data-flow architecture for testability. The goal is not clever SwiftUI. The goal is predictable, accessible, performant UI that scales across product requirements.

## Core Mental Models

- Custom layout encodes placement rules.
- Geometry is an escape hatch, not a layout strategy.
- Preferences move layout data upward.
- Navigation is route state.
- Performance must be measured.
- Representables are boundary adapters.
- Drawing APIs reduce view explosion.
- Animation communicates state changes.
- Dependency injection keeps SwiftUI testable.
- Async UI is a state machine.

## Junior-To-Mid Bridge Questions

Why not use `GeometryReader` everywhere?

Because it expands to available space and can create brittle layouts. Use stacks/grids first, custom `Layout` for reusable layout rules, and geometry only for measurement.

Why are stable IDs important?

They help SwiftUI understand which items are the same across updates, preserving state, animations, scrolling, and performance.

What is a representable?

A wrapper that lets UIKit/AppKit views or controllers appear in SwiftUI.

## Senior Questions

How do you diagnose SwiftUI performance?

I reproduce the slow interaction, profile with the SwiftUI Instruments template, inspect long body updates or frequent updates, use Time Profiler for expensive work, change one thing, then re-profile. I avoid guessing.

How do you design deep linking?

I parse URLs into typed route values, validate them, handle authentication gates, route to the correct tab/stack, and preserve intended destination after login. Route values should be stable and usually ID-based.

When do you build a custom layout?

When built-in containers cannot express a reusable placement algorithm cleanly. I avoid custom layout for one-off spacing problems and test with dynamic type, localization, and different container sizes.

How do you use UIKit in SwiftUI safely?

I wrap UIKit with representables, use coordinators for delegates, make updates idempotent, avoid recreating platform views unnecessarily, keep memory safe, and profile long platform updates.

How do you design advanced async UI?

I model loading, refresh, pagination, search, retry, cancellation, optimistic updates, and stale response prevention explicitly. UI mutations are main-actor isolated.

## Senior iOS Engineer Artifact

```text
Artifact: Advanced SwiftUI Architecture Review
Feature:
Custom layout needed:
Geometry/preference usage:
Navigation routes:
Deep link behavior:
Split view/iPad behavior:
Async state machine:
Cancellation strategy:
UIKit interop boundary:
Animation purpose:
Drawing/performance risk:
Dependency injection:
Environment usage:
Accessibility:
Dynamic type/localization:
Instruments evidence:
Test coverage:
Preview matrix:
Tradeoffs:
```

## Real Senior Scenario

Feature: Product discovery screen.

Requirements:

- Adaptive grid.
- Filter chips that wrap.
- Search with debounce.
- Product detail deep links.
- Favorite optimistic update.
- Animated card expansion.
- Analytics.
- Empty/error/loading states.
- Large data performance.

Senior design:

- `FlowLayout` for filter chips.
- `LazyVGrid` for products.
- `SearchModel` handles debounce and cancellation.
- `AppRoute.product(id)` handles navigation.
- Favorites update optimistically with rollback.
- Matched geometry uses stable product IDs.
- Analytics injected through environment.
- Product rows receive preformatted row models.
- Instruments verifies scrolling.
- Previews cover long names and large text.

## Latest SwiftUI Points To Mention

Recent SwiftUI updates strengthen advanced areas:

- `ContentBuilder` unifies type-agnostic content closure building.
- Reordering and swipe actions extend to more containers and custom layouts.
- Newer navigation overloads improve item-driven navigation.
- Split view column control continues improving.
- Drawing APIs include mesh gradients, shader compilation, text rendering effects, and color mixing.
- SwiftUI Instruments help diagnose long body updates, platform updates, and frequent update causes.

## Common Advanced Traps

- Custom layout too early.
- Geometry feedback loops.
- Route models carrying mutable objects.
- Performance guessing.
- Representables doing full resets in every update.
- Animation hiding slow state transitions.
- Environment used as global state.
- Async work without cancellation.
- Preview states too clean.
- No accessibility alternative for custom interactions.

## Strong Senior Closing Answer

At advanced level, SwiftUI is about engineering constraints: layout, identity, navigation, data flow, async work, performance, interop, accessibility, and testing. I reach for advanced APIs only when they solve a real problem, and I validate the result with previews, tests, and Instruments.

## Practice Prompts

1. Build a custom flow layout for tags.
2. Add a tab underline using anchor preferences.
3. Implement a deep link router.
4. Profile and fix a slow list row.
5. Wrap a UIKit control with `UIViewRepresentable`.
6. Draw a waveform with `Canvas`.
7. Add debounced search with stale response protection.
8. Create an advanced SwiftUI review checklist for a feature.
