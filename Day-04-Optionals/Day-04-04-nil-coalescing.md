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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Nil-Coalescing is not only a syntax topic. In production Swift, it affects absence modeling, safe unwrapping, nil propagation, and avoiding crash-prone assumptions. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while decoding API payloads, optional user profile fields, and view state that may not exist yet. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Nil-Coalescing in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Ask whether nil is a valid business state, an incomplete loading state, or a data-quality bug.
- Prefer explicit state enums when multiple optional values must stay consistent.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Nil-Coalescing correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Nil-Coalescing, but it shows the kind of production shape you should connect this topic to:

```swift
struct ProfileResponse: Decodable {
    let id: String
    let displayName: String?
    let avatarURL: URL?
}

struct ProfileViewState {
    let title: String
    let avatarURL: URL?

    init(response: ProfileResponse) {
        let trimmedName = response.displayName?.trimmingCharacters(in: .whitespacesAndNewlines)
        self.title = trimmedName?.isEmpty == false ? trimmedName! : "Guest"
        self.avatarURL = response.avatarURL
    }
}
```

This shows optional thinking in UI code: not every missing value is an error. Some nil values become fallback UI, while others should become validation or decoding failures.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

