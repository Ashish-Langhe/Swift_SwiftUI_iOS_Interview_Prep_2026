# Day 37: Swift And iOS Practical Coding Prompts

For iOS interviews, logical questions are often adapted into Swift or app-like scenarios. These prompts test whether you can write clean Swift, model data, use collections, handle optionals, and think like an app developer rather than only solving abstract puzzles.

## Swift Fundamentals Coding Prompts

1. Write a function to safely unwrap an optional string and return "Unknown" if nil or empty.

2. Write a function that accepts `[Int?]` and returns only non-nil even numbers.

3. Convert an array of names into a comma-separated display string.

4. Group users by city using a dictionary.

5. Count how many times each status appears in an array.

```swift
enum OrderStatus: Hashable {
    case pending
    case shipped
    case delivered
    case cancelled
}

func countStatuses(_ statuses: [OrderStatus]) -> [OrderStatus: Int] {
    var result: [OrderStatus: Int] = [:]
    for status in statuses {
        result[status, default: 0] += 1
    }
    return result
}
```

6. Sort products by price, then by rating.

7. Remove duplicate users by ID.

8. Find the most expensive product.

9. Find all products in stock.

10. Convert API DTOs into display models.

11. Write a generic function that returns the first duplicate element.

12. Write a function that partitions numbers into even and odd arrays.

13. Write a function to paginate an array.

14. Write a function to debounce repeated search input conceptually.

15. Write a function that validates email format at a basic level.

## Model And Data Transformation Prompts

16. Given `[Transaction]`, calculate total expense by category.

17. Given `[Order]`, group orders by month.

18. Given `[Message]`, group messages by date.

19. Given `[User]`, find active premium users.

20. Given `[Product]`, produce rows for a collection view.

21. Given API response with optional fields, create safe UI display text.

22. Given two arrays of users, find newly added users.

23. Given local cached items and remote items, merge by ID.

24. Given notifications, count unread notifications by type.

25. Given expenses, find the highest spending day.

## Optionals And Error Handling Prompts

26. Parse an optional price string into `Double`.

27. Safely access nested optional API response data.

28. Convert throwing function into `Result`.

29. Convert `Result` into user-facing view state.

30. Write a function that validates a signup form and returns typed errors.

```swift
enum SignupError: Error, Equatable {
    case emptyEmail
    case invalidEmail
    case passwordTooShort
}

func validate(email: String, password: String) throws {
    guard !email.isEmpty else { throw SignupError.emptyEmail }
    guard email.contains("@") else { throw SignupError.invalidEmail }
    guard password.count >= 8 else { throw SignupError.passwordTooShort }
}
```

31. Decode JSON and handle missing fields gracefully.

32. Write a safe subscript for arrays.

33. Write a function that returns nil for invalid input instead of crashing.

34. Replace force unwraps in a sample snippet.

35. Explain when `try?` is acceptable and when it hides errors.

## UIKit/SwiftUI State Prompts

36. Model loading, loaded, empty, and failed states for a screen.

37. Convert API result into view state.

38. Write a simple view model for search filtering.

39. Write a function that applies filter and sort to products.

40. Given selected IDs, mark matching rows as selected.

41. Implement a favorite toggle using value semantics.

42. Build section models for a settings screen.

43. Create enum-driven rows for a profile screen.

44. Convert validation result into button enabled/disabled state.

45. Create a diffable-friendly item model with stable identity.

## Concurrency Prompts

46. Fetch user profile asynchronously and return display model.

47. Fetch profile and orders in parallel.

48. Cancel a search task when query changes.

49. Retry a failing async operation up to three times.

50. Write an actor-isolated cache.

51. Explain how you would avoid updating UI off the main actor.

52. Convert callback-based API to async/await using continuation.

53. Process an array of image URLs with a task group.

54. Handle partial success when multiple async calls run.

55. Add timeout behavior around an async request.

## Memory And Closure Prompts

56. Identify retain cycle in a closure-based callback.

57. Fix a timer retaining a view controller.

58. Explain weak vs unowned with example.

59. Write a closure property safely in a custom cell.

60. Avoid capturing index path in a cell callback.

```swift
cell.onTap = { [weak self] in
    self?.openDetails(id: row.id)
}
```

61. Explain why capturing `self` strongly in async work can be risky.

62. Create a delegate protocol with weak delegate.

63. Show how to cancel a task in `deinit`.

64. Explain why NotificationCenter observers can leak in some patterns.

65. Debug a sample retain cycle.

## Networking Prompts

66. Build a URL with query parameters safely.

67. Model a request type with path, method, headers, and body.

68. Decode success and error responses.

69. Add bearer token header to a request.

70. Refresh token after 401 response.

71. Retry network request with exponential backoff.

72. Upload multipart form data conceptually.

73. Cache response with expiration.

74. Convert network errors into user-facing messages.

75. Write a mock API client for tests.

## Persistence Prompts

76. Save user preference in UserDefaults.

77. Store token in Keychain conceptually.

78. Save Codable object to file.

79. Load cached JSON safely.

80. Migrate stored settings from old key to new key.

81. Decide where to store user profile, token, cache, and theme preference.

82. Explain why UserDefaults is not secure storage.

83. Build a repository that reads cache first, then network.

84. Handle corrupted local file.

85. Clear user data on logout.

## Testing Prompts

86. Write tests for email validator.

87. Write tests for view model loading state.

88. Write tests for mapper from DTO to view model.

89. Write tests for retry logic.

90. Write tests for pagination logic.

91. Write tests for duplicate-removal function.

92. Create fake repository for a view model.

93. Test async function.

94. Test error mapping.

95. Test date formatting with fixed calendar/time zone.

## Senior-Level Practical Prompts

96. Design an image loader with memory cache, disk cache, and cancellation.

97. Design a paginated feed with pull-to-refresh and retry.

98. Design offline-first notes sync.

99. Design token refresh so multiple requests do not refresh simultaneously.

100. Design a search screen with debounce, cancellation, empty state, and error state.

101. Design app settings using enum-driven rows.

102. Design a type-safe analytics event system.

103. Design a feature flag system.

104. Design a reusable API client.

105. Design dependency injection for a medium-sized iOS app.

## What Interviewers Look For

- Clean Swift function signatures.
- Correct optional handling.
- No force unwraps unless justified.
- Data modeling with structs/enums.
- Stable identity for UI lists.
- Clear separation between DTO, domain, and view model.
- Main actor awareness for UI.
- Testability.
- Edge-case thinking.

## Common Mistakes

- Solving app prompts with only abstract algorithm thinking.
- Ignoring user-facing states like empty, loading, and error.
- Returning raw API models directly to UI.
- Hiding errors with `try?`.
- Not canceling stale async work.
- Capturing index paths instead of stable IDs.
- Storing secrets in UserDefaults.

