# Day 31: UserDefaults

## What UserDefaults Is

`UserDefaults` is a key-value persistence system for small app settings and preferences.

Use it for:

- Theme preference
- Last selected tab
- Sort option
- Onboarding completed flag
- Feature hint dismissed flag
- Small app configuration

```swift
UserDefaults.standard.set(true, forKey: "hasCompletedOnboarding")

let completed = UserDefaults.standard.bool(forKey: "hasCompletedOnboarding")
```

## What UserDefaults Is Not

Do not store:

- Passwords
- Auth tokens
- Personal sensitive data
- Large JSON blobs
- Images/videos/files
- Complex relational data

Apple explicitly notes that defaults are stored unencrypted on disk and should not hold personal or sensitive information.

## Strongly Typed Wrapper

Avoid scattering string keys everywhere.

```swift
enum DefaultsKey {
    static let selectedTheme = "selectedTheme"
    static let hasCompletedOnboarding = "hasCompletedOnboarding"
}

final class AppSettingsStore {
    private let defaults: UserDefaults

    init(defaults: UserDefaults = .standard) {
        self.defaults = defaults
    }

    var hasCompletedOnboarding: Bool {
        get { defaults.bool(forKey: DefaultsKey.hasCompletedOnboarding) }
        set { defaults.set(newValue, forKey: DefaultsKey.hasCompletedOnboarding) }
    }
}
```

## SwiftUI AppStorage

SwiftUI provides `@AppStorage` for simple defaults-backed UI state.

```swift
struct SettingsView: View {
    @AppStorage("selectedTheme") private var selectedTheme = "system"

    var body: some View {
        Picker("Theme", selection: $selectedTheme) {
            Text("System").tag("system")
            Text("Light").tag("light")
            Text("Dark").tag("dark")
        }
    }
}
```

Use this for simple preferences. For business-critical settings, consider a store abstraction so keys, migration, and tests stay controlled.

## Storing Enums

```swift
enum AppTheme: String, CaseIterable {
    case system
    case light
    case dark
}

var selectedTheme: AppTheme {
    get {
        let rawValue = defaults.string(forKey: DefaultsKey.selectedTheme) ?? AppTheme.system.rawValue
        return AppTheme(rawValue: rawValue) ?? .system
    }
    set {
        defaults.set(newValue.rawValue, forKey: DefaultsKey.selectedTheme)
    }
}
```

This is safer than passing raw strings around the app.

## Testing UserDefaults

Use a suite for tests.

```swift
let defaults = UserDefaults(suiteName: "SettingsStoreTests")!
defaults.removePersistentDomain(forName: "SettingsStoreTests")

let store = AppSettingsStore(defaults: defaults)
store.hasCompletedOnboarding = true

#expect(store.hasCompletedOnboarding)
```

Do not let tests mutate `.standard`.

## Migration Basics

Defaults keys are persistent API. Renaming a key loses the old value unless migrated.

```swift
func migrateIfNeeded() {
    if defaults.object(forKey: "theme") != nil,
       defaults.object(forKey: "selectedTheme") == nil {
        defaults.set(defaults.string(forKey: "theme"), forKey: "selectedTheme")
        defaults.removeObject(forKey: "theme")
    }
}
```

## Common Mistakes

- Storing tokens in UserDefaults.
- Storing large arrays or JSON documents.
- String keys duplicated across codebase.
- No migration for renamed keys.
- Tests using `.standard`.
- Treating defaults as a database.

## Senior iOS Engineer Perspective

Ask:

- Is this small, nonsensitive preference data?
- Is the key stable?
- Do tests use isolated defaults?
- Should this be wrapped behind a settings store?
- Is there a migration path for renamed/changed values?

## Interview Notes

Junior:

`UserDefaults` stores small key-value app preferences.

Mid-level:

Use it for nonsensitive settings, wrap keys, and avoid large data.

Senior:

I treat UserDefaults as a small preferences store, not a secure or relational database. I wrap access, test with suites, and migrate keys deliberately.

## Practice

1. Store a theme enum in UserDefaults.
2. Wrap defaults in a `SettingsStore`.
3. Write a test using a suite.
4. Explain why auth tokens do not belong in UserDefaults.
