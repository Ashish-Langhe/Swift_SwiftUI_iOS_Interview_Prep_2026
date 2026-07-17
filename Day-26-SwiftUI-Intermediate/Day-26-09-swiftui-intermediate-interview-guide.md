# Day 26: SwiftUI Intermediate Interview Guide

## One-Minute Senior Answer

Intermediate SwiftUI is where syntax turns into product engineering. Lists need stable identity and efficient rows. Forms need draft state, validation, focus, and submission flows. Sheets and alerts should be modeled as explicit presentation state. Animations should describe meaningful state transitions and respect accessibility. Custom components need intentional APIs. MVVM should be used pragmatically, and the Observation framework should clarify ownership rather than hide it.

## Core Mental Models

- Lists are identity-driven containers.
- Forms are state machines.
- Sheets are explicit modal presentation state.
- Alerts are interruption state and should be rare.
- Animations are state transitions.
- Components are API design.
- MVVM is a tool, not a requirement.
- Observation tracks model changes but does not replace architecture.

## Junior Questions

What is `List`?

A SwiftUI container for displaying rows of data.

What is `Form`?

A container for structured input and settings screens.

What is a sheet?

A modal presentation for temporary tasks.

What is an alert?

A focused message or confirmation shown to the user.

What is `@Observable`?

A macro that makes a model observable by SwiftUI.

## Mid-Level Questions

How do you handle multiple sheets?

Use an identifiable enum instead of many booleans.

How do you validate a form?

Use draft state, computed validation, disabled submit buttons, inline messages, and backend error handling.

How do you animate a SwiftUI view?

Change state inside `withAnimation` or attach `.animation(_:value:)` to a specific state change.

How do you design a reusable component?

Make its API describe intent, use bindings only when editing parent-owned state, callbacks for actions, and previews for important states.

## Senior Questions

How do you design a production list screen?

I start with stable model identity, explicit loading/empty/error states, lightweight rows, search/filter strategy, row actions, selection behavior, accessibility, and performance with realistic data.

How do you design a form flow?

I use a draft model, define validation rules, handle dirty state, prevent duplicate submission, manage focus, support cancellation/discard, and keep domain validation outside random view code.

How do you decide between sheet and navigation?

A sheet is for temporary, interruptible work. Navigation is for moving deeper into the user's primary task. If the flow is long, deep-linkable, or core to the app hierarchy, it likely belongs in navigation.

How do you use MVVM without overengineering?

I use ViewModels for non-trivial state transitions, async work, validation, and testability. I avoid ViewModels for tiny pure views and avoid giant ViewModels that own unrelated features.

How do you migrate to Observation?

I migrate feature by feature, replace `ObservableObject` and `@Published` with `@Observable`, use `@State` for owned models, use `@Bindable` for form bindings, review environment injection, and test update behavior.

## Senior iOS Engineer Artifact

```text
Artifact: SwiftUI Intermediate Feature Review
Feature:
List identity:
List row performance:
Search/filter strategy:
Form draft model:
Validation rules:
Focus/keyboard flow:
Sheet state model:
Alert/error model:
Animation states:
Custom components:
ViewModel ownership:
Observation usage:
Environment dependencies:
Accessibility checks:
Dynamic type:
Preview matrix:
Test evidence:
Known risks:
```

## Real Feature Scenario

Feature: Settings screen with editable profile, notification preferences, and account deletion.

Design:

- `SettingsModel` is `@Observable`.
- `Form` renders profile and notification sections.
- Profile editing uses a sheet with draft state.
- Account deletion uses a destructive alert.
- Save button disables during async submission.
- Environment provides analytics and session.
- Previews cover logged-in, saving, validation error, and large text.

Sketch:

```swift
@Observable
@MainActor
final class SettingsModel {
    var profile: Profile
    var notificationsEnabled: Bool
    var activeSheet: ActiveSheet?
    var activeAlert: ActiveAlert?
    var isSaving = false

    @ObservationIgnored
    private let service: SettingsService

    init(profile: Profile, service: SettingsService) {
        self.profile = profile
        self.notificationsEnabled = profile.notificationsEnabled
        self.service = service
    }

    func saveNotifications() async {
        isSaving = true
        defer { isSaving = false }

        do {
            try await service.saveNotifications(enabled: notificationsEnabled)
        } catch {
            activeAlert = .error("Could not save notifications.")
        }
    }
}
```

Senior discussion:

- Presentation state is explicit.
- Service dependency is ignored by observation.
- UI-bound mutations are main-actor isolated.
- Alerts are display-safe.
- Sheets use draft state.
- The model is feature-scoped, not app-global.

## Latest SwiftUI Points to Mention

- Observation with `@Observable` and `@Bindable` is the modern data-flow path.
- Newer alert and confirmation dialog overloads support item/error-driven presentation.
- Reordering and swipe-action APIs continue expanding beyond classic list-only thinking.
- Binding has concurrency-safety considerations; do not mutate UI bindings from arbitrary domains.
- SwiftUI keeps improving presentation, layout, toolbar, animation, and container APIs, but fundamentals still decide correctness.

## Common Traps

- Treating `List` like a simple `VStack`.
- Using unstable row IDs.
- Mutating saved data directly in forms.
- Using many booleans for sheets and alerts.
- Overusing alerts.
- Animating without stable identity.
- Creating components with too many styling parameters.
- Creating ViewModels inside `body`.
- Thinking Observation automatically fixes architecture.

## Strong Senior Closing Answer

For intermediate SwiftUI, I focus on state ownership, identity, presentation modeling, and user workflow. Lists, forms, sheets, alerts, animations, components, MVVM, and Observation all work well when the data flow is explicit and the lifecycle is predictable. Most SwiftUI bugs are not syntax bugs; they are ownership, identity, or state-transition bugs.

## Practice Prompts

1. Design a list with search, sections, swipe actions, and empty/error states.
2. Build a form with draft state, validation, focus, and async save.
3. Replace multiple sheet booleans with one enum.
4. Replace multiple alert booleans with item-driven alert state.
5. Add reduce-motion support to an animation.
6. Design a reusable component API and preview matrix.
7. Build an `@Observable` ViewModel and test async loading.
8. Explain MVVM tradeoffs in SwiftUI to a senior interviewer.
