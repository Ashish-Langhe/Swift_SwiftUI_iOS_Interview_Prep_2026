# Day 5: When To Use Array Vs Set Vs Dictionary

## Quick Decision

Use `Array` when order matters.

Use `Set` when uniqueness and membership checks matter.

Use `Dictionary` when lookup by key matters.

## Array Example

```swift
let feedPosts: [Post] = []
```

Use because feed order matters.

## Set Example

```swift
var selectedIds: Set<String> = []
```

Use because each ID should appear once and `contains` should be fast.

## Dictionary Example

```swift
var usersById: [String: User] = [:]
```

Use because you need fast lookup by user ID.

## Real iOS Scenario

You fetch users from an API:

```swift
let users: [User]
let selectedUserIds: Set<String>
let usersById: [String: User]
```

Same domain, different access patterns.

## Modern Swift 6.x Notes

Modern Swift encourages precise modeling. Choosing the right collection reduces accidental complexity and makes concurrency-safe snapshots easier to reason about.

## Interview Levels

Junior:

Array is ordered, Set is unique, Dictionary has keys and values.

Mid-level:

I choose based on whether I need order, uniqueness, or key lookup.

Senior:

The correct collection follows the access pattern. UI rendering often uses arrays; selection state often uses sets; normalized caches often use dictionaries. Sometimes you keep more than one structure when both order and lookup performance matter.

## Quick Notes

- Ordered list: `Array`
- Unique membership: `Set`
- Lookup by ID: `Dictionary`
- Sort sets/dictionaries when displaying
- Avoid using arrays as dictionaries with manual searches

## Interview Depth

Junior answer: Use array for ordered lists, set for unique values, and dictionary for key-value pairs.

Mid-level answer: The same domain may need multiple collection types. A user list can be an array for display, a set for selected IDs, and a dictionary for lookup.

Senior answer: Collection choice should follow the dominant read/write pattern. If you need both order and fast lookup, use two structures deliberately and keep them consistent through a single owner.

iOS use case:

```swift
struct UsersState {
    var orderedIds: [String] = []
    var usersById: [String: User] = [:]
    var selectedIds: Set<String> = []
}
```

Common mistakes:

- Using array for everything.
- Losing display order by using only dictionary.
- Using set when duplicate counts matter.

Practice:

1. Choose a collection for search results.
2. Choose a collection for blocked user IDs.
3. Choose a collection for cached products by ID.
