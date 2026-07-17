# Day 31: Codable Persistence

## What Codable Persistence Means

Codable persistence means encoding Swift models to data and saving them, often as JSON or property list files.

Good for:

- Small local settings objects
- Draft forms
- Offline cache snapshots
- Simple app data
- Export/import files

Not ideal for:

- Large relational data
- Complex queries
- Frequent partial updates
- Multi-object relationships
- Conflict resolution

## Basic Codable Model

```swift
struct DraftProfile: Codable, Equatable {
    var displayName: String
    var bio: String
    var isPublic: Bool
}
```

Save:

```swift
struct CodableFileStore<Value: Codable> {
    let url: URL
    let encoder = JSONEncoder()
    let decoder = JSONDecoder()

    func save(_ value: Value) throws {
        let data = try encoder.encode(value)
        try data.write(to: url, options: [.atomic])
    }

    func load() throws -> Value {
        let data = try Data(contentsOf: url)
        return try decoder.decode(Value.self, from: data)
    }
}
```

## Real iOS Example: Draft Form

```swift
struct CheckoutDraft: Codable, Equatable {
    var name = ""
    var street = ""
    var city = ""
    var postalCode = ""
}

@Observable
@MainActor
final class CheckoutDraftModel {
    var draft = CheckoutDraft()

    private let store: CodableFileStore<CheckoutDraft>

    init(store: CodableFileStore<CheckoutDraft>) {
        self.store = store
    }

    func loadDraft() {
        if let saved = try? store.load() {
            draft = saved
        }
    }

    func saveDraft() {
        try? store.save(draft)
    }
}
```

## JSONEncoder Configuration

```swift
let encoder = JSONEncoder()
encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
encoder.dateEncodingStrategy = .iso8601

let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
```

Choose date/key strategies deliberately.

## Versioning

Codable persistence needs version planning.

```swift
struct PersistedSettings: Codable {
    var schemaVersion: Int
    var selectedTheme: String
    var notificationsEnabled: Bool
}
```

Migration:

```swift
func migrate(_ settings: PersistedSettings) -> PersistedSettings {
    switch settings.schemaVersion {
    case 1:
        return PersistedSettings(
            schemaVersion: 2,
            selectedTheme: settings.selectedTheme,
            notificationsEnabled: true
        )
    default:
        return settings
    }
}
```

## Backward-Compatible Decoding

Use defaults for new fields.

```swift
struct Settings: Codable {
    var selectedTheme: String
    var notificationsEnabled: Bool

    enum CodingKeys: String, CodingKey {
        case selectedTheme
        case notificationsEnabled
    }

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        selectedTheme = try container.decode(String.self, forKey: .selectedTheme)
        notificationsEnabled = try container.decodeIfPresent(Bool.self, forKey: .notificationsEnabled) ?? true
    }
}
```

## Atomicity And Corruption

Use atomic writes and consider backup files for important data.

```swift
try data.write(to: url, options: [.atomic])
```

For critical data, save previous known-good version.

## Security

Codable does not encrypt. If encoded data contains secrets, store it in Keychain or encrypt deliberately before saving.

## Common Mistakes

- Using Codable files as a database.
- No migration/versioning.
- Force-try load on app launch.
- Storing secrets in JSON.
- No atomic writes.
- No default handling for new fields.
- Loading huge JSON files on main thread.

## Senior Artifact

```text
Artifact: Codable Persistence Review
Data:
File location:
Size:
Encoding format:
Date/key strategies:
Version:
Migration:
Security:
Atomicity:
Tests:
```

## Interview Notes

Junior:

`Codable` can encode models to JSON and decode them later.

Mid-level:

Use it for simple persisted snapshots and configure date/key strategies.

Senior:

I use Codable persistence for small document-like data, with versioning, migration, atomic writes, corruption handling, and no sensitive plaintext.

## Practice

1. Save a draft profile as JSON.
2. Add a schema version.
3. Decode a missing field with a default.
4. Explain when Codable persistence is worse than SQLite/Core Data.
