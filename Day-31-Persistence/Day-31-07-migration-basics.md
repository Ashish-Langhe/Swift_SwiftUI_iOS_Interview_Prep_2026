# Day 31: Migration Basics

## What Migration Means

Migration means changing persisted data from an old schema or format to a new one.

Persistence formats change when you:

- Add fields
- Rename fields
- Remove fields
- Split models
- Merge models
- Change relationships
- Change enum cases
- Change database schema

If users already have data, you need a migration strategy.

## UserDefaults Migration

Renaming a key:

```swift
func migrateDefaults(_ defaults: UserDefaults) {
    if defaults.object(forKey: "theme") != nil,
       defaults.object(forKey: "selectedTheme") == nil {
        defaults.set(defaults.string(forKey: "theme"), forKey: "selectedTheme")
        defaults.removeObject(forKey: "theme")
    }
}
```

## Codable Migration

Use schema versioning.

```swift
struct PersistedState: Codable {
    var schemaVersion: Int
    var selectedTheme: String
    var notificationsEnabled: Bool
}
```

Decode old formats with default values.

```swift
notificationsEnabled = try container.decodeIfPresent(Bool.self, forKey: .notificationsEnabled) ?? true
```

## SQLite Migration

Use `PRAGMA user_version`.

```swift
func migrate(database: SQLiteDatabase) throws {
    let version = try database.userVersion()

    if version < 2 {
        try database.execute("ALTER TABLE products ADD COLUMN category TEXT")
        try database.setUserVersion(2)
    }

    if version < 3 {
        try database.execute("CREATE INDEX idx_products_category ON products(category)")
        try database.setUserVersion(3)
    }
}
```

Important: run migrations in order.

## Core Data Migration

Core Data stores are tied to their model version. When the model changes, existing stores may require migration.

Core Data supports:

- Lightweight migration for simple changes.
- Mapping models for custom migrations.
- Versioned model files.
- Migration manager flows for complex changes.

Examples of lightweight changes:

- Add optional attribute.
- Rename with renaming identifier.
- Add relationship in compatible way.

Complex changes may need custom mapping.

## SwiftData Migration

SwiftData has schema/migration capabilities and continues to evolve. For production apps, treat SwiftData schema changes with the same seriousness as Core Data:

- Plan versions.
- Test upgrade from old app data.
- Avoid destructive changes casually.
- Keep sample old stores for tests.
- Validate relationship changes.

## Migration Testing

Senior teams test migration with real old data.

```text
Migration Test Plan:
Install old app version
Create realistic data
Upgrade to new version
Open app
Validate data integrity
Validate queries
Validate sync
Validate rollback risk
```

Automated tests can include fixture stores/files from previous versions.

## Backward Compatibility

Ask:

- Can old app read new data?
- Can new app read old data?
- Is downgrade supported?
- Is cloud sync involved?
- Is migration destructive?

Many apps only support forward migration, but the decision should be explicit.

## Common Mistakes

- Changing persisted model without migration.
- Testing only fresh installs.
- Force-unwrapping new fields.
- Deleting old data silently.
- No version number.
- Migration runs on main thread with large data.
- No backup/rollback consideration.

## Senior Artifact

```text
Artifact: Migration Plan
Current version:
Target version:
Storage type:
Schema changes:
Migration steps:
Data loss risk:
Backup strategy:
Test fixtures:
Performance risk:
Rollback/downgrade:
Release checklist:
```

## Interview Notes

Junior:

Migration updates stored data when the app's data model changes.

Mid-level:

Use versioning and migrate old keys/files/database schemas step by step.

Senior:

I treat migrations as release-critical. I test upgrades from realistic old data, plan schema versions, avoid silent data loss, and consider performance, cloud sync, and rollback behavior.

## Practice

1. Migrate a UserDefaults key.
2. Add a Codable schema version.
3. Add a SQLite column migration.
4. Explain lightweight vs custom Core Data migration.
