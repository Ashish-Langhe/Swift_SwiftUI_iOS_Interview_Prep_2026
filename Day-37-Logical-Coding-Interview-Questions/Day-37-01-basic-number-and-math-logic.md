# Day 37: Basic Number And Math Logic Questions

These are the classic warm-up coding questions asked in junior and mid-level iOS interviews. They are not only about math. Interviewers use them to check loops, conditionals, edge cases, integer handling, function design, and whether you can explain your approach clearly.

## How To Practice These Questions

For each question, practice:

- Brute-force approach first.
- Edge cases.
- Time and space complexity.
- Clean Swift function signature.
- Test examples.
- Explanation in simple words.

## Number Basics

1. Check whether a number is even or odd.
   - Tests: modulo operator, conditionals.
   - Edge cases: zero, negative numbers.

2. Find the largest of two numbers.
   - Tests: comparison.
   - Senior note: mention generic `Comparable` if asked to generalize.

3. Find the largest of three numbers.
   - Tests: nested conditionals or `max`.
   - Edge cases: equal values.

4. Find the smallest of three numbers.
   - Tests: comparison logic.
   - Edge cases: duplicates.

5. Swap two numbers without using a third variable.
   - Tests: arithmetic reasoning.
   - Swift note: prefer tuple swap in real Swift.

```swift
var a = 10
var b = 20
(a, b) = (b, a)
```

6. Check whether a number is positive, negative, or zero.
   - Tests: branching.

7. Find the absolute value of a number.
   - Tests: conditionals and standard library awareness.

8. Check whether a number is divisible by another number.
   - Tests: modulo and division-by-zero safety.

9. Print numbers from 1 to N.
   - Tests: loops.
   - Edge cases: N <= 0.

10. Print numbers from N to 1.
    - Tests: reverse loops.

## Digit-Based Logic

11. Count the number of digits in an integer.
    - Tests: while loop, division by 10.
    - Edge cases: 0, negative numbers.

12. Find the sum of digits of a number.
    - Example: 1234 -> 10.
    - Tests: modulo and division.

13. Find the product of digits of a number.
    - Example: 234 -> 24.
    - Edge cases: digit 0.

14. Reverse the digits of a number.
    - Example: 1234 -> 4321.
    - Edge cases: trailing zero, negative numbers.

15. Check whether a number is a palindrome.
    - Example: 121 -> true.
    - Tests: reverse number or two-pointer on string.

16. Find the first digit of a number.
    - Tests: repeated division.

17. Find the last digit of a number.
    - Tests: modulo.

18. Count even and odd digits in a number.
    - Tests: loop and counters.

19. Count zeroes in a number.
    - Edge case: input 0.

20. Find the largest digit in a number.
    - Example: 79241 -> 9.

21. Find the smallest digit in a number.
    - Example: 79241 -> 1.

22. Check whether a number contains a specific digit.
    - Example: 12345 contains 3.

23. Remove all zeroes from a number.
    - Example: 102030 -> 123.
    - Tests: digit reconstruction.

24. Convert a number to the sum of squares of its digits.
    - Example: 123 -> 1 + 4 + 9 = 14.

## Prime, Factor, And Multiple Questions

25. Check whether a number is prime.
    - Tests: loops and optimization.
    - Senior note: only check divisors up to square root.

```swift
func isPrime(_ number: Int) -> Bool {
    if number < 2 { return false }
    if number == 2 { return true }
    if number % 2 == 0 { return false }

    var divisor = 3
    while divisor * divisor <= number {
        if number % divisor == 0 { return false }
        divisor += 2
    }
    return true
}
```

26. Print all prime numbers from 1 to N.
    - Tests: helper function reuse.

27. Count prime numbers in a range.
    - Tests: loops and range handling.

28. Find all factors of a number.
    - Example: 12 -> 1, 2, 3, 4, 6, 12.

29. Count the number of factors of a number.
    - Senior note: square-root optimization.

30. Find the sum of factors of a number.

31. Check whether a number is a perfect number.
    - Example: 28 -> 1 + 2 + 4 + 7 + 14 = 28.

32. Find the greatest common divisor of two numbers.
    - Tests: Euclidean algorithm.

```swift
func gcd(_ a: Int, _ b: Int) -> Int {
    var x = abs(a)
    var y = abs(b)

    while y != 0 {
        let remainder = x % y
        x = y
        y = remainder
    }

    return x
}
```

33. Find the least common multiple of two numbers.
    - Formula: `abs(a * b) / gcd(a, b)`.
    - Edge case: zero.

34. Check whether two numbers are coprime.
    - Condition: GCD is 1.

35. Find common factors of two numbers.

36. Find the highest power of 2 less than or equal to N.

37. Check whether a number is a power of 2.
    - Tests: loop or bitwise logic.

## Series Questions

38. Print the Fibonacci series up to N terms.
    - Tests: loops and variables.

39. Find the Nth Fibonacci number.
    - Junior: iterative.
    - Senior: discuss recursion, memoization, and overflow.

40. Print the arithmetic progression for N terms.

41. Print the geometric progression for N terms.

42. Find the sum of first N natural numbers.
    - Formula: `n * (n + 1) / 2`.

43. Find the sum of squares of first N natural numbers.

44. Find the factorial of a number.
    - Edge cases: 0, negative input, overflow.

45. Calculate power without using built-in power function.
    - Example: `2^5 = 32`.
    - Senior: discuss fast exponentiation.

46. Check whether a number is an Armstrong number.
    - Example: 153 -> `1^3 + 5^3 + 3^3 = 153`.

47. Print all Armstrong numbers in a range.

48. Check whether a number is a strong number.
    - Sum of factorial of digits equals the number.

49. Check whether a number is a happy number.
    - Tests: cycle detection.

50. Check whether a number is a harshad number.
    - Number divisible by sum of digits.

## Date, Time, And Calendar-Style Logic

51. Check whether a year is a leap year.
    - Rule: divisible by 400 or divisible by 4 but not 100.

52. Find the number of days in a month.
    - Tests: switch statements.
    - Edge case: February leap year.

53. Convert seconds into hours, minutes, and seconds.
    - Example: 3661 -> 1h 1m 1s.

54. Convert minutes into days, hours, and minutes.

55. Compare two time values and return the later one.

## Basic Interview Tips

- Always clarify input range.
- Mention overflow when multiplying large integers.
- Use helper functions for repeated logic.
- Avoid string conversion if the interviewer asks for numeric logic.
- Use string conversion if the interview is about readability rather than numeric constraints.
- For Swift, make functions pure when possible.

## Common Mistakes

- Forgetting that 0 and 1 are not prime.
- Infinite loops when repeatedly dividing negative numbers.
- Division by zero.
- Ignoring duplicate/equal cases.
- Not handling input 0 in digit-count problems.
- Using recursion for factorial/Fibonacci without discussing stack depth or performance.

