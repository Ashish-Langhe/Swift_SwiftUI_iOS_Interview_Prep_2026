# Day 37: Patterns, Recursion, Sorting, And Problem Solving Questions

Pattern printing, recursion, and sorting questions are common in early interview rounds, campus rounds, screening rounds, and service-company coding assessments. Senior iOS roles may not ask pattern printing often, but recursion, sorting, and problem-solving patterns still matter.

## Pattern Printing Questions

1. Print a square pattern.

```text
* * * *
* * * *
* * * *
* * * *
```

2. Print a right triangle.

```text
*
* *
* * *
* * * *
```

3. Print an inverted right triangle.

```text
* * * *
* * *
* *
*
```

4. Print a number triangle.

```text
1
1 2
1 2 3
1 2 3 4
```

5. Print repeated number rows.

```text
1
2 2
3 3 3
4 4 4 4
```

6. Print an inverted number triangle.

7. Print a pyramid pattern.

8. Print an inverted pyramid.

9. Print a diamond pattern.

10. Print a hollow square.

11. Print a hollow triangle.

12. Print Floyd's triangle.

```text
1
2 3
4 5 6
7 8 9 10
```

13. Print Pascal's triangle.

14. Print an alphabet triangle.

15. Print a checkerboard pattern.

## Recursion Basics

16. Print numbers from 1 to N using recursion.

17. Print numbers from N to 1 using recursion.

18. Find factorial using recursion.

19. Find Fibonacci number using recursion.

20. Find sum of first N numbers using recursion.

21. Find power of a number using recursion.

22. Reverse a string using recursion.

23. Check palindrome using recursion.

24. Find GCD using recursion.

25. Count digits using recursion.

26. Sum digits using recursion.

27. Find maximum element in an array using recursion.

28. Find minimum element in an array using recursion.

29. Check whether array is sorted using recursion.

30. Generate all subsets of an array.

31. Generate all permutations of a string.

32. Solve Tower of Hanoi.

33. Generate valid parentheses.

34. Find all paths in a grid from top-left to bottom-right.

35. Solve basic backtracking maze problem.

## Recursion Interview Points

Every recursive solution needs:

- Base case.
- Recursive case.
- Progress toward base case.
- Stack-depth awareness.
- Complexity discussion.

Bad recursion:

```swift
func countdown(_ n: Int) {
    print(n)
    countdown(n - 1)
}
```

Good recursion:

```swift
func countdown(_ n: Int) {
    guard n >= 0 else { return }
    print(n)
    countdown(n - 1)
}
```

## Sorting Questions

36. Implement bubble sort.

37. Implement selection sort.

38. Implement insertion sort.

39. Implement merge sort.

40. Implement quick sort.

41. Sort an array in ascending order.

42. Sort an array in descending order.

43. Sort strings alphabetically.

44. Sort array by frequency.

45. Sort array of 0s, 1s, and 2s.

46. Sort dictionary entries by value.

47. Sort custom objects by multiple fields.

```swift
struct Candidate {
    let name: String
    let score: Int
    let experience: Int
}

let sorted = candidates.sorted {
    if $0.score == $1.score {
        return $0.experience > $1.experience
    }
    return $0.score > $1.score
}
```

48. Find Kth largest element.

49. Find Kth smallest element.

50. Merge overlapping intervals.

51. Sort intervals by start time.

52. Sort dates.

53. Sort case-insensitive strings.

54. Stable sort vs unstable sort discussion.

55. Explain when not to sort.
    - Example: when a hash map gives O(n) solution.

## Stack Questions

56. Implement stack using array.

57. Check balanced parentheses.

58. Evaluate postfix expression.

59. Convert infix to postfix.

60. Next greater element.

61. Previous greater element.

62. Min stack.

63. Remove adjacent duplicates from string.

64. Decode encoded string.

65. Daily temperatures.

## Queue Questions

66. Implement queue using array.

67. Implement queue using two stacks.

68. Implement circular queue.

69. Generate binary numbers from 1 to N using queue.

70. First non-repeating character in stream.

71. Sliding window maximum.

## Linked List Basics

72. Create a singly linked list.

73. Insert node at beginning.

74. Insert node at end.

75. Delete a node by value.

76. Reverse a linked list.

77. Find middle node.

78. Detect cycle in linked list.

79. Merge two sorted linked lists.

80. Remove Nth node from end.

Swift note: linked lists are less common in production Swift, but interviewers may ask them to test pointer/reference thinking.

## Basic Tree Questions

81. Preorder traversal.

82. Inorder traversal.

83. Postorder traversal.

84. Level-order traversal.

85. Find height of binary tree.

86. Count nodes in binary tree.

87. Find maximum value in binary tree.

88. Check if two trees are identical.

89. Check if binary tree is balanced.

90. Validate binary search tree.

## Basic Graph Questions

91. Represent graph using adjacency list.

92. Breadth-first search.

93. Depth-first search.

94. Find connected components.

95. Detect cycle in an undirected graph.

96. Detect cycle in a directed graph.

97. Find shortest path in unweighted graph.

98. Topological sort basics.

99. Number of islands.

100. Flood fill.

## Problem-Solving Patterns To Recognize

- Frequency map: counting duplicates, anagrams, top K.
- Two pointers: sorted array pair sum, palindrome, merging.
- Sliding window: substring/subarray with size or constraint.
- Stack: parentheses, next greater element, undo-style flows.
- Queue: BFS, level order, task processing.
- Recursion/backtracking: permutations, subsets, maze, tree traversal.
- Binary search: sorted search, answer-space search.
- Sorting: ordering, intervals, ranking.
- Greedy: choose best local decision when provable.
- Dynamic programming: overlapping subproblems and optimal substructure.

## Common Mistakes

- Writing recursive code without a base case.
- Not explaining complexity.
- Sorting when O(n) dictionary solution exists.
- Using nested loops for everything.
- Not testing empty input.
- Forgetting stack overflow risk with recursion.
- Not separating pattern printing logic from output formatting.

