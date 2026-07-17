# Day 31: SQLite Basics

## What SQLite Is

SQLite is an embedded relational database. It stores structured data in tables inside a local database file.

Use SQLite when you need:

- Structured tables
- Queries
- Filtering/sorting
- Indexes
- Joins
- Transactions
- Large local datasets
- Predictable storage without Core Data/SwiftData abstraction

## Table Example

```sql
CREATE TABLE products (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    price REAL NOT NULL,
    is_favorite INTEGER NOT NULL
);
```

## Insert

```sql
INSERT INTO products (id, name, price, is_favorite)
VALUES (?, ?, ?, ?);
```

Always use parameter binding. Do not build SQL with string interpolation from user input.

## Query

```sql
SELECT id, name, price, is_favorite
FROM products
WHERE name LIKE ?
ORDER BY name ASC;
```

## Index

```sql
CREATE INDEX idx_products_name ON products(name);
```

Indexes speed reads but cost storage and write performance.

## Transaction

```sql
BEGIN TRANSACTION;
INSERT INTO products VALUES (?, ?, ?, ?);
INSERT INTO products VALUES (?, ?, ?, ?);
COMMIT;
```

Use transactions for batches so writes are atomic and faster.

## Swift Access Options

Options:

- Raw SQLite C API
- SQLite.swift
- GRDB
- FMDB legacy codebases
- Core Data/SwiftData if you want object graph abstraction

Raw SQLite is powerful but verbose. Many production apps use a wrapper library.

## Repository Boundary

Hide SQLite behind a repository.

```swift
protocol ProductLocalStore {
    func save(_ products: [Product]) throws
    func products(matching query: String) throws -> [Product]
    func product(id: Product.ID) throws -> Product?
}
```

Your ViewModel should not know SQL.

## Schema Migration

SQLite schema needs versioning.

```sql
PRAGMA user_version;
```

Set:

```sql
PRAGMA user_version = 2;
```

Migration example:

```sql
ALTER TABLE products ADD COLUMN category TEXT;
```

## SQLite vs Core Data/SwiftData

SQLite:

- Direct relational control
- SQL queries
- Manual mapping/migration
- Great for predictable data storage

Core Data/SwiftData:

- Object graph management
- Change tracking
- Relationships
- SwiftUI integration
- Migrations supported by framework

## Common Mistakes

- SQL injection through string interpolation.
- No transactions for batch writes.
- No schema versioning.
- No indexes for common queries.
- Too many indexes.
- Database access on main thread.
- SQL leaking into views/ViewModels.

## Senior Artifact

```text
Artifact: SQLite Persistence Review
Tables:
Primary keys:
Queries:
Indexes:
Transactions:
Migration plan:
Threading:
Repository boundary:
Backup:
Tests:
```

## Interview Notes

Junior:

SQLite is a local relational database.

Mid-level:

Use tables, parameterized queries, indexes, and transactions.

Senior:

I use SQLite when I need direct relational control, predictable queries, and explicit schema migrations. I hide it behind repositories and keep database work off the main thread.

## Practice

1. Design a products table.
2. Add an index for search.
3. Explain parameter binding.
4. Write a migration adding a column.
