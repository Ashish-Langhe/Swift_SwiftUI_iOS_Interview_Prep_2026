# Day 19: Global Actors

## What Global Actors Are

A global actor provides one shared actor isolation domain.

The most common global actor is `MainActor`.

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
}
```

All isolated members are protected by the main actor.

## Why Global Actors Exist

Some state belongs to one global execution context.

Examples:

- UI state on `MainActor`
- Persistence coordinator
- Analytics pipeline
- App-wide session state

## Custom Global Actor

```swift
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
}

@DatabaseActor
final class DatabaseService {
    func save(_ user: User) {
        // Serialized through DatabaseActor.
    }
}
```

Now all `@DatabaseActor` code shares one isolation domain.

## Main Actor

```swift
@MainActor
func updateUI() {
    title = "Loaded"
}
```

Use for:

- UIKit updates
- SwiftUI observable state
- UI navigation
- View model state consumed by UI

## Global Actor On Protocols

```swift
@MainActor
protocol ScreenViewModel: ObservableObject {
    func load() async
}
```

Conforming types are expected to satisfy main-actor requirements.

This is powerful but becomes part of the API contract.

## Crossing Isolation

```swift
func backgroundWork() async {
    let value = await service.load()

    await MainActor.run {
        self.title = value.title
    }
}
```

Crossing into a global actor from outside usually requires `await`.

## Real iOS Use Case

```swift
@MainActor
final class SearchViewModel: ObservableObject {
    @Published private(set) var results: [SearchResult] = []

    private let service: SearchService

    func search(_ query: String) async {
        do {
            let results = try await service.search(query)
            self.results = results
        } catch {
            self.results = []
        }
    }
}
```

This keeps UI state serialized.

## Common Mistakes

- Marking heavy services `@MainActor`
- Adding global actor isolation to public protocols casually
- Forgetting that global actor annotations affect callers
- Using custom global actors when a normal actor instance would work
- Treating actor isolation as security

## Modern Swift 6.x Notes

Swift 6.2 default actor isolation can infer main-actor isolation in selected targets, reducing boilerplate for UI-heavy code.

But explicit global actors are still important for:

- Public APIs
- Framework boundaries
- Non-main global resources
- Documentation of isolation behavior

## Junior Interview Answer

A global actor is a shared actor used to isolate code across many types or functions. `MainActor` is the main example for UI work.

## Mid-Level Interview Answer

I mark UI view models and UI-updating methods `@MainActor`. I use custom global actors only when many independent types must share one serialized execution domain.

## Senior Interview Answer

Global actors are API-level isolation tools. They are useful for UI and shared global resources, but they should be applied carefully because isolation annotations affect callers, protocol conformances, testing, and public API evolution.

## Practice

1. Mark a view model as `@MainActor`.
2. Create a custom `DatabaseActor`.
3. Explain when a normal actor is better than a global actor.
4. Identify a public API where `@MainActor` would affect callers.
