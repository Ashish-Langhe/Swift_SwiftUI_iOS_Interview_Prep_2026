# Day 31: Persistence Decision Guide And Security

## The Core Question

Before choosing persistence, ask:

```text
What kind of data is this?
```

The answer determines storage.

## Decision Table

```text
Theme preference:
UserDefaults / AppStorage

Auth token:
Keychain

Draft form:
Codable file or SwiftData, depending complexity

Downloaded image cache:
FileManager caches directory

Large relational local dataset:
SQLite / Core Data / SwiftData

Object graph with relationships:
SwiftData / Core Data

Small exportable document:
Codable file

Sensitive note:
Encrypted file or Keychain if small
```

## Security Classification

Classify data:

- Public
- Internal app data
- User personal data
- Secret
- Regulated/sensitive

Secrets need Keychain or encryption. Personal data needs privacy review and careful backup/sync decisions.

## Backup And Sync

Ask:

- Should this survive reinstall?
- Should it sync across devices?
- Should it be backed up?
- Can it be recreated?
- Is it too large for backup?

Examples:

- User-created document: back up.
- Image cache: do not back up.
- Token: Keychain behavior depends on accessibility/sync choices.
- Local database: depends on app data model.

## Offline Strategy

Persistence often supports offline behavior.

Patterns:

- Cache-first
- Network-first with cache fallback
- Offline write queue
- Snapshot cache
- Full local source of truth

Repository example:

```swift
func products() async throws -> [Product] {
    do {
        let fresh = try await api.products()
        await cache.save(fresh)
        return fresh
    } catch {
        if let cached = await cache.products() {
            return cached
        }
        throw error
    }
}
```

## Privacy

Persistence decisions affect privacy:

- Minimize stored data.
- Delete on logout when appropriate.
- Do not store secrets in logs.
- Encrypt sensitive files.
- Respect user deletion requests.
- Be clear about sync/backup.

## Testing Persistence

Test:

- Fresh install
- Upgrade migration
- Corrupt file
- Missing Keychain item
- Offline cache fallback
- Logout cleanup
- Large dataset performance
- Background/locked-device access if relevant

## Senior Artifact

```text
Artifact: Persistence Decision Review
Data:
Sensitivity:
Size:
Shape:
Query needs:
Lifetime:
Backup:
Sync:
Offline:
Migration:
Storage choice:
Why:
Risks:
Tests:
```

## Common Mistakes

- Choosing storage by habit.
- UserDefaults for everything.
- Keychain for large data.
- No migration plan.
- No logout cleanup.
- No offline/cache strategy.
- Ignoring backup and privacy.
- Persistence code inside views.

## Interview Notes

Junior:

Different data needs different storage.

Mid-level:

Use UserDefaults for preferences, Keychain for secrets, files for documents/caches, and databases for structured data.

Senior:

I choose persistence by sensitivity, size, shape, query needs, lifetime, backup/sync behavior, migration risk, and testability.

## Practice

1. Choose storage for an auth token.
2. Choose storage for a product cache.
3. Design logout cleanup.
4. Write a persistence decision review for checkout drafts.
