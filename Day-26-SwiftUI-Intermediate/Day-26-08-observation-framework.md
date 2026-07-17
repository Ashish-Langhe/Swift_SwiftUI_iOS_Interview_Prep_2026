# Day 26: Observation Framework

## What Observation Is

Observation is Swift's modern system for tracking changes in observable models and updating dependent views.

In SwiftUI, it commonly appears through:

- `@Observable`
- `@Bindable`
- `@State` owning observable models
- Environment-injected observable models

## Basic `@Observable`

```swift
import Observation

@Observable
final class ProfileModel {
    var name = ""
    var bio = ""
    var isPublic = true
}
```

Use in a view:

```swift
struct ProfileScreen: View {
    @State private var model = ProfileModel()

    var body: some View {
        VStack {
            Text(model.name)
            Text(model.bio)
        }
    }
}
```

When observed properties change, SwiftUI updates views that depend on them.

## `@Bindable`

Use `@Bindable` to create bindings to observable properties.

```swift
struct ProfileEditor: View {
    @Bindable var model: ProfileModel

    var body: some View {
        Form {
            TextField("Name", text: $model.name)
            TextField("Bio", text: $model.bio, axis: .vertical)
            Toggle("Public", isOn: $model.isPublic)
        }
    }
}
```

Without `@Bindable`, you can read the model but cannot use `$model.name` binding syntax in that view.

## Environment with Observable Models

```swift
@Observable
final class SessionModel {
    var user: User?
    var isLoggedIn: Bool { user != nil }
}

@main
struct AppMain: App {
    @State private var session = SessionModel()

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(session)
        }
    }
}
```

Read:

```swift
struct AccountView: View {
    @Environment(SessionModel.self) private var session

    var body: some View {
        Text(session.user?.name ?? "Guest")
    }
}
```

For bindings:

```swift
struct AccountEditor: View {
    @Environment(SessionModel.self) private var session

    var body: some View {
        @Bindable var session = session
        Toggle("Logged in", isOn: $session.isLoggedInDebugFlag)
    }
}
```

In real code, avoid editable debug flags like this unless the property is genuinely user-editable.

## Observation vs ObservableObject

`ObservableObject`:

```swift
final class OldModel: ObservableObject {
    @Published var name = ""
}
```

Observation:

```swift
@Observable
final class NewModel {
    var name = ""
}
```

Observation reduces boilerplate and supports more precise dependency tracking.

## Migration Strategy

Do not migrate the entire app blindly.

Good migration steps:

1. Start with one feature model.
2. Replace `ObservableObject` and `@Published` with `@Observable`.
3. Replace `@StateObject` ownership with `@State` where appropriate.
4. Replace `@ObservedObject` pass-through with plain parameters or `@Bindable` when bindings are needed.
5. Replace environment object usage with type-based environment injection when appropriate.
6. Test view updates, form bindings, previews, and navigation.

## Main Actor and Observation

Observable UI models often belong on the main actor.

```swift
@Observable
@MainActor
final class CheckoutModel {
    var state: CheckoutState = .idle

    func submit() async {
        state = .submitting
        // await service call
        state = .complete
    }
}
```

This keeps UI mutations isolated.

## Ignoring Observation

Some properties should not trigger observation.

Conceptually:

```swift
@Observable
final class SearchModel {
    var query = ""
    var results: [Product] = []

    @ObservationIgnored
    private let service: SearchService
}
```

Dependencies are not UI state.

## Binding Concurrency Caution

SwiftUI documentation notes that a `Binding` may be sendable to pass, but reading or writing the wrapped value across concurrency domains depends on how it was created. Practically: mutate UI-bound observable state from the right actor and avoid treating bindings as thread-safe domain primitives.

## Real iOS Example: Settings Model

```swift
@Observable
@MainActor
final class SettingsModel {
    var notificationsEnabled = false
    var selectedTheme: AppTheme = .system
    var isSaving = false
    var errorMessage: String?

    @ObservationIgnored
    private let service: SettingsService

    init(service: SettingsService) {
        self.service = service
    }

    func save() async {
        isSaving = true
        defer { isSaving = false }

        do {
            try await service.save(
                notificationsEnabled: notificationsEnabled,
                selectedTheme: selectedTheme
            )
        } catch {
            errorMessage = "Could not save settings."
        }
    }
}
```

View:

```swift
struct SettingsView: View {
    @State private var model: SettingsModel

    var body: some View {
        @Bindable var model = model

        Form {
            Toggle("Notifications", isOn: $model.notificationsEnabled)
            Picker("Theme", selection: $model.selectedTheme) {
                ForEach(AppTheme.allCases) { theme in
                    Text(theme.title).tag(theme)
                }
            }

            Button("Save") {
                Task { await model.save() }
            }
            .disabled(model.isSaving)
        }
    }
}
```

## Common Mistakes

- Mixing `@Observable` with `@ObservedObject` incorrectly.
- Forgetting `@Bindable` for form bindings.
- Putting services in observable state without ignoring observation.
- Mutating observable UI state off the intended actor.
- Migrating everything without testing.
- Using one global observable model for unrelated features.

## Senior iOS Engineer Perspective

Senior Observation usage is about lifecycle and boundaries:

- Who owns the model?
- Which views read which properties?
- Which properties are UI state?
- Which dependencies should be ignored?
- Are mutations actor-safe?
- Is binding access limited to UI editing?
- Can previews and tests create the model easily?

## Interview Notes

Junior:

Observation lets SwiftUI update when model properties change.

Mid-level:

Use `@Observable` for models and `@Bindable` when a view needs bindings to model properties.

Senior:

I use Observation to reduce boilerplate and improve data-flow precision, but I still design ownership, actor isolation, dependency injection, and migration carefully. Observation does not replace architecture.

## Practice

1. Convert an `ObservableObject` model to `@Observable`.
2. Use `@Bindable` in a form.
3. Inject an observable session model through environment.
4. Mark a service dependency as ignored by observation.
