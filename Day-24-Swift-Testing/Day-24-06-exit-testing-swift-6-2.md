# Day 24: Exit Testing From Swift 6.2

## What Exit Testing Is

Swift 6.2 added exit testing to Swift Testing.

Exit testing verifies code terminates under specific conditions.

This is useful for:

- `preconditionFailure`
- `fatalError`
- critical invalid states
- command-line tools
- safety checks

Exit tests run in a separate process and validate exit behavior.

## Why Exit Testing Matters

Some code is supposed to terminate.

Example:

```swift
func requireValidConfiguration(_ config: AppConfig) {
    precondition(!config.apiKey.isEmpty, "Missing API key")
}
```

Before exit testing, verifying this behavior was difficult.

## Real iOS Use Case

Production iOS app code should rarely terminate casually. But libraries and tools may use preconditions for programmer errors.

Examples:

- Invalid dependency graph
- Impossible state in a reducer
- Required configuration missing in debug builds
- CLI tool invalid invocation

## Senior iOS Engineer Artifact

```text
Artifact: Exit Test Justification
Code path:
Why termination is correct:
Failure type: precondition/fatalError/exit
Process isolation:
Expected message:
Alternative recovery:
Production risk:
```

Senior lens:

- Exit behavior should be reserved for programmer errors or impossible states.
- User-recoverable errors should not terminate an app.
- Exit tests document intentional crash/termination contracts.
- Critical failure paths deserve evidence.

## More Coding Examples

### Example 1: Conceptual Exit Test

```swift
@Test
func missingConfiguration_exits() async throws {
    // Conceptual shape:
    // await #expect(exitsWith: .failure) {
    //     requireValidConfiguration(.missingAPIKey)
    // }
}
```

Exact APIs may vary by toolchain, but the purpose is to validate termination behavior in a separate process.

### Example 2: Prefer Throwing For User Errors

```swift
func validateEmail(_ email: String) throws {
    guard email.contains("@") else {
        throw ValidationError.invalidEmail
    }
}
```

Do not use exit behavior for normal user input.

## Common Mistakes

- Testing user-recoverable errors with exit tests.
- Using fatal errors for network or validation failures.
- Not documenting why termination is correct.
- Adding exit tests without checking message/condition.
- Treating exit testing as a replacement for error handling.

## Interview Guide

Junior:

Exit testing checks code that is expected to terminate.

Mid-level:

Use it for preconditions and critical programmer-error paths, not normal user errors.

Senior:

Exit tests document intentional termination contracts and should be used sparingly for impossible states or safety-critical tooling behavior.

## Practice

1. Identify one code path that should throw, not exit.
2. Identify one precondition that could deserve an exit test.
3. Explain why exit tests run separately.
4. Write an exit-test justification artifact.
