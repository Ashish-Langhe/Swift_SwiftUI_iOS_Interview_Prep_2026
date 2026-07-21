# Day 35: Biometric Auth

## What Biometric Auth Is

iOS biometric authentication uses the LocalAuthentication framework.

It supports:

- Face ID
- Touch ID
- Optic ID
- Device passcode fallback

Your app never receives biometric data. The Secure Enclave manages biometric secrets outside your app.

## Basic LocalAuthentication

```swift
import LocalAuthentication

func authenticate() async throws -> Bool {
    let context = LAContext()
    var error: NSError?

    guard context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) else {
        throw AuthError.biometryUnavailable
    }

    return try await context.evaluatePolicy(
        .deviceOwnerAuthentication,
        localizedReason: "Unlock your account"
    )
}
```

Use `.deviceOwnerAuthentication` when passcode fallback is acceptable.

Use `.deviceOwnerAuthenticationWithBiometrics` when you specifically require biometric only.

## Biometric Auth Is Not Login

Biometric auth proves:

```text
The current device owner authenticated locally.
```

It does not prove:

```text
The backend session is valid.
The user's server account password is correct.
The device is not compromised.
```

Use biometrics to unlock local access or release Keychain secrets.

## Protecting Keychain Items

Apple supports protecting Keychain items with Face ID/Touch ID through Security + LocalAuthentication.

Use access control:

```swift
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    .biometryCurrentSet,
    nil
)!
```

This can require biometric authentication each time the item is read.

## LAContext Reuse

You can reuse `LAContext` carefully to avoid repeated prompts for related operations.

But avoid storing authenticated state forever.

Senior design:

- Short unlock window
- Revalidate for sensitive operations
- Invalidate on app background if needed

## Biometry Change

`.biometryCurrentSet` invalidates access if enrolled biometrics change.

Use for high-security secrets.

Tradeoff:

- User may need to sign in again after adding/removing Face ID/Touch ID.

## UX Copy

```swift
localizedReason: "Use Face ID to view your saved cards"
```

Good reason:

- Specific
- User-facing
- Explains protected action

Weak:

```swift
"Authenticate"
```

## Error Handling

Common cases:

- User canceled
- System canceled
- Biometry unavailable
- Biometry lockout
- Passcode not set
- Authentication failed

Map to user-safe UI:

```swift
switch error {
case LAError.userCancel:
    return
case LAError.biometryLockout:
    showPasscodeFallback()
default:
    showUnableToAuthenticate()
}
```

## Common Mistakes

- Treating biometric auth as server login.
- Storing a boolean like `isBiometricAuthenticated = true` permanently.
- No fallback flow.
- Vague localized reason.
- Not handling biometry enrollment change.
- Storing secrets outside Keychain after biometric unlock.

## Senior Artifact

```text
Artifact: Biometric Auth Review
Protected action:
Policy:
Passcode fallback:
Keychain access control:
Biometry change behavior:
Session/backend relationship:
Background invalidation:
Error handling:
UX copy:
Tests:
```

## Interview Notes

Junior:

LocalAuthentication lets the app ask the user to authenticate with Face ID, Touch ID, Optic ID, or passcode.

Mid-level:

Use biometrics to unlock local secrets, handle fallback/errors, and provide clear localized reason text.

Senior:

I separate local biometric authentication from server authentication, use Keychain access control for secrets, handle biometry changes and lockout, and design short-lived unlock behavior.

## Practice

1. Add Face ID prompt using `LAContext`.
2. Protect a Keychain token with biometric access control.
3. Handle user cancel and biometry lockout.
4. Explain why biometrics are not backend login.
