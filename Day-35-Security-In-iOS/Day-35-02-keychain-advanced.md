# Day 35: Keychain Advanced

## What Keychain Is

Keychain Services securely stores small secrets.

Use Keychain for:

- Access tokens
- Refresh tokens
- Passwords
- Private keys
- Certificates
- Short sensitive values

Do not use Keychain for large files or general app data.

## Keychain Item Anatomy

Common attributes:

```swift
kSecClass
kSecAttrService
kSecAttrAccount
kSecValueData
kSecAttrAccessible
kSecAttrAccessGroup
kSecAttrAccessControl
```

Example save:

```swift
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrService as String: "com.example.auth",
    kSecAttrAccount as String: "refreshToken",
    kSecValueData as String: Data(token.utf8),
    kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
]

SecItemDelete(query as CFDictionary)
let status = SecItemAdd(query as CFDictionary, nil)
```

## Accessibility Classes

Common choices:

```text
kSecAttrAccessibleWhenUnlocked:
Available only while device is unlocked.

kSecAttrAccessibleAfterFirstUnlock:
Available after first unlock after boot. Useful for some background access.

kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly:
Requires device passcode and does not migrate to another device.

kSecAttrAccessibleWhenUnlockedThisDeviceOnly:
Available while unlocked and does not migrate to another device.
```

Choose based on security and background needs.

## ThisDeviceOnly

`ThisDeviceOnly` prevents migration through encrypted backups.

Use for:

- Device-bound sessions
- High-security tokens
- Enterprise secrets

Tradeoff:

- User may need to sign in again after device migration.

## Access Control With Biometrics

```swift
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    .biometryCurrentSet,
    nil
)!
```

Store:

```swift
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrService as String: service,
    kSecAttrAccount as String: account,
    kSecValueData as String: data,
    kSecAttrAccessControl as String: accessControl
]
```

`.biometryCurrentSet` invalidates access if enrolled biometrics change.

## Keychain Access Groups

Use access groups to share Keychain items with:

- App extensions
- Widgets
- Share extensions
- App suite targets

Requires entitlements.

Senior warning: access groups widen access. Keep them minimal.

## Wrapper Protocol

```swift
protocol SecureTokenStoring {
    func save(_ session: AuthSession) throws
    func session() throws -> AuthSession
    func clear() throws
}
```

Feature code should not use `SecItemCopyMatching` directly.

## Logout Cleanup

```swift
func clear() throws {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service
    ]

    let status = SecItemDelete(query as CFDictionary)

    guard status == errSecSuccess || status == errSecItemNotFound else {
        throw KeychainError.unhandledStatus(status)
    }
}
```

Logout must clear secrets, cached personal data, and session state.

## Testing

Use fake stores:

```swift
final class InMemorySecureTokenStore: SecureTokenStoring {
    var storedSession: AuthSession?

    func save(_ session: AuthSession) {
        storedSession = session
    }

    func session() throws -> AuthSession {
        guard let storedSession else {
            throw KeychainError.itemNotFound
        }
        return storedSession
    }

    func clear() {
        storedSession = nil
    }
}
```

Avoid using real Keychain in most unit tests.

## Common Mistakes

- Storing tokens in UserDefaults.
- Not choosing accessibility deliberately.
- No logout cleanup.
- Access group too broad.
- Tests depend on real Keychain state.
- Ignoring `errSecItemNotFound`.
- Storing large encrypted blobs in Keychain.

## Senior iOS Engineer Perspective

Ask:

- What is the secret?
- Should it migrate to new devices?
- Does background access need it?
- Should biometric enrollment changes invalidate it?
- Which targets need access?
- How is logout handled?
- How are tests isolated?

## Interview Notes

Junior:

Keychain stores sensitive small secrets securely.

Mid-level:

Use accessibility classes, access groups, wrappers, and clear secrets on logout.

Senior:

I design Keychain use around threat model, accessibility, migration, biometric access control, extension access, logout cleanup, and test isolation.

## Practice

1. Store a refresh token using `ThisDeviceOnly`.
2. Add biometric access control to a Keychain item.
3. Design logout cleanup.
4. Explain access groups.
