# Day 37: Strings, Arrays, And Collections Logical Questions

Strings and arrays are the most common coding interview area. For iOS roles, these questions test Swift fundamentals, Unicode awareness, value semantics, collection APIs, indexing, hash maps, sorting, and clean problem decomposition.

## String Questions

1. Reverse a string.
   - Tests: `String`, characters, Unicode awareness.
   - Swift note: do not assume integer indexing works on `String`.

```swift
func reversedString(_ text: String) -> String {
    String(text.reversed())
}
```

2. Check whether a string is a palindrome.
   - Example: "madam".
   - Variations: ignore spaces, punctuation, and case.

3. Count vowels and consonants in a string.
   - Tests: character iteration.

4. Count uppercase and lowercase letters.

5. Count digits, letters, spaces, and special characters.

6. Find the first non-repeating character.
   - Tests: dictionary frequency map.

7. Find the first repeating character.

8. Count frequency of each character.

9. Remove duplicate characters while preserving order.

10. Check whether two strings are anagrams.
    - Example: "listen" and "silent".
    - Tests: sorting or frequency map.

11. Find the longest word in a sentence.

12. Count the number of words in a sentence.

13. Reverse words in a sentence.
    - Example: "i love swift" -> "swift love i".

14. Reverse each word in a sentence.
    - Example: "i love swift" -> "i evol tfiws".

15. Capitalize the first letter of every word.

16. Check whether a string contains only digits.

17. Remove all spaces from a string.

18. Compress a string using character counts.
    - Example: "aaabbc" -> "a3b2c1".

19. Expand a compressed string.
    - Example: "a3b2" -> "aaabb".

20. Find the longest common prefix in an array of strings.

21. Check whether one string is a rotation of another.
    - Example: "abcd" and "cdab".

22. Find all substrings of a string.

23. Find all permutations of a string.
    - Senior note: discuss factorial complexity.

24. Find the longest substring without repeating characters.
    - Tests: sliding window.

25. Check balanced parentheses.
    - Example: `"({[]})"` is valid.
    - Tests: stack.

## Array Basics

26. Find the largest element in an array.

27. Find the smallest element in an array.

28. Find the second largest element.
    - Edge cases: duplicates, array count less than 2.

29. Find the second smallest element.

30. Find the sum of all array elements.

31. Find the average of array elements.

32. Count even and odd numbers in an array.

33. Reverse an array without using built-in reverse.

34. Check whether an array is sorted.

35. Move all zeroes to the end.
    - Example: `[0, 1, 0, 3, 12] -> [1, 3, 12, 0, 0]`.

36. Remove duplicates from an array.
    - Version 1: order does not matter.
    - Version 2: preserve order.

37. Find duplicate elements in an array.

38. Find missing number from 1 to N.
    - Example: `[1, 2, 4, 5] -> 3`.

39. Find multiple missing numbers from 1 to N.

40. Find the element that appears once when others appear twice.
    - Tests: XOR or frequency map.

41. Count frequency of each element.

42. Merge two arrays.

43. Merge two sorted arrays.

44. Find intersection of two arrays.

45. Find union of two arrays.

46. Rotate array left by one.

47. Rotate array right by one.

48. Rotate array by K positions.

49. Find leaders in an array.
    - Element greater than all elements to its right.

50. Find maximum difference between two elements where larger element comes after smaller.

## Searching Questions

51. Linear search in an array.

52. Binary search in a sorted array.
    - Tests: bounds and mid calculation.

```swift
func binarySearch(_ numbers: [Int], target: Int) -> Int? {
    var low = 0
    var high = numbers.count - 1

    while low <= high {
        let mid = low + (high - low) / 2

        if numbers[mid] == target {
            return mid
        } else if numbers[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }

    return nil
}
```

53. Find first occurrence of a target in sorted array.

54. Find last occurrence of a target in sorted array.

55. Count occurrences of a target in sorted array.

56. Search in a rotated sorted array.

57. Find peak element.

58. Find floor and ceiling of a number in sorted array.

## Two-Pointer Questions

59. Check if a sorted array has two numbers with a given sum.

60. Two Sum in an unsorted array.
    - Tests: dictionary.

```swift
func twoSum(_ numbers: [Int], target: Int) -> (Int, Int)? {
    var seen: [Int: Int] = [:]

    for (index, value) in numbers.enumerated() {
        let needed = target - value
        if let otherIndex = seen[needed] {
            return (otherIndex, index)
        }
        seen[value] = index
    }

    return nil
}
```

61. Remove duplicates from sorted array in-place.

62. Merge sorted arrays using two pointers.

63. Reverse only vowels in a string.

64. Check whether a string is palindrome using two pointers.

65. Sort array of 0s, 1s, and 2s.
    - Dutch national flag problem.

## Sliding Window Questions

66. Maximum sum subarray of size K.

67. Average of every subarray of size K.

68. First negative number in every window of size K.

69. Longest substring without repeating characters.

70. Minimum length subarray with sum greater than or equal to target.

71. Maximum number of vowels in a substring of length K.

## Dictionary And Set Questions

72. Check if array contains duplicates.

73. Find common elements between two arrays.

74. Find unique elements in an array.

75. Group anagrams.

76. Find top K frequent elements.

77. Check if two arrays have the same frequency of elements.

78. Find the first element that repeats.

79. Find the first element that does not repeat.

80. Check whether a string has all unique characters.

## Matrix Questions

81. Print a matrix row-wise.

82. Print a matrix column-wise.

83. Find sum of all elements in a matrix.

84. Find row with maximum sum.

85. Find diagonal sum of a square matrix.

86. Transpose a matrix.

87. Rotate a matrix by 90 degrees.

88. Search in a row-wise and column-wise sorted matrix.

89. Print matrix in spiral order.

90. Set matrix zeroes.

## Common Mistakes

- Using `String` integer indexing incorrectly in Swift.
- Forgetting empty array/string cases.
- Returning index paths or indices after sorting without tracking original positions.
- Ignoring duplicate values.
- Using nested loops when a dictionary would make it O(n).
- Overusing dictionaries when sorted two-pointer approach is cleaner.

## Interview Tip

When solving array/string questions, say:

1. "I will start with the simple approach."
2. "Then I will optimize using a dictionary/two pointers/sliding window if needed."
3. "Here are edge cases."
4. "Time complexity is..."
5. "Space complexity is..."

