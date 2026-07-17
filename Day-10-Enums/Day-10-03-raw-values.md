# Day 10: Raw Values

## Core Idea

Raw values give enum cases fixed values.

```swift
enum APIPath: String {
    case users = "/users"
    case products = "/products"
}
```

## Initialization

```swift
let path = APIPath(rawValue: "/users")
```

This returns optional because the raw value may not match.

## Interview Levels

Junior:

Raw values assign fixed values to enum cases.

Senior:

Raw values are useful for persistence, API mapping, and simple interop. Associated values are better when each case needs different dynamic data.

## Quick Notes

- Fixed value per case
- Common raw types: String, Int
- `init(rawValue:)` returns optional
- Good for API constants

## Interview Depth

Junior answer: Raw values give enum cases fixed values like strings or integers.

Mid-level answer: Raw values are useful for persistence, API constants, analytics names, and mapping external values into Swift enums.

Senior answer: Raw values are stable representations. Use them when cases map to external fixed values. Use associated values when data is dynamic and case-specific.

iOS use case:

```swift
enum AnalyticsEvent: String {
    case loginTapped = "login_tapped"
    case purchaseCompleted = "purchase_completed"
}
```

Common mistakes:

- Using raw values for dynamic data.
- Force-unwrapping `init(rawValue:)`.
- Changing persisted raw values without migration.

Practice:

1. Create raw string enum for API path.
2. Safely initialize from raw value.
3. Explain raw vs associated values.
