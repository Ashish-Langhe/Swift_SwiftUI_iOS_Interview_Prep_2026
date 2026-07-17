# Day 26: AppStorage, SceneStorage, And Persistence Boundaries

## Why This Topic Matters

SwiftUI has convenient persistence wrappers, but they are easy to misuse.

Common wrappers:

- `@AppStorage`
- `@SceneStorage`

They are great for small UI preferences. They are not a replacement for a database, secure storage, or domain persistence.

## `@AppStorage`

`@AppStorage` reads and writes values from user defaults.

```swift
struct SettingsView: View {
    @AppStorage("notificationsEnabled") private var notificationsEnabled = true

    var body: some View {
        Form {
            Toggle("Notifications", isOn: $notificationsEnabled)
        }
    }
}
```

Use it for simple preferences.

Good examples:

- Theme preference
- Last selected tab
- Feature hint dismissed
- Sort option
- Small UI preference

## Avoid Sensitive Data in `@AppStorage`

Do not store:

- Auth tokens
- Passwords
- Payment data
- Personal secrets

Use Keychain or a secure storage abstraction for sensitive data.

## `@SceneStorage`

`@SceneStorage` preserves UI state for a scene.

```swift
struct SearchView: View {
    @SceneStorage("search.query") private var query = ""

    var body: some View {
        List(results) { result in
            SearchResultRow(result: result)
        }
        .searchable(text: $query)
    }
}
```

This is useful for restoring temporary UI state when a scene is recreated.

## AppStorage vs SceneStorage

`@AppStorage`:

- App-wide user defaults
- Persists across launches
- Shared by app instances
- Good for preferences

`@SceneStorage`:

- Scene-specific UI restoration
- Useful for multiwindow
- Good for temporary view state

## Persistence Boundary

For important domain data, use a service/repository.

```swift
protocol SettingsStore {
    var selectedTheme: AppTheme { get async }
    func saveSelectedTheme(_ theme: AppTheme) async throws
}
```

Then expose it through a model:

```swift
@Observable
@MainActor
final class SettingsModel {
    var selectedTheme: AppTheme = .system

    @ObservationIgnored
    private let store: SettingsStore

    init(store: SettingsStore) {
        self.store = store
    }

    func load() async {
        selectedTheme = await store.selectedTheme
    }

    func save() async throws {
        try await store.saveSelectedTheme(selectedTheme)
    }
}
```

This is more testable than scattering storage wrappers everywhere.

## Migration Risk

If you change keys, you may lose user settings.

```swift
@AppStorage("selectedTheme") private var selectedTheme = "system"
```

Treat keys as persistent API. Rename carefully.

## Common Mistakes

- Storing secrets in user defaults.
- Using `@AppStorage` for complex domain models.
- Duplicating the same storage key in many views.
- Forgetting key migration.
- Using scene storage for app-wide preferences.
- Making storage wrappers the architecture.

## Senior iOS Engineer Perspective

Senior persistence thinking:

- Is this UI preference, scene restoration, secure data, or domain data?
- Who owns the persistence contract?
- Are keys stable?
- Can this be tested?
- Does multiwindow behavior matter?
- Is the data sensitive?
- Should this be behind a store/repository?

## Interview Notes

Junior:

`@AppStorage` stores small values in user defaults. `@SceneStorage` stores scene-specific UI state.

Mid-level:

Use `@AppStorage` for preferences and `@SceneStorage` for restoration-like state.

Senior:

I use SwiftUI storage wrappers for small UI state only. For domain, secure, or business-critical persistence, I use explicit storage abstractions with migration and test strategy.

## Practice

1. Store a theme preference using `@AppStorage`.
2. Preserve search text using `@SceneStorage`.
3. Move domain persistence behind a `SettingsStore`.
4. Explain why auth tokens do not belong in `@AppStorage`.
