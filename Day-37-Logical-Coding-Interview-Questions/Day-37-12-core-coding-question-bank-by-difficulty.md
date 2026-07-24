# Day 37: Core Coding Question Bank By Difficulty

This file lists the core coding questions I would use in interview preparation, grouped by difficulty. Use this as a checklist for coding rounds.

## Easy: Must-Solve Basics

1. Reverse a string.
2. Check string palindrome.
3. Count vowels in a string.
4. Count words in a sentence.
5. Find first non-repeating character.
6. Find first repeating character.
7. Check if two strings are anagrams.
8. Remove duplicate characters.
9. Reverse words in a sentence.
10. Compress repeated characters.
11. Find largest element in array.
12. Find second largest element.
13. Find smallest element.
14. Check if array is sorted.
15. Reverse an array.
16. Move zeroes to end.
17. Remove duplicates from array.
18. Find missing number from 1 to N.
19. Count frequency of elements.
20. Find intersection of two arrays.
21. Find union of two arrays.
22. Linear search.
23. Binary search.
24. Factorial.
25. Fibonacci.
26. Prime number.
27. GCD and LCM.
28. Sum of digits.
29. Reverse number.
30. Palindrome number.

## Easy: What A Good Answer Should Mention

- Empty input.
- Single item input.
- Duplicate values.
- Negative numbers where relevant.
- Time and space complexity.
- Why a dictionary or set improves performance.
- Swift `String` indexing limitations.

## Medium: Arrays And Strings

31. Two Sum.
32. Three Sum basics.
33. Longest substring without repeating characters.
34. Maximum sum subarray of size K.
35. Minimum size subarray sum.
36. Sort array of 0s, 1s, and 2s.
37. Rotate array by K.
38. Search in rotated sorted array.
39. Find first and last position of element in sorted array.
40. Merge two sorted arrays.
41. Merge intervals.
42. Insert interval.
43. Product of array except self.
44. Maximum subarray sum.
45. Kadane's algorithm.
46. Container with most water.
47. Best time to buy and sell stock.
48. Group anagrams.
49. Longest common prefix.
50. Valid parentheses.
51. Remove adjacent duplicates.
52. Decode string.
53. Next greater element.
54. Daily temperatures.
55. Sliding window maximum.

## Medium: Recursion And Backtracking

56. Generate all subsets.
57. Generate all permutations.
58. Generate valid parentheses.
59. Combination sum.
60. Letter combinations of phone number.
61. Word search in grid.
62. Rat in a maze.
63. N-Queens basics.
64. Recursive binary search.
65. Recursive tree traversal.

## Medium: Linked List

66. Reverse linked list.
67. Find middle node.
68. Detect cycle.
69. Merge two sorted linked lists.
70. Remove Nth node from end.
71. Find intersection node.
72. Add two numbers represented by linked lists.
73. Reorder list.
74. Palindrome linked list.
75. Sort linked list basics.

## Medium: Trees And Graphs

76. Binary tree preorder traversal.
77. Binary tree inorder traversal.
78. Binary tree postorder traversal.
79. Level order traversal.
80. Maximum depth of binary tree.
81. Minimum depth of binary tree.
82. Validate binary search tree.
83. Lowest common ancestor basics.
84. Diameter of binary tree.
85. Invert binary tree.
86. Check if tree is balanced.
87. Number of islands.
88. Flood fill.
89. Clone graph.
90. BFS shortest path in grid.

## Harder But Useful

91. LRU cache.
92. Trie implementation.
93. Word break.
94. Coin change.
95. Longest increasing subsequence.
96. Edit distance basics.
97. Median of two sorted arrays.
98. Minimum window substring.
99. Course schedule.
100. Serialize and deserialize binary tree.

## iOS-Flavored Core Logic Questions

101. Given messages, group them by day and sort newest first.
102. Given transactions, calculate monthly spending summary.
103. Given contacts, group by first letter.
104. Given products, filter by category, sort by price, and paginate.
105. Given local and remote arrays, merge by stable ID.
106. Given diffable rows, find inserted, removed, and updated IDs.
107. Given API errors, group retryable vs non-retryable.
108. Given image URLs, remove duplicates while preserving order.
109. Given selected IDs, update visible rows.
110. Given cached records, remove expired items.
111. Given search query, rank results by prefix match before contains match.
112. Given notifications, return unread count by section.
113. Given app events, calculate session duration.
114. Given route strings, parse app deep links.
115. Given feature flags, resolve final enabled state from defaults and overrides.

## Senior-Level Core Logic Questions

116. Design and implement an LRU cache.
117. Implement a thread-safe cache using actor.
118. Coalesce duplicate in-flight requests.
119. Build retry policy with backoff and max attempts.
120. Build pagination state machine.
121. Prevent stale search results from rendering.
122. Merge offline edits with remote response.
123. Detect conflicts between local and remote records.
124. Build a dependency graph and detect cycles.
125. Topologically sort feature modules.
126. Build a rate limiter.
127. Build a debouncer.
128. Build a throttler.
129. Build undo/redo stack.
130. Build recent searches with capacity limit and uniqueness.

## Detailed Example: Recent Searches

### Prompt

Maintain recent searches. Requirements:

- Most recent first.
- No duplicates.
- Capacity is limited.
- Searching existing query moves it to top.

### Solution

```swift
struct RecentSearches {
    private(set) var values: [String] = []
    let capacity: Int

    mutating func add(_ query: String) {
        let trimmed = query.trimmingCharacters(in: .whitespacesAndNewlines)
        guard !trimmed.isEmpty else { return }

        values.removeAll { $0.caseInsensitiveCompare(trimmed) == .orderedSame }
        values.insert(trimmed, at: 0)

        if values.count > capacity {
            values.removeLast(values.count - capacity)
        }
    }
}
```

### Why This Is A Good iOS Interview Question

It tests:

- Array mutation.
- Duplicate handling.
- Order preservation.
- Case-insensitive comparison.
- Capacity limit.
- Real app behavior.

## Detailed Example: Debouncer

### Prompt

Create search debounce logic so typing does not call API for every character.

### Solution

```swift
@MainActor
final class Debouncer {
    private var task: Task<Void, Never>?
    private let delay: Duration

    init(delay: Duration) {
        self.delay = delay
    }

    func schedule(_ operation: @escaping @MainActor () -> Void) {
        task?.cancel()
        task = Task {
            do {
                try await Task.sleep(for: delay)
                guard !Task.isCancelled else { return }
                operation()
            } catch {
                return
            }
        }
    }

    deinit {
        task?.cancel()
    }
}
```

### Senior Discussion

This checks cancellation, main actor awareness, and lifecycle cleanup. A production version may support async operations, test clocks, and explicit cancel.

## Detailed Example: LRU Cache

### Prompt

Implement an LRU cache with fixed capacity.

### Simple Interview Version

```swift
struct LRUCache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    private var usageOrder: [Key] = []
    let capacity: Int

    mutating func value(for key: Key) -> Value? {
        guard let value = storage[key] else { return nil }
        markUsed(key)
        return value
    }

    mutating func setValue(_ value: Value, for key: Key) {
        storage[key] = value
        markUsed(key)

        while usageOrder.count > capacity, let leastUsed = usageOrder.first {
            usageOrder.removeFirst()
            storage.removeValue(forKey: leastUsed)
        }
    }

    private mutating func markUsed(_ key: Key) {
        usageOrder.removeAll { $0 == key }
        usageOrder.append(key)
    }
}
```

### Complexity

This simple version has O(n) usage updates because `removeAll` and `removeFirst` are array operations.

### Senior Discussion

For production-level O(1), use:

- Dictionary from key to linked-list node.
- Doubly linked list for usage order.

In a Swift interview, I would accept the simple version if the candidate clearly explains the tradeoff and how to improve it.

## Detailed Example: Pagination State Machine

```swift
enum PaginationState<Item> {
    case idle
    case loadingFirstPage
    case loaded(items: [Item], nextCursor: String?)
    case loadingNextPage(items: [Item], currentCursor: String)
    case failedFirstPage(String)
    case failedNextPage(items: [Item], message: String, retryCursor: String)
}
```

Why this matters:

- Prevents invalid boolean combinations.
- Separates first-page failure from next-page failure.
- Keeps existing content visible during pagination errors.

## What To Master First

Priority order:

1. Arrays and strings.
2. Dictionary and set problems.
3. Two pointers.
4. Sliding window.
5. Stack and queue.
6. Binary search.
7. Sorting and intervals.
8. Recursion/backtracking.
9. Trees and graphs.
10. Basic dynamic programming.

## Architect Takeaway

Core coding logic is not separate from iOS work. The same patterns power search, feeds, diffing, caching, pagination, sync, state machines, analytics, local persistence, and performance-sensitive UI data preparation.

