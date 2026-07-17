# Day 10: Identifiable Enums

## Core Idea

Enums can conform to `Identifiable`.

```swift
enum Tab: String, CaseIterable, Identifiable {
    case home
    case settings

    var id: String { rawValue }
}
```

## SwiftUI Use Case

```swift
ForEach(Tab.allCases) { tab in
    Text(tab.rawValue)
}
```

## Interview Levels

Junior:

`Identifiable` gives each value an ID.

Senior:

Identifiable enums work well for SwiftUI lists, tabs, sheets, and stable UI identity.

## Quick Notes

- Provide `id`
- Useful in SwiftUI
- Raw value often works as ID

## Interview Depth

Junior answer: `Identifiable` means a value has a stable `id`.

Mid-level answer: SwiftUI uses identity to diff and render lists, sheets, and repeated views.

Senior answer: Enum identity should be stable and unique for the UI purpose. Raw values work for simple cases; associated-value cases often need custom IDs.

iOS use case:

```swift
enum ActiveSheet: Identifiable {
    case profile(userId: String)
    case settings

    var id: String {
        switch self {
        case .profile(let userId): return "profile-\(userId)"
        case .settings: return "settings"
        }
    }
}
```

Common mistakes:

- Returning non-unique IDs.
- Using display text as identity.
- Changing IDs between renders.

Practice:

1. Make enum `Identifiable`.
2. Use raw value as ID.
3. Build custom ID for associated value.
