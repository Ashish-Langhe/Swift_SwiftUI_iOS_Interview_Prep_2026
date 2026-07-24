# Day 37: Basic Array Coding Questions With Solutions

Arrays are the first serious coding interview topic for many iOS developers. These questions test loops, indexing, mutation, edge cases, and Swift collection APIs. A beginner should solve these with loops first, then learn the standard-library version.

## How To Approach Array Questions

For every array question, clarify:

- Can the array be empty?
- Can values repeat?
- Are negative numbers allowed?
- Should original order be preserved?
- Should the solution mutate the original array?
- Should we return value, index, or both?

## 1. Find The Largest Element

### Loop Solution

```swift
func largestElement(_ numbers: [Int]) -> Int? {
    guard var largest = numbers.first else {
        return nil
    }

    for number in numbers.dropFirst() {
        if number > largest {
            largest = number
        }
    }

    return largest
}
```

### Swift Standard Library

```swift
func largestElementUsingMax(_ numbers: [Int]) -> Int? {
    numbers.max()
}
```

### Edge Cases

- `[] -> nil`
- `[5] -> 5`
- `[-10, -3, -8] -> -3`

## 2. Find The Smallest Element

```swift
func smallestElement(_ numbers: [Int]) -> Int? {
    guard var smallest = numbers.first else {
        return nil
    }

    for number in numbers.dropFirst() {
        if number < smallest {
            smallest = number
        }
    }

    return smallest
}
```

Complexity:

- Time: O(n)
- Space: O(1)

## 3. Find The Sum Of Elements

```swift
func sumOfElements(_ numbers: [Int]) -> Int {
    var total = 0

    for number in numbers {
        total += number
    }

    return total
}
```

Swift version:

```swift
func sumUsingReduce(_ numbers: [Int]) -> Int {
    numbers.reduce(0, +)
}
```

## 4. Find The Average

```swift
func average(_ numbers: [Int]) -> Double? {
    guard !numbers.isEmpty else {
        return nil
    }

    let total = numbers.reduce(0, +)
    return Double(total) / Double(numbers.count)
}
```

Senior note: for very large integer arrays, discuss overflow risk when summing.

## 5. Count Even And Odd Numbers

```swift
func countEvenAndOdd(_ numbers: [Int]) -> (even: Int, odd: Int) {
    var even = 0
    var odd = 0

    for number in numbers {
        if number % 2 == 0 {
            even += 1
        } else {
            odd += 1
        }
    }

    return (even, odd)
}
```

Example:

```swift
countEvenAndOdd([1, 2, 3, 4, 5])
// even: 2, odd: 3
```

## 6. Reverse An Array

### New Array Solution

```swift
func reversedArray(_ numbers: [Int]) -> [Int] {
    var result: [Int] = []

    for number in numbers.reversed() {
        result.append(number)
    }

    return result
}
```

### In-Place Two-Pointer Solution

```swift
func reverseInPlace(_ numbers: inout [Int]) {
    var left = 0
    var right = numbers.count - 1

    while left < right {
        numbers.swapAt(left, right)
        left += 1
        right -= 1
    }
}
```

Important: for an empty array, `right` becomes `-1`, and the loop safely does not run.

## 7. Check If Array Is Sorted

```swift
func isSortedAscending(_ numbers: [Int]) -> Bool {
    guard numbers.count > 1 else {
        return true
    }

    for index in 1..<numbers.count {
        if numbers[index] < numbers[index - 1] {
            return false
        }
    }

    return true
}
```

Edge cases:

- Empty array is sorted.
- Single-element array is sorted.
- Equal adjacent values are allowed.

## 8. Find Second Largest Element

```swift
func secondLargest(_ numbers: [Int]) -> Int? {
    var largest: Int?
    var second: Int?

    for number in numbers {
        if largest == nil || number > largest! {
            if number != largest {
                second = largest
            }
            largest = number
        } else if number != largest, (second == nil || number > second!) {
            second = number
        }
    }

    return second
}
```

Examples:

```swift
secondLargest([10, 5, 8, 10]) // 8
secondLargest([3, 3, 3]) // nil
```

Interview note: clarify whether duplicates count. Usually second largest means second distinct largest.

## 9. Remove Duplicates While Preserving Order

```swift
func removeDuplicates(_ numbers: [Int]) -> [Int] {
    var seen = Set<Int>()
    var result: [Int] = []

    for number in numbers {
        if seen.insert(number).inserted {
            result.append(number)
        }
    }

    return result
}
```

Complexity:

- Time: O(n)
- Space: O(n)

## 10. Find Duplicate Values

```swift
func duplicateValues(_ numbers: [Int]) -> [Int] {
    var seen = Set<Int>()
    var duplicates = Set<Int>()

    for number in numbers {
        if !seen.insert(number).inserted {
            duplicates.insert(number)
        }
    }

    return Array(duplicates)
}
```

Senior note: result order is not guaranteed because `Set` is unordered. Preserve order if the UI needs stable display.

## 11. Find Missing Number From 1 To N

```swift
func missingNumber(_ numbers: [Int], n: Int) -> Int {
    let expected = n * (n + 1) / 2
    let actual = numbers.reduce(0, +)
    return expected - actual
}
```

Example:

```swift
missingNumber([1, 2, 4, 5], n: 5) // 3
```

Senior note: sum formula can overflow for huge `n`; XOR is an alternative.

## 12. Move Zeroes To End

```swift
func moveZeroesToEnd(_ numbers: [Int]) -> [Int] {
    var nonZero: [Int] = []
    var zeroCount = 0

    for number in numbers {
        if number == 0 {
            zeroCount += 1
        } else {
            nonZero.append(number)
        }
    }

    return nonZero + Array(repeating: 0, count: zeroCount)
}
```

In-place version:

```swift
func moveZeroesToEndInPlace(_ numbers: inout [Int]) {
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

## 13. Rotate Array Right By One

```swift
func rotateRightByOne(_ numbers: [Int]) -> [Int] {
    guard let last = numbers.last else {
        return []
    }

    return [last] + numbers.dropLast()
}
```

## 14. Rotate Array Right By K

```swift
func rotateRight(_ numbers: [Int], by k: Int) -> [Int] {
    guard !numbers.isEmpty else { return [] }

    let shift = k % numbers.count
    guard shift != 0 else { return numbers }

    let splitIndex = numbers.count - shift
    return Array(numbers[splitIndex...]) + Array(numbers[..<splitIndex])
}
```

Edge cases:

- Empty array.
- `k == 0`.
- `k > count`.

## 15. Merge Two Sorted Arrays

```swift
func mergeSortedArrays(_ first: [Int], _ second: [Int]) -> [Int] {
    var i = 0
    var j = 0
    var result: [Int] = []

    while i < first.count && j < second.count {
        if first[i] <= second[j] {
            result.append(first[i])
            i += 1
        } else {
            result.append(second[j])
            j += 1
        }
    }

    result.append(contentsOf: first[i...])
    result.append(contentsOf: second[j...])
    return result
}
```

Careful: the slices `first[i...]` and `second[j...]` are safe here because `i` and `j` can equal `count`; Swift supports an empty suffix at `endIndex`.

## 16. Find Intersection Of Two Arrays

```swift
func intersection(_ first: [Int], _ second: [Int]) -> [Int] {
    let secondSet = Set(second)
    var result: [Int] = []
    var added = Set<Int>()

    for value in first {
        if secondSet.contains(value), added.insert(value).inserted {
            result.append(value)
        }
    }

    return result
}
```

This preserves order from the first array.

## 17. Find Pair With Given Sum

```swift
func pairWithSum(_ numbers: [Int], target: Int) -> (Int, Int)? {
    var seen = Set<Int>()

    for number in numbers {
        let needed = target - number

        if seen.contains(needed) {
            return (needed, number)
        }

        seen.insert(number)
    }

    return nil
}
```

## 18. Find Maximum Subarray Sum

Kadane's algorithm:

```swift
func maximumSubarraySum(_ numbers: [Int]) -> Int? {
    guard var best = numbers.first else {
        return nil
    }

    var current = best

    for number in numbers.dropFirst() {
        current = max(number, current + number)
        best = max(best, current)
    }

    return best
}
```

Example:

```swift
maximumSubarraySum([-2, 1, -3, 4, -1, 2, 1, -5, 4])
// 6 for [4, -1, 2, 1]
```

## 19. Product Of Array Except Self

```swift
func productExceptSelf(_ numbers: [Int]) -> [Int] {
    guard !numbers.isEmpty else { return [] }

    var result = Array(repeating: 1, count: numbers.count)
    var prefix = 1

    for index in numbers.indices {
        result[index] = prefix
        prefix *= numbers[index]
    }

    var suffix = 1

    for index in numbers.indices.reversed() {
        result[index] *= suffix
        suffix *= numbers[index]
    }

    return result
}
```

Senior note: this avoids division and handles zeroes naturally.

## 20. Basic Array Pagination

```swift
func page<T>(_ values: [T], page: Int, pageSize: Int) -> [T] {
    guard page >= 1, pageSize > 0 else {
        return []
    }

    let start = (page - 1) * pageSize
    guard start < values.count else {
        return []
    }

    let end = min(start + pageSize, values.count)
    return Array(values[start..<end])
}
```

This appears in real iOS features when preparing local data for a UI list.

## Basic Array Interview Checklist

- Empty array.
- One element.
- Duplicates.
- Negative numbers.
- Preserve order or not.
- In-place or new array.
- Return index or value.
- Time complexity.
- Space complexity.

