# Day 37: Core Coding Logic Patterns With Solutions

This file focuses on the core coding logic patterns that repeatedly appear in interviews. These are not iOS-specific, but every iOS developer should be comfortable solving them in Swift because they test raw problem-solving ability.

## Pattern 1: Frequency Map

Use when the problem asks for:

- Count occurrences.
- Find duplicates.
- Find first non-repeating item.
- Check anagram.
- Top frequent values.

### Problem: First Repeating Number

```swift
func firstRepeatingNumber(_ numbers: [Int]) -> Int? {
    var seen = Set<Int>()

    for number in numbers {
        if seen.contains(number) {
            return number
        }
        seen.insert(number)
    }

    return nil
}
```

Complexity:

- Time: O(n)
- Space: O(n)

Senior note: clarify whether "first repeating" means first duplicate encountered or value whose first occurrence is earliest.

### Problem: Check Anagram

```swift
func areAnagrams(_ first: String, _ second: String) -> Bool {
    guard first.count == second.count else { return false }

    var counts: [Character: Int] = [:]

    for character in first {
        counts[character, default: 0] += 1
    }

    for character in second {
        counts[character, default: 0] -= 1
        if counts[character, default: 0] < 0 {
            return false
        }
    }

    return true
}
```

Senior note: ask if comparison should be case-insensitive and whether spaces/punctuation should be ignored.

## Pattern 2: Two Pointers

Use when:

- Array is sorted.
- You compare from both ends.
- You need pair sum.
- You need palindrome.
- You merge two sorted collections.

### Problem: Pair Sum In Sorted Array

```swift
func hasPairWithSum(_ numbers: [Int], target: Int) -> Bool {
    var left = 0
    var right = numbers.count - 1

    while left < right {
        let sum = numbers[left] + numbers[right]

        if sum == target {
            return true
        } else if sum < target {
            left += 1
        } else {
            right -= 1
        }
    }

    return false
}
```

Complexity:

- Time: O(n)
- Space: O(1)

### Problem: Valid Palindrome Ignoring Case And Spaces

```swift
func isCleanPalindrome(_ text: String) -> Bool {
    let characters = Array(
        text
            .lowercased()
            .filter { $0.isLetter || $0.isNumber }
    )

    var left = 0
    var right = characters.count - 1

    while left < right {
        if characters[left] != characters[right] {
            return false
        }
        left += 1
        right -= 1
    }

    return true
}
```

Senior note: converting to array is readable. For very large strings, discuss memory tradeoff.

## Pattern 3: Sliding Window

Use when:

- You need best subarray or substring.
- The problem says contiguous.
- Window size is fixed or condition-based.

### Problem: Maximum Sum Of K Consecutive Elements

```swift
func maxSumOfWindow(_ numbers: [Int], size k: Int) -> Int? {
    guard k > 0, numbers.count >= k else { return nil }

    var current = numbers.prefix(k).reduce(0, +)
    var best = current

    for index in k..<numbers.count {
        current += numbers[index]
        current -= numbers[index - k]
        best = max(best, current)
    }

    return best
}
```

Complexity:

- Time: O(n)
- Space: O(1)

### Problem: Longest Subarray With Sum Less Than Or Equal To Target

Works for non-negative numbers.

```swift
func longestSubarrayLength(numbers: [Int], maxSum: Int) -> Int {
    var left = 0
    var currentSum = 0
    var best = 0

    for right in numbers.indices {
        currentSum += numbers[right]

        while currentSum > maxSum, left <= right {
            currentSum -= numbers[left]
            left += 1
        }

        best = max(best, right - left + 1)
    }

    return best
}
```

Senior note: this logic depends on non-negative numbers. If negatives are allowed, shrinking the window is not always valid.

## Pattern 4: Prefix Sum

Use when:

- Many range sum queries.
- Need subarray sum.
- Need compare cumulative values.

### Problem: Range Sum Queries

```swift
struct RangeSum {
    private let prefix: [Int]

    init(_ numbers: [Int]) {
        var result = [0]
        for number in numbers {
            result.append(result.last! + number)
        }
        prefix = result
    }

    func sum(from start: Int, to end: Int) -> Int? {
        guard start >= 0, end >= start, end + 1 < prefix.count else {
            return nil
        }

        return prefix[end + 1] - prefix[start]
    }
}
```

Example:

```swift
let rangeSum = RangeSum([2, 4, 6, 8])
rangeSum.sum(from: 1, to: 3) // 18
```

Complexity:

- Build: O(n)
- Query: O(1)
- Space: O(n)

## Pattern 5: Stack

Use when:

- Last-in-first-out behavior.
- Parentheses.
- Undo.
- Next greater/smaller element.
- Monotonic stack.

### Problem: Remove Adjacent Duplicates

```swift
func removeAdjacentDuplicates(_ text: String) -> String {
    var stack: [Character] = []

    for character in text {
        if stack.last == character {
            stack.removeLast()
        } else {
            stack.append(character)
        }
    }

    return String(stack)
}
```

Example:

```swift
removeAdjacentDuplicates("abbaca") // "ca"
```

### Problem: Next Greater Element

```swift
func nextGreaterElements(_ numbers: [Int]) -> [Int] {
    var result = Array(repeating: -1, count: numbers.count)
    var stack: [Int] = []

    for index in numbers.indices {
        while let lastIndex = stack.last, numbers[index] > numbers[lastIndex] {
            result[lastIndex] = numbers[index]
            stack.removeLast()
        }
        stack.append(index)
    }

    return result
}
```

Complexity:

- Time: O(n)
- Space: O(n)

## Pattern 6: Queue And BFS

Use when:

- Level-order traversal.
- Shortest path in unweighted graph.
- Processing items in arrival order.

### Problem: Number Of Islands

```swift
func numberOfIslands(_ grid: [[Character]]) -> Int {
    guard !grid.isEmpty else { return 0 }

    var grid = grid
    let rows = grid.count
    let cols = grid[0].count
    var count = 0

    func bfs(row: Int, col: Int) {
        var queue = [(row, col)]
        grid[row][col] = "0"
        var index = 0

        while index < queue.count {
            let (r, c) = queue[index]
            index += 1

            for (nr, nc) in [(r + 1, c), (r - 1, c), (r, c + 1), (r, c - 1)] {
                guard nr >= 0, nr < rows, nc >= 0, nc < cols else { continue }
                guard grid[nr][nc] == "1" else { continue }

                grid[nr][nc] = "0"
                queue.append((nr, nc))
            }
        }
    }

    for row in 0..<rows {
        for col in 0..<cols {
            if grid[row][col] == "1" {
                count += 1
                bfs(row: row, col: col)
            }
        }
    }

    return count
}
```

Senior note: avoid `removeFirst()` for queues in Swift arrays because it is O(n). Use an index pointer.

## Pattern 7: Binary Search

Use when:

- Data is sorted.
- Search space is monotonic.
- You can answer yes/no for a candidate value.

### Problem: First Index Greater Than Or Equal To Target

```swift
func lowerBound(_ numbers: [Int], target: Int) -> Int {
    var low = 0
    var high = numbers.count

    while low < high {
        let mid = low + (high - low) / 2

        if numbers[mid] < target {
            low = mid + 1
        } else {
            high = mid
        }
    }

    return low
}
```

Example:

```swift
lowerBound([1, 3, 3, 5], target: 3) // 1
lowerBound([1, 3, 3, 5], target: 4) // 3
```

## Pattern 8: Sorting And Intervals

Use sorting when ordering simplifies relationships.

### Problem: Can Attend All Meetings

```swift
struct Meeting {
    let start: Int
    let end: Int
}

func canAttendAllMeetings(_ meetings: [Meeting]) -> Bool {
    let sorted = meetings.sorted { $0.start < $1.start }

    for index in 1..<sorted.count {
        if sorted[index].start < sorted[index - 1].end {
            return false
        }
    }

    return true
}
```

Senior note: clarify whether one meeting ending at 10 and another starting at 10 overlaps. Usually it does not.

## Pattern 9: Greedy

Use when a local best decision can be proven to lead to global best.

### Problem: Minimum Number Of Coins For Canonical Currency

```swift
func minimumCoins(amount: Int, coins: [Int]) -> Int? {
    guard amount >= 0 else { return nil }

    var remaining = amount
    var count = 0

    for coin in coins.sorted(by: >) {
        count += remaining / coin
        remaining %= coin
    }

    return remaining == 0 ? count : nil
}
```

Senior note: greedy coin change is not correct for every coin system. For arbitrary coins, use dynamic programming.

## Pattern 10: Dynamic Programming Basics

Use when:

- Problems overlap.
- Choices repeat.
- You need best/count/min/max over states.

### Problem: Climbing Stairs

```swift
func climbStairs(_ n: Int) -> Int {
    guard n > 1 else { return 1 }

    var previous = 1
    var current = 1

    for _ in 2...n {
        let next = previous + current
        previous = current
        current = next
    }

    return current
}
```

Complexity:

- Time: O(n)
- Space: O(1)

### Problem: Minimum Cost Climbing Stairs

```swift
func minCostClimbingStairs(_ cost: [Int]) -> Int {
    var oneStepBefore = 0
    var twoStepsBefore = 0

    for value in cost {
        let current = min(oneStepBefore, twoStepsBefore) + value
        twoStepsBefore = oneStepBefore
        oneStepBefore = current
    }

    return min(oneStepBefore, twoStepsBefore)
}
```

## Core Logic Pattern Recognition Checklist

- Count or frequency? Dictionary.
- Unique membership? Set.
- Sorted pair? Two pointers.
- Contiguous range? Sliding window or prefix sum.
- Nested/last-open behavior? Stack.
- Level traversal? Queue/BFS.
- Sorted search or monotonic condition? Binary search.
- Overlapping ranges? Sort intervals.
- Local best with proof? Greedy.
- Repeated choices/states? Dynamic programming.

