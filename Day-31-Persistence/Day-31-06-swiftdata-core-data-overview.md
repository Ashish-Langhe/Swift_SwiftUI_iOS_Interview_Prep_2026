# Day 31: SwiftData And Core Data Overview

## What SwiftData Is

SwiftData is Apple's modern Swift-native persistence framework. It uses macros and Swift language features to define persistent models.

```swift
import SwiftData

@Model
final class Trip {
    var name: String
    var startDate: Date
    var endDate: Date

    init(name: String, startDate: Date, endDate: Date) {
        self.name = name
        self.startDate = startDate
        self.endDate = endDate
    }
}
```

SwiftUI usage:

```swift
@Query(sort: \Trip.startDate) private var trips: [Trip]
@Environment(\.modelContext) private var modelContext
```

Insert:

```swift
let trip = Trip(name: "Pune", startDate: .now, endDate: .now)
modelContext.insert(trip)
try modelContext.save()
```

## What Core Data Is

Core Data is Apple's mature object graph and persistence framework.

It provides:

- Object graph management
- Persistence
- Relationships
- Change tracking
- Fetch requests
- Undo
- Batch operations
- CloudKit integration
- Mature migration support

Core Data is not just SQLite. It can use SQLite as a persistent store, but it is an object graph framework.

## SwiftData vs Core Data

SwiftData:

- Swift-first
- Macro-based models
- Tight SwiftUI integration
- Less boilerplate
- Good for new SwiftUI apps

Core Data:

- Mature and powerful
- More configuration/control
- Existing enterprise adoption
- Strong migration history
- Better known for complex legacy apps

## Latest SwiftData Notes

Recent SwiftData updates include:

- Query sectioning with query macros.
- Codable model attribute support through `Schema.Attribute` options.
- `ResultsObserver` for real-time model updates matching fetch criteria.
- `HistoryObserver` for remote model changes.
- Previous updates added model inheritance flexibility and transaction history sorting improvements.

Senior takeaway: SwiftData continues to mature, but you still need to design schema, migration, relationships, and concurrency carefully.

## Relationships

```swift
@Model
final class Folder {
    var name: String
    var notes: [Note] = []

    init(name: String) {
        self.name = name
    }
}

@Model
final class Note {
    var title: String
    var body: String

    init(title: String, body: String) {
        self.title = title
        self.body = body
    }
}
```

Relationships make object persistence easier than plain Codable files.

## Querying

```swift
@Query(
    filter: #Predicate<Trip> { $0.startDate > Date.now },
    sort: \Trip.startDate
) private var upcomingTrips: [Trip]
```

For complex data access, consider repository/store abstractions instead of putting all queries directly in views.

## Repository Boundary With SwiftData

```swift
protocol NotesStore {
    func notes() throws -> [Note]
    func addNote(title: String, body: String) throws
}
```

This helps tests and keeps UI from owning persistence details.

## When To Use SwiftData/Core Data

Use when:

- App has object graph data.
- Relationships matter.
- Local persistence is core to app.
- Querying/filtering/sorting matters.
- SwiftUI integration is useful.
- CloudKit sync may matter.

Maybe not needed when:

- You store one preference.
- You store small draft JSON.
- Data is only a simple cache.
- You need direct SQL control.

## Common Mistakes

- Treating SwiftData as magic.
- No migration plan.
- Query logic scattered through views.
- Large work on main context.
- Confusing domain models and persistence models.
- Poor relationship/delete-rule design.
- No test strategy.

## Senior Artifact

```text
Artifact: SwiftData/Core Data Review
Entities/models:
Relationships:
Delete rules:
Queries:
Concurrency:
Migration:
Cloud sync:
Repository boundary:
Tests:
```

## Interview Notes

Junior:

SwiftData and Core Data persist model data locally.

Mid-level:

SwiftData is Swift-first and integrates with SwiftUI; Core Data is mature and powerful.

Senior:

I choose SwiftData/Core Data when object graph persistence, relationships, queries, migrations, or sync matter. I still design schema, concurrency, migration, and repository boundaries deliberately.

## Practice

1. Create a SwiftData `Note` model.
2. Add a relationship between `Folder` and `Note`.
3. Query upcoming trips.
4. Compare SwiftData and Core Data for a large legacy app.
