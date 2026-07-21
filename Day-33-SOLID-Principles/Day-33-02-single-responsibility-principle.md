# Day 33: Single Responsibility Principle

## What SRP Means

Single Responsibility Principle says:

```text
A type should have one reason to change.
```

This does not mean one method per class. It means one coherent responsibility.

In iOS, SRP asks:

- Is this ViewModel doing presentation and networking?
- Is this View doing business validation?
- Is this service doing persistence and analytics?
- Is this repository doing UI formatting?
- Is this coordinator doing business decisions?

## Bad Example: Massive ViewModel

```swift
@MainActor
final class ProfileViewModel {
    var name = ""
    var bio = ""
    var image: UIImage?
    var errorMessage: String?

    func loadProfile() async {
        let url = URL(string: "https://api.example.com/profile")!
        let data = try! await URLSession.shared.data(from: url).0
        let response = try! JSONDecoder().decode(ProfileResponse.self, from: data)
        name = response.displayName
        bio = response.bio ?? ""
    }

    func saveProfile() async {
        UserDefaults.standard.set(name, forKey: "profileName")
        AnalyticsManager.shared.track("profile_saved")
    }

    func resizeImage() {
        image = image?.resized(to: CGSize(width: 200, height: 200))
    }
}
```

This ViewModel changes when:

- API changes
- JSON mapping changes
- UI state changes
- Persistence changes
- Analytics changes
- Image processing changes

Too many reasons.

## Refactored Design

Models:

```swift
struct Profile: Equatable {
    var displayName: String
    var bio: String
    var imageURL: URL?
}

struct ProfileDraft: Equatable {
    var displayName: String
    var bio: String
}
```

Repository:

```swift
protocol ProfileRepository {
    func profile() async throws -> Profile
    func save(_ draft: ProfileDraft) async throws -> Profile
}
```

Image processor:

```swift
protocol ImageProcessing {
    func thumbnail(from image: UIImage) async throws -> UIImage
}
```

Analytics:

```swift
protocol AnalyticsClient {
    func track(_ event: String)
}
```

ViewModel:

```swift
@Observable
@MainActor
final class ProfileViewModel {
    var state: LoadState<ProfileDraft> = .idle

    private let repository: ProfileRepository
    private let analytics: AnalyticsClient

    init(repository: ProfileRepository, analytics: AnalyticsClient) {
        self.repository = repository
        self.analytics = analytics
    }

    func load() async {
        state = .loading

        do {
            let profile = try await repository.profile()
            state = .loaded(ProfileDraft(
                displayName: profile.displayName,
                bio: profile.bio
            ))
        } catch {
            state = .failed("Unable to load profile.")
        }
    }

    func save(_ draft: ProfileDraft) async {
        do {
            _ = try await repository.save(draft)
            analytics.track("profile_saved")
        } catch {
            state = .failed("Unable to save profile.")
        }
    }
}
```

Now each type has a clearer reason to change.

## SRP In SwiftUI Views

Bad:

```swift
struct CheckoutView: View {
    @State private var cardNumber = ""
    @State private var isSubmitting = false

    var body: some View {
        Form {
            TextField("Card", text: $cardNumber)

            Button("Pay") {
                if cardNumber.count == 16 {
                    Task {
                        isSubmitting = true
                        try await URLSession.shared.pay(cardNumber)
                        UserDefaults.standard.set(true, forKey: "hasPaid")
                    }
                }
            }
        }
    }
}
```

Better:

```swift
struct CheckoutView: View {
    @Bindable var viewModel: CheckoutViewModel

    var body: some View {
        Form {
            TextField("Card", text: $viewModel.cardNumber)

            Button("Pay") {
                Task { await viewModel.pay() }
            }
            .disabled(!viewModel.canPay)
        }
    }
}
```

The view renders and forwards intent. The ViewModel coordinates presentation behavior.

## How SRP Aligns With Reusability

Focused types are easier to reuse.

Example:

```swift
struct EmailValidator {
    func isValid(_ email: String) -> Bool {
        email.contains("@") && email.contains(".")
    }
}
```

This can be reused in:

- Login
- Sign up
- Profile edit
- Invite user
- Account recovery

If validation is embedded inside `LoginView`, it cannot be reused cleanly.

## When To Apply SRP

Apply when:

- Type has many unrelated dependencies.
- Tests require too much setup.
- One change breaks unrelated behavior.
- File is hard to review.
- ViewModel/service is growing in every direction.
- Logic should be reused elsewhere.

Do not over-apply when:

- Splitting makes code harder to follow.
- Responsibility is already cohesive.
- You are abstracting before change pressure exists.

## Senior Refactoring Technique

Use "reason to change" grouping:

```text
Networking change -> API service/repository
Business rule change -> domain/use case/policy
UI display change -> View/ViewModel/display model
Navigation change -> Coordinator/Router
Persistence change -> Store/Repository
Analytics change -> Analytics client/use case
```

## Common Mistakes

- Thinking SRP means a class can have only one method.
- Splitting every helper into a new type.
- Leaving business logic in SwiftUI views.
- ViewModels that own networking, persistence, and navigation.
- Repositories formatting UI strings.
- Coordinators making domain decisions.

## Senior iOS Engineer Artifact

```text
Artifact: SRP Refactor Plan
Type:
Current responsibilities:
Reasons to change:
Dependencies:
Responsibilities to extract:
New types:
Tests unlocked:
Tradeoff:
```

## Interview Notes

Junior:

SRP means one type should have one clear job.

Mid-level:

It reduces coupling by separating networking, UI state, validation, persistence, and navigation.

Senior:

I use SRP to identify reasons for change. I extract responsibilities when they improve testability, reuse, or reviewability, but I avoid splitting cohesive logic into unnecessary ceremony.

## Practice

1. Find five responsibilities in a massive ViewModel.
2. Extract validation into a policy/helper.
3. Move persistence out of a view.
4. Explain when SRP refactoring is overkill.
