# Day 37: Solved Core Logic Problems In Swift

This file contains solved logical problems that I would use in interviews to test core Swift problem solving. Each problem includes what it tests, a clean solution, complexity, and senior-level discussion.

## Problem 1: First Non-Repeating Character

### Prompt

Given a string, return the first character that appears only once.

### What It Tests

- Dictionary frequency map.
- Character iteration.
- Unicode-safe Swift string handling.
- Two-pass reasoning.

### Solution

```swift
func firstNonRepeatingCharacter(in text: String) -> Character? {
    var counts: [Character: Int] = [:]

    for character in text {
        counts[character, default: 0] += 1
    }

    for character in text {
        if counts[character] == 1 {
            return character
        }
    }

    return nil
}
```

### Complexity

- Time: O(n)
- Space: O(k), where k is the number of unique characters.

### Senior Discussion

Ask whether comparison should be case-sensitive. For `"Swift swift"`, should `S` and `s` be treated as same? If the input is user-facing text, Unicode normalization may matter. In most interviews, `Character` iteration is enough.

## Problem 2: Remove Duplicates While Preserving Order

### Prompt

Given an array of integers, remove duplicates while preserving the first occurrence order.

### Solution

```swift
func removeDuplicatesPreservingOrder(_ numbers: [Int]) -> [Int] {
    var seen = Set<Int>()
    var result: [Int] = []

    for number in numbers {
        if !seen.contains(number) {
            seen.insert(number)
            result.append(number)
        }
    }

    return result
}
```

### Example

```swift
removeDuplicatesPreservingOrder([3, 1, 3, 2, 1, 4])
// [3, 1, 2, 4]
```

### Complexity

- Time: O(n)
- Space: O(n)

### Senior Discussion

If this were an array of users, use stable ID instead of entire object equality.

```swift
struct User {
    let id: String
    let name: String
}

func uniqueUsersByID(_ users: [User]) -> [User] {
    var seenIDs = Set<String>()
    var result: [User] = []

    for user in users {
        if seenIDs.insert(user.id).inserted {
            result.append(user)
        }
    }

    return result
}
```

This is more realistic for iOS list data.

## Problem 3: Two Sum

### Prompt

Given an array and a target, return indices of two numbers that add up to the target.

### Solution

```swift
func twoSum(_ numbers: [Int], target: Int) -> (Int, Int)? {
    var indexByValue: [Int: Int] = [:]

    for (index, value) in numbers.enumerated() {
        let needed = target - value

        if let otherIndex = indexByValue[needed] {
            return (otherIndex, index)
        }

        indexByValue[value] = index
    }

    return nil
}
```

### Complexity

- Time: O(n)
- Space: O(n)

### Senior Discussion

Clarify:

- Can the same element be used twice? Usually no.
- Are duplicates allowed? Usually yes.
- Return indices or values?
- Return first pair or all pairs?

For sorted arrays, a two-pointer solution can use O(1) extra space.

## Problem 4: Move Zeroes To End

### Prompt

Move all zeroes to the end while preserving the order of non-zero elements.

### Solution

```swift
func moveZeroesToEnd(_ numbers: inout [Int]) {
    var insertIndex = 0

    for number in numbers where number != 0 {
        numbers[insertIndex] = number
        insertIndex += 1
    }

    while insertIndex < numbers.count {
        numbers[insertIndex] = 0
        insertIndex += 1
    }
}
```

### Example

```swift
var values = [0, 1, 0, 3, 12]
moveZeroesToEnd(&values)
// [1, 3, 12, 0, 0]
```

### Complexity

- Time: O(n)
- Space: O(1)

### Senior Discussion

This is stable for non-zero elements. If order does not matter, swapping can reduce writes in some cases, but changes behavior.

## Problem 5: Longest Substring Without Repeating Characters

### Prompt

Given a string, return the length of the longest substring without repeating characters.

### Solution

```swift
func lengthOfLongestUniqueSubstring(_ text: String) -> Int {
    let characters = Array(text)
    var lastSeen: [Character: Int] = [:]
    var windowStart = 0
    var best = 0

    for (index, character) in characters.enumerated() {
        if let previousIndex = lastSeen[character], previousIndex >= windowStart {
            windowStart = previousIndex + 1
        }

        lastSeen[character] = index
        best = max(best, index - windowStart + 1)
    }

    return best
}
```

### Complexity

- Time: O(n)
- Space: O(k)

### Senior Discussion

Converting to `Array(text)` makes indexing easier but costs O(n) memory. For many interviews this is acceptable. In production text-heavy code, be more careful with Unicode and indexing.

## Problem 6: Group Anagrams

### Prompt

Group words that are anagrams.

### Solution

```swift
func groupAnagrams(_ words: [String]) -> [[String]] {
    var groups: [String: [String]] = [:]

    for word in words {
        let key = String(word.sorted())
        groups[key, default: []].append(word)
    }

    return Array(groups.values)
}
```

### Complexity

- Time: O(n * m log m), where m is average word length.
- Space: O(n * m)

### Senior Discussion

Sorting each word is simple. A frequency-signature key can be O(m) for limited alphabets, but gets more complex with Unicode. Discuss constraints before optimizing.

## Problem 7: Merge Intervals

### Prompt

Given intervals, merge overlapping intervals.

### Solution

```swift
struct Interval: Equatable {
    let start: Int
    let end: Int
}

func mergeIntervals(_ intervals: [Interval]) -> [Interval] {
    guard !intervals.isEmpty else { return [] }

    let sorted = intervals.sorted { $0.start < $1.start }
    var result: [Interval] = []
    var current = sorted[0]

    for interval in sorted.dropFirst() {
        if interval.start <= current.end {
            current = Interval(
                start: current.start,
                end: max(current.end, interval.end)
            )
        } else {
            result.append(current)
            current = interval
        }
    }

    result.append(current)
    return result
}
```

### Complexity

- Time: O(n log n)
- Space: O(n)

### Senior Discussion

Clarify whether touching intervals merge:

- `[1, 3]` and `[3, 5]` merge if boundaries are inclusive.
- They may not merge if end is exclusive.

This kind of question appears in calendar, booking, timeline, and analytics features.

## Problem 8: Safe Array Subscript

### Prompt

Write a safe way to access an array element by index without crashing.

### Solution

```swift
extension Array {
    subscript(safe index: Index) -> Element? {
        indices.contains(index) ? self[index] : nil
    }
}
```

### Example

```swift
let values = ["A", "B"]
values[safe: 1] // "B"
values[safe: 5] // nil
```

### Complexity

- Time: O(1) for Array index check.
- Space: O(1)

### Senior Discussion

This is useful at UI boundaries, but do not use it to hide logic errors everywhere. If an index should always exist, failing fast during development may reveal a real bug.

## Problem 9: Top K Frequent Elements

### Prompt

Return the K most frequent integers.

### Solution

```swift
func topKFrequent(_ numbers: [Int], k: Int) -> [Int] {
    guard k > 0 else { return [] }

    var counts: [Int: Int] = [:]
    for number in numbers {
        counts[number, default: 0] += 1
    }

    return counts
        .sorted { lhs, rhs in
            if lhs.value == rhs.value {
                return lhs.key < rhs.key
            }
            return lhs.value > rhs.value
        }
        .prefix(k)
        .map(\.key)
}
```

### Complexity

- Time: O(n + u log u), where u is unique values.
- Space: O(u)

### Senior Discussion

For very large input and small K, use a heap to avoid sorting all unique elements. Swift standard library does not include a built-in heap, so discuss implementing one or using a priority queue abstraction.

## Problem 10: Balanced Parentheses

### Prompt

Check if brackets are balanced.

### Solution

```swift
func isBalanced(_ text: String) -> Bool {
    let pairs: [Character: Character] = [
        ")": "(",
        "]": "[",
        "}": "{"
    ]
    var stack: [Character] = []

    for character in text {
        if ["(", "[", "{"].contains(character) {
            stack.append(character)
        } else if let expectedOpening = pairs[character] {
            guard stack.last == expectedOpening else {
                return false
            }
            stack.removeLast()
        }
    }

    return stack.isEmpty
}
```

### Complexity

- Time: O(n)
- Space: O(n)

### Senior Discussion

Ask whether non-bracket characters should be ignored or make input invalid. The solution above ignores non-bracket characters.

## Problem 11: Binary Search

### Prompt

Return the index of a target in a sorted array.

### Solution

```swift
func binarySearch(_ numbers: [Int], target: Int) -> Int? {
    var low = 0
    var high = numbers.count - 1

    while low <= high {
        let mid = low + (high - low) / 2

        if numbers[mid] == target {
            return mid
        }

        if numbers[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }

    return nil
}
```

### Complexity

- Time: O(log n)
- Space: O(1)

### Senior Discussion

Be careful with empty arrays. In Swift, `numbers.count - 1` is safe here only because `high` becomes `-1` for empty arrays and the loop does not execute. With unsigned integers in other languages, this can underflow.

## Problem 12: Maximum Sum Subarray Of Size K

### Prompt

Given an array and window size K, return the maximum sum of any contiguous subarray of size K.

### Solution

```swift
func maximumWindowSum(_ numbers: [Int], size k: Int) -> Int? {
    guard k > 0, numbers.count >= k else { return nil }

    var currentSum = numbers.prefix(k).reduce(0, +)
    var best = currentSum

    for index in k..<numbers.count {
        currentSum += numbers[index]
        currentSum -= numbers[index - k]
        best = max(best, currentSum)
    }

    return best
}
```

### Complexity

- Time: O(n)
- Space: O(1)

### Senior Discussion

This tests whether the candidate recognizes sliding window instead of recalculating each window sum from scratch.

