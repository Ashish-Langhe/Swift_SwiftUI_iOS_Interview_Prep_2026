# Day 18: Main Actor Basics

## What The Main Actor Is

The main actor is a global actor that protects UI-related state.

In iOS apps, UI updates should happen on the main actor.

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var title = ""
}
```

This means the view model's isolated state should be accessed from the main actor.

## Main Actor vs Main Thread

The main actor is closely related to the main thread, but they are not the same concept.

Beginner answer:

- Main thread is an OS thread.
- Main actor is Swift's isolation mechanism for main/UI work.

The main actor gives the compiler a way to reason about UI state safety.

## Why UI Needs Main Actor

UIKit and SwiftUI state updates are UI work.

```swift
@MainActor
func updateTitle(_ title: String) {
    self.title = title
}
```

Without main actor isolation, concurrent tasks could update UI state unpredictably.

## Annotating A Type

```swift
@MainActor
final class LoginViewModel: ObservableObject {
    @Published private(set) var state: State = .idle

    func login() async {
        state = .loading
    }
}
```

All isolated instance members run on the main actor unless marked otherwise.

## Annotating A Method

```swift
final class ImageLoader {
    @MainActor
    func updateImageView(_ imageView: UIImageView, image: UIImage) {
        imageView.image = image
    }
}
```

Use method-level isolation when only one part of a type touches UI.

## Calling Main Actor Code

From async context:

```swift
await MainActor.run {
    self.title = "Loaded"
}
```

This is useful when a non-main service needs to publish a UI result.

## SwiftUI And Main Actor

SwiftUI views and UI-bound observable models are usually main-actor oriented.

```swift
struct ProfileView: View {
    @StateObject private var viewModel = ProfileViewModel()

    var body: some View {
        Text(viewModel.title)
            .task {
                await viewModel.load()
            }
    }
}
```

If `ProfileViewModel` is `@MainActor`, state updates are safe for UI.

## Do Not Do Heavy Work On Main Actor

Bad:

```swift
@MainActor
func loadAndDecode() async throws {
    let data = try await network.data()
    let image = decodeHugeImage(data) // CPU-heavy on main actor.
    self.image = image
}
```

Better:

```swift
func decodedImage(from data: Data) async throws -> UIImage {
    try await decoder.decode(data)
}

@MainActor
func apply(_ image: UIImage) {
    self.image = image
}
```

Keep UI updates on main actor. Move heavy work elsewhere.

## Swift 6.2 Default Actor Isolation

Swift 6.2 introduced configuration that can infer unannotated code as main-actor isolated for selected targets.

In SwiftPM, this can be configured with default isolation settings.

Why this matters:

- UI apps often start on the main actor.
- Beginners need less annotation overhead.
- You still explicitly mark code that should run concurrently.
- Long-running CPU work should not stay on the main actor.

## Common Mistakes

- Equating `@MainActor` with "always on one exact thread"
- Doing image decoding or parsing on the main actor
- Forgetting `await` when crossing into main actor isolation
- Marking everything `@MainActor` to silence warnings
- Updating UI state from detached tasks
- Assuming `private` state is concurrency-safe without actor isolation

## Junior Interview Answer

The main actor is Swift's way to isolate UI work. UI state should be updated on the main actor.

## Mid-Level Interview Answer

I mark UI view models as `@MainActor` so published state changes are safe. If background work needs to update UI, I hop to the main actor with `await MainActor.run`.

## Senior Interview Answer

The main actor is a global actor for UI isolation. I use it to make UI state compiler-checked, but I avoid placing CPU-heavy or blocking work on it. In Swift 6.2, default actor isolation can reduce boilerplate, but good architecture still separates UI state from concurrent services.

## Points To Remember

- Main actor protects UI state.
- Main actor and main thread are related but distinct.
- Use `@MainActor` for UI-bound models.
- Use `MainActor.run` for explicit UI updates.
- Do heavy work off the main actor.

## Practice

1. Mark a SwiftUI view model as `@MainActor`.
2. Move image decoding out of a main-actor method.
3. Explain main actor vs main thread.
4. Use `MainActor.run` from a service callback.
