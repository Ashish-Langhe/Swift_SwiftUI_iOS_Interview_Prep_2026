# Day 31: Persistence Interview Guide

## One-Minute Senior Answer

iOS persistence is not one tool. I use UserDefaults for small nonsensitive preferences, Keychain for secrets, FileManager for files and caches, Codable for simple document-like snapshots, SQLite for relational control, and SwiftData/Core Data for object graph persistence with relationships, queries, and migrations. Senior persistence design considers sensitivity, data size, query shape, lifetime, backup, sync, migration, concurrency, privacy, and testability.

## Core Mental Models

- UserDefaults: preferences, not secrets.
- Keychain: small secrets.
- FileManager: files, documents, caches, temp data.
- Codable: simple encoded snapshots/documents.
- SQLite: relational database with SQL control.
- SwiftData/Core Data: object graph persistence.
- Migration: release-critical data evolution.
- Repository/store: boundary hiding persistence details.

## Junior Questions

What is UserDefaults for?

Small app settings and preferences.

What is Keychain for?

Secure storage of small sensitive values like tokens or passwords.

What is FileManager for?

Reading and writing files in the app sandbox.

What is Codable persistence?

Encoding models to data such as JSON and saving them.

## Mid-Level Questions

When use SQLite?

When you need relational tables, queries, indexes, transactions, and direct control.

SwiftData vs Core Data?

SwiftData is Swift-first and integrates well with SwiftUI. Core Data is older, mature, powerful, and common in large legacy apps.

Why use repository around persistence?

To keep ViewModels/UI from knowing storage details and make tests easier.

## Senior Questions

How do you choose storage?

I classify data by sensitivity, size, shape, query needs, lifetime, backup/sync behavior, migration risk, and testability.

How do you handle migrations?

I version schemas/keys/files, migrate step by step, test with realistic old data, and avoid silent data loss.

How do you store auth tokens?

In Keychain with an accessibility level chosen for security and app behavior. I wrap Keychain behind a protocol and clean up on logout.

How do you support offline data?

Usually through a repository that coordinates network and local storage with a defined cache strategy, such as cache-first or network-first fallback.

## Senior Artifact: Persistence Architecture Review

```text
Feature:
Data types:
Sensitivity:
Storage choices:
Repositories/stores:
Migration plan:
Backup/sync:
Offline behavior:
Concurrency:
Logout cleanup:
Tests:
Risks:
```

## Real Scenario: Shopping App

Storage choices:

- Theme: UserDefaults.
- Refresh token: Keychain.
- Product image cache: FileManager Caches.
- Checkout draft: Codable file or SwiftData.
- Product catalog cache: SQLite/SwiftData/Core Data depending query complexity.
- Orders: SwiftData/Core Data if relationships and offline access matter.

Senior discussion:

- Product cache can be recreated, so backup may be unnecessary.
- Token must be secure and cleaned on logout.
- Checkout draft migration matters if form fields change.
- Orders may need schema migration and sync strategy.

## Common Interview Traps

- Storing tokens in UserDefaults.
- Calling Core Data "just SQLite."
- No migration answer.
- Ignoring backup and privacy.
- Saying Keychain is for large encrypted data.
- Putting persistence code directly in SwiftUI views.
- No testing strategy for corrupted/missing data.

## Strong Closing Answer

Persistence is product risk. Bad storage choices can leak secrets, lose user data, break upgrades, or make offline behavior unreliable. I design persistence behind clear boundaries, choose storage based on data characteristics, and test migration and failure paths.

## Practice Prompts

1. Choose storage for every data type in a notes app.
2. Design a Keychain token store.
3. Design a Codable draft migration.
4. Compare SQLite and SwiftData for product catalog cache.
5. Write a migration test plan.
6. Explain logout cleanup.
