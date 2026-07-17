# Day 4: Nil-Coalescing

## Core Idea

The nil-coalescing operator `??` provides a fallback value when an optional is nil.

```swift
let displayName: String? = nil
let title = displayName ?? "Guest"
```

If `displayName` has a value, `title` uses it. Otherwise, it uses `"Guest"`.

## Syntax

```swift
optionalValue ?? fallbackValue
```

The fallback must match the wrapped type.

```swift
let count: Int? = nil
let visibleCount = count ?? 0
```

## With Optional Chaining

```swift
let city = user.address?.city ?? "Unknown city"
```

This is common in UI display code.

## Lazy Evaluation

The fallback is evaluated only if the optional is nil.

```swift
func expensiveFallback() -> String {
    print("Calculating fallback")
    return "Fallback"
}

let value: String? = "Actual"
let result = value ?? expensiveFallback()
```

`expensiveFallback()` is not called because `value` exists.

## Good Use Cases

### Display Fallback

```swift
let subtitle = product.subtitle ?? "No description"
```

### Default Count

```swift
let unreadCount = response.unreadCount ?? 0
```

### Optional String Cleanup

```swift
let query = searchText?.trimmingCharacters(in: .whitespaces) ?? ""
```

## Risk: Hiding Important Missing Data

Nil-coalescing is not always the right answer.

```swift
let userId = response.userId ?? ""
```

This may hide a broken API response.

Better:

```swift
guard let userId = response.userId else {
    throw APIError.missingUserId
}
```

Use fallback values only when fallback is valid in the domain.

## Real iOS Use Cases

### SwiftUI Text

```swift
Text(user.nickname ?? user.fullName)
```

### API Defaults

```swift
let isVerified = response.isVerified ?? false
```

Only do this if missing verification truly means false.

### Settings

```swift
let selectedTheme = storedTheme ?? "system"
```

## Junior-Level Interview Answer

`??` gives a default value when an optional is nil.

## Mid-Level Interview Answer

Nil-coalescing is useful for display fallbacks and harmless defaults. It avoids verbose `if let` code when the only decision is a fallback.

## Senior-Level Interview Answer

Nil-coalescing should reflect domain truth. It is convenient, but it can hide data quality issues if used for required fields. For user-facing display, a fallback is often appropriate. For IDs, tokens, permissions, and critical API fields, I prefer validation and explicit failure.

## Quick Interview Notes

- `??` provides a fallback for nil.
- The fallback is lazily evaluated.
- The fallback type must match.
- Good for display defaults.
- Dangerous when it hides missing required data.

## Practice Questions

1. What does `??` do?
2. Is the fallback always evaluated?
3. When is nil-coalescing useful?
4. When can nil-coalescing be harmful?
5. How would you handle a missing required user ID?

