# Day 25: Environment

## What Environment Means

The SwiftUI environment is a value propagation system. Parent views can provide configuration or shared dependencies, and child views can read them without explicit parameters at every level.

```swift
struct ThemeAwareView: View {
    @Environment(\.colorScheme) private var colorScheme

    var body: some View {
        Text(colorScheme == .dark ? "Dark mode" : "Light mode")
    }
}
```

SwiftUI automatically provides many values:

- `colorScheme`
- `locale`
- `dynamicTypeSize`
- `scenePhase`
- `dismiss`
- `horizontalSizeClass`
- `accessibilityReduceMotion`

## Reading Environment Values

```swift
struct WelcomeView: View {
    @Environment(\.locale) private var locale
    @Environment(\.dynamicTypeSize) private var dynamicTypeSize

    var body: some View {
        VStack {
            Text("Locale: \(locale.identifier)")
            Text("Dynamic type: \(String(describing: dynamicTypeSize))")
        }
    }
}
```

Use environment for ambient context that many views may need.

## Setting Environment Values

```swift
ContentView()
    .environment(\.locale, Locale(identifier: "en_US"))
```

This affects the subtree below `ContentView`.

## Environment for Observable Models

Modern SwiftUI can place observable objects in the environment.

```swift
@Observable
final class SessionModel {
    var currentUser: User?
    var isLoggedIn: Bool { currentUser != nil }
}

@main
struct ShopApp: App {
    @State private var session = SessionModel()

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(session)
        }
    }
}
```

Read it:

```swift
struct AccountButton: View {
    @Environment(SessionModel.self) private var session

    var body: some View {
        if session.isLoggedIn {
            Text(session.currentUser?.name ?? "Account")
        } else {
            Text("Sign In")
        }
    }
}
```

## Custom Environment Values

For smaller values, define custom environment values.

```swift
private struct AnalyticsClientKey: EnvironmentKey {
    static let defaultValue: AnalyticsClient = NoopAnalyticsClient()
}

extension EnvironmentValues {
    var analyticsClient: AnalyticsClient {
        get { self[AnalyticsClientKey.self] }
        set { self[AnalyticsClientKey.self] = newValue }
    }
}
```

Use:

```swift
struct ProductDetailsView: View {
    @Environment(\.analyticsClient) private var analytics

    var body: some View {
        Button("Buy") {
            analytics.track("buy_tapped")
        }
    }
}
```

Recent SwiftUI documentation also highlights macro-based environment entry patterns, reducing boilerplate for custom values in newer toolchains.

## Environment vs Dependency Injection

Environment is dependency injection for a view tree, but it should not become invisible global state.

Good environment values:

- Theme
- Locale
- Session
- Analytics
- Feature flags
- Dismiss action
- App-wide services used by many views

Better as explicit parameter:

- A row's product
- A screen's selected filter
- A callback specific to one child
- A domain value needed by only one component

## Real iOS Example: Feature Flag

```swift
struct FeatureFlags {
    var isNewCheckoutEnabled = false
}

private struct FeatureFlagsKey: EnvironmentKey {
    static let defaultValue = FeatureFlags()
}

extension EnvironmentValues {
    var featureFlags: FeatureFlags {
        get { self[FeatureFlagsKey.self] }
        set { self[FeatureFlagsKey.self] = newValue }
    }
}

struct CheckoutButton: View {
    @Environment(\.featureFlags) private var flags

    var body: some View {
        if flags.isNewCheckoutEnabled {
            NewCheckoutButton()
        } else {
            LegacyCheckoutButton()
        }
    }
}
```

This avoids passing feature flags through every intermediate view.

## Environment and Testing

Environment is powerful for previews and tests.

```swift
#Preview("Logged in") {
    AccountScreen()
        .environment(SessionModel.previewLoggedIn)
}

#Preview("Large text") {
    ProductRow(product: .sample)
        .environment(\.dynamicTypeSize, .accessibility3)
}
```

Previews become much more useful when environment dependencies are easy to override.

## Common Mistakes

- Putting everything in environment.
- Hiding important dependencies.
- Using environment for values only one child needs.
- Assuming environment values are global rather than subtree-scoped.
- Mutating shared environment models from many unrelated views without clear ownership.

## Senior iOS Engineer Perspective

A senior engineer uses environment for ambient dependencies and configuration, not as a dumping ground.

Ask:

- Is this dependency truly shared by a subtree?
- Would explicit parameters be clearer?
- Can previews override it easily?
- Is mutation centralized?
- Does this create hidden coupling?
- Is this value user/session/app scoped?

## Interview Notes

Junior:

`@Environment` reads values provided by SwiftUI or by parent views.

Mid-level:

Environment avoids parameter drilling for shared context like locale, color scheme, dismiss, session, and app services.

Senior:

I use environment for ambient, subtree-scoped dependencies and make mutation boundaries explicit. I avoid turning environment into global state and design it so previews, tests, and feature modules can override dependencies cleanly.

## Practice

1. Read `colorScheme` and adapt a view.
2. Add a custom `analyticsClient` environment value.
3. Place an observable `SessionModel` in the environment.
4. Explain when environment is worse than an explicit initializer parameter.
