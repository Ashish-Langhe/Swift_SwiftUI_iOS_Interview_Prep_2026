# Day 28: Model Layer, Domain, DTO, And Repository Boundaries

## Why The Model Layer Matters

In weak MVVM, "Model" often means any struct with properties. In strong MVVM, the model layer is where business meaning lives.

The model layer can include:

- Domain models
- API DTOs
- Persistence entities
- Repositories
- Services
- Business rules
- Mappers

A senior engineer separates these when their responsibilities differ.

## Domain Model

Domain models represent app meaning.

```swift
struct UserProfile: Equatable, Identifiable {
    let id: UserID
    var displayName: String
    var bio: String
    var isPublic: Bool

    var canBePublished: Bool {
        !displayName.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
    }
}
```

This model has stronger meaning than raw JSON.

## DTO

DTO means Data Transfer Object. It represents network payload shape.

```swift
struct UserProfileResponse: Decodable {
    let id: String
    let display_name: String
    let bio: String?
    let visibility: String
}
```

DTOs often contain:

- Optional fields
- Backend naming
- Weak typing
- Backward compatibility quirks

Do not blindly use DTOs as domain models.

## Mapping DTO To Domain

```swift
extension UserProfile {
    init(response: UserProfileResponse) throws {
        guard let id = UserID(rawValue: response.id) else {
            throw MappingError.invalidUserID
        }

        self.id = id
        self.displayName = response.display_name
        self.bio = response.bio ?? ""
        self.isPublic = response.visibility == "public"
    }
}
```

Mapping is where weak external data becomes stronger app data.

## Persistence Entity

Database shape may differ too.

```swift
struct UserProfileRecord {
    let id: String
    let displayName: String
    let bio: String
    let isPublic: Bool
    let lastSyncedAt: Date
}
```

Persistence models may include local-only metadata like sync dates.

## Repository

A repository hides where data comes from.

```swift
protocol UserProfileRepository {
    func profile(userID: UserID) async throws -> UserProfile
    func save(_ profile: UserProfile) async throws
}
```

Implementation can combine:

- Network
- Cache
- Database
- Sync rules

```swift
final class LiveUserProfileRepository: UserProfileRepository {
    private let api: UserProfileAPI
    private let cache: UserProfileCache

    init(api: UserProfileAPI, cache: UserProfileCache) {
        self.api = api
        self.cache = cache
    }

    func profile(userID: UserID) async throws -> UserProfile {
        if let cached = await cache.profile(userID: userID) {
            return cached
        }

        let response = try await api.profile(userID: userID.rawValue)
        let profile = try UserProfile(response: response)
        await cache.save(profile)
        return profile
    }

    func save(_ profile: UserProfile) async throws {
        try await api.save(profile)
        await cache.save(profile)
    }
}
```

## ViewModel Uses Repository

```swift
@Observable
@MainActor
final class ProfileViewModel {
    var state: State = .idle

    private let userID: UserID
    private let repository: UserProfileRepository

    init(userID: UserID, repository: UserProfileRepository) {
        self.userID = userID
        self.repository = repository
    }

    func load() async {
        state = .loading

        do {
            state = .loaded(try await repository.profile(userID: userID))
        } catch {
            state = .failed("Unable to load profile.")
        }
    }
}
```

The ViewModel does not know if data came from cache, network, or disk.

## When To Separate DTO And Domain

Separate when:

- API shape differs from app needs.
- Backend data is optional/weakly typed.
- Domain has invariants.
- You need offline/cache entities.
- You need stable app behavior across API changes.
- You test business logic.

You can keep one model when:

- App is tiny.
- API exactly matches UI.
- There is no business logic.
- The cost of mapping is not justified.

Senior decision: separation is a tradeoff, not a religion.

## Common Mistakes

- Using API DTOs directly in SwiftUI views.
- Putting URLSession code inside ViewModels.
- Mapping network errors directly to UI strings everywhere.
- No domain validation.
- Repositories that become giant service buckets.
- Duplicate models with no real reason.

## Senior iOS Engineer Artifact

```text
Artifact: Model Boundary Review
Feature:
API DTO:
Domain model:
Persistence model:
Mapping rules:
Invariants:
Repository:
Cache strategy:
Error mapping:
Test doubles:
Tradeoff:
```

## Interview Notes

Junior:

Models represent data used by the app.

Mid-level:

DTOs represent API shape; domain models represent app meaning. Repositories hide data sources.

Senior:

I separate DTO, domain, and persistence models when they have different responsibilities or invariants. ViewModels depend on repositories so presentation logic is not coupled to network or storage implementation.

## Practice

1. Create a DTO and domain model for a profile API.
2. Add a mapper that validates invalid IDs.
3. Define a repository protocol.
4. Explain when model separation is overengineering.
