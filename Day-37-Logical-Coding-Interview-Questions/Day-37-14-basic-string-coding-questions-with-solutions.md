# Day 37: Basic String Coding Questions With Solutions

String questions are very common in interviews. In Swift, they are especially important because `String` is Unicode-correct. A strong iOS candidate should know simple string logic and also understand why Swift strings are not indexed like arrays of bytes.

## Swift String Interview Reminder

Swift `String` is a collection of `Character` values. A `Character` may contain multiple Unicode scalars.

```swift
let text = "cafe\u{301}"
print(text.count) // 4
```

Do not assume every visible character is one byte.

## 1. Reverse A String

```swift
func reverseString(_ text: String) -> String {
    String(text.reversed())
}
```

Loop version:

```swift
func reverseStringUsingLoop(_ text: String) -> String {
    var result = ""

    for character in text {
        result = String(character) + result
    }

    return result
}
```

Senior note: repeated string prepending can be inefficient. Prefer `reversed()` or build an array for larger input.

## 2. Check Palindrome

```swift
func isPalindrome(_ text: String) -> Bool {
    let characters = Array(text)
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

Edge cases:

- Empty string -> true.
- One character -> true.
- Case-sensitive unless specified.

## 3. Palindrome Ignoring Spaces, Case, And Punctuation

```swift
func isNormalizedPalindrome(_ text: String) -> Bool {
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

Example:

```swift
isNormalizedPalindrome("A man, a plan, a canal: Panama") // true
```

## 4. Count Vowels

```swift
func vowelCount(_ text: String) -> Int {
    let vowels = Set("aeiouAEIOU")
    var count = 0

    for character in text {
        if vowels.contains(character) {
            count += 1
        }
    }

    return count
}
```

## 5. Count Vowels And Consonants

```swift
func countVowelsAndConsonants(_ text: String) -> (vowels: Int, consonants: Int) {
    let vowels = Set("aeiouAEIOU")
    var vowelCount = 0
    var consonantCount = 0

    for character in text {
        guard character.isLetter else { continue }

        if vowels.contains(character) {
            vowelCount += 1
        } else {
            consonantCount += 1
        }
    }

    return (vowelCount, consonantCount)
}
```

## 6. Count Words

```swift
func countWords(_ sentence: String) -> Int {
    sentence
        .split(whereSeparator: { $0.isWhitespace })
        .count
}
```

Why this is better than splitting only by `" "`:

- Handles multiple spaces.
- Handles tabs and newlines.

## 7. Count Character Frequency

```swift
func characterFrequency(_ text: String) -> [Character: Int] {
    var counts: [Character: Int] = [:]

    for character in text {
        counts[character, default: 0] += 1
    }

    return counts
}
```

## 8. First Non-Repeating Character

```swift
func firstNonRepeatingCharacter(_ text: String) -> Character? {
    let counts = characterFrequency(text)

    for character in text {
        if counts[character] == 1 {
            return character
        }
    }

    return nil
}
```

## 9. First Repeating Character

```swift
func firstRepeatingCharacter(_ text: String) -> Character? {
    var seen = Set<Character>()

    for character in text {
        if !seen.insert(character).inserted {
            return character
        }
    }

    return nil
}
```

## 10. Remove Duplicate Characters Preserving Order

```swift
func removeDuplicateCharacters(_ text: String) -> String {
    var seen = Set<Character>()
    var result = ""

    for character in text {
        if seen.insert(character).inserted {
            result.append(character)
        }
    }

    return result
}
```

Example:

```swift
removeDuplicateCharacters("banana") // "ban"
```

## 11. Check Anagram

```swift
func areAnagrams(_ first: String, _ second: String) -> Bool {
    guard first.count == second.count else {
        return false
    }

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

Case-insensitive version:

```swift
func areCaseInsensitiveAnagrams(_ first: String, _ second: String) -> Bool {
    areAnagrams(first.lowercased(), second.lowercased())
}
```

## 12. Reverse Words In Sentence

```swift
func reverseWords(_ sentence: String) -> String {
    sentence
        .split(whereSeparator: { $0.isWhitespace })
        .reversed()
        .joined(separator: " ")
}
```

Example:

```swift
reverseWords("Swift is powerful") // "powerful is Swift"
```

## 13. Reverse Each Word

```swift
func reverseEachWord(_ sentence: String) -> String {
    sentence
        .split(whereSeparator: { $0.isWhitespace })
        .map { String($0.reversed()) }
        .joined(separator: " ")
}
```

Example:

```swift
reverseEachWord("Swift is powerful") // "tfiwS si lufrewop"
```

## 14. Find Longest Word

```swift
func longestWord(_ sentence: String) -> String? {
    sentence
        .split(whereSeparator: { $0.isWhitespace })
        .map(String.init)
        .max { $0.count < $1.count }
}
```

## 15. Check If String Contains Only Digits

```swift
func containsOnlyDigits(_ text: String) -> Bool {
    !text.isEmpty && text.allSatisfy { $0.isNumber }
}
```

Senior note: clarify whether Unicode digits are allowed. `isNumber` can match more than ASCII `0...9`.

## 16. Remove Spaces

```swift
func removingSpaces(_ text: String) -> String {
    text.filter { !$0.isWhitespace }
}
```

## 17. Capitalize First Letter Of Each Word

```swift
func titleCasedWords(_ sentence: String) -> String {
    sentence
        .split(whereSeparator: { $0.isWhitespace })
        .map { word in
            let lowercased = word.lowercased()
            guard let first = lowercased.first else { return "" }
            return first.uppercased() + lowercased.dropFirst()
        }
        .joined(separator: " ")
}
```

## 18. String Compression

```swift
func compressed(_ text: String) -> String {
    guard let first = text.first else {
        return ""
    }

    var result = ""
    var current = first
    var count = 0

    for character in text {
        if character == current {
            count += 1
        } else {
            result += "\(current)\(count)"
            current = character
            count = 1
        }
    }

    result += "\(current)\(count)"
    return result
}
```

Example:

```swift
compressed("aaabbc") // "a3b2c1"
```

## 19. Longest Common Prefix

```swift
func longestCommonPrefix(_ words: [String]) -> String {
    guard var prefix = words.first else {
        return ""
    }

    for word in words.dropFirst() {
        while !word.hasPrefix(prefix) {
            prefix.removeLast()

            if prefix.isEmpty {
                return ""
            }
        }
    }

    return prefix
}
```

## 20. Check If One String Is Rotation Of Another

```swift
func isRotation(_ first: String, of second: String) -> Bool {
    guard first.count == second.count else {
        return false
    }

    return (second + second).contains(first)
}
```

Example:

```swift
isRotation("erbottlewat", of: "waterbottle") // true
```

## Basic String Interview Checklist

- Empty string.
- One-character string.
- Case sensitivity.
- Spaces and punctuation.
- Unicode behavior.
- Whether order should be preserved.
- Whether comparison is ASCII-only or user-facing localized text.
- Time and space complexity.

## Common Beginner Mistakes

- Treating Swift `String` like an array with integer indexes.
- Forgetting empty string cases.
- Ignoring case sensitivity.
- Using only `" "` split and failing on multiple spaces.
- Not clarifying whether punctuation should be ignored.
- Repeatedly prepending strings in loops for large inputs.

