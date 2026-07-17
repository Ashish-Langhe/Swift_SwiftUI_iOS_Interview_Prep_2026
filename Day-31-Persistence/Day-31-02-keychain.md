# Day 31: Keychain

## What Keychain Is

Keychain Services securely stores small secrets on behalf of the user.

Use Keychain for:

- Auth tokens
- Refresh tokens
- Passwords
- Private keys
- Certificates
- Short sensitive values

Apple describes Keychain as encrypted storage for small bits of user data and secrets.

## Keychain vs UserDefaults

UserDefaults:

- Nonsensitive preferences
- Unencrypted on disk
- Simple key-value settings

Keychain:

- Sensitive secrets
- Encrypted storage
- Access controlled by device/app security

Rule:

```text
Preference -> UserDefaults
Secret -> Keychain
```

## Keychain Wrapper

The Keychain C API is verbose. Wrap it.

```swift
enum KeychainError: Error {
    case itemNotFound
    case invalidData
    case unhandledStatus(OSStatus)
}

protocol KeychainStoring {
    func save(_ data: Data, service: String, account: String) throws
    func read(service: String, account: String) throws -> Data
    func delete(service: String, account: String) throws
}
```

Implementation sketch:

```swift
final class KeychainStore: KeychainStoring {
    func save(_ data: Data, service: String, account: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]

        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)

        guard status == errSecSuccess else {
            throw KeychainError.unhandledStatus(status)
        }
    }
}
```

## Reading

```swift
func read(service: String, account: String) throws -> Data {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecReturnData as String: true,
        kSecMatchLimit as String: kSecMatchLimitOne
    ]

    var item: CFTypeRef?
    let status = SecItemCopyMatching(query as CFDictionary, &item)

    guard status != errSecItemNotFound else {
        throw KeychainError.itemNotFound
    }

    guard status == errSecSuccess else {
        throw KeychainError.unhandledStatus(status)
    }

    guard let data = item as? Data else {
        throw KeychainError.invalidData
    }

    return data
}
```

## Token Store

```swift
struct TokenStore {
    private let keychain: KeychainStoring
    private let service = "com.example.app.auth"
    private let account = "refreshToken"

    init(keychain: KeychainStoring) {
        self.keychain = keychain
    }

    func saveRefreshToken(_ token: String) throws {
        try keychain.save(Data(token.utf8), service: service, account: account)
    }

    func refreshToken() throws -> String {
        let data = try keychain.read(service: service, account: account)

        guard let token = String(data: data, encoding: .utf8) else {
            throw KeychainError.invalidData
        }

        return token
    }
}
```

## Accessibility

Keychain items can specify accessibility with `kSecAttrAccessible`.

Common choices:

- `kSecAttrAccessibleWhenUnlocked`
- `kSecAttrAccessibleAfterFirstUnlock`
- `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly`
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`

Senior decision:

- Need background access? Consider after-first-unlock.
- Need stronger local-only protection? Consider this-device-only/passcode options.
- Need iCloud sync? Avoid this-device-only.

## Access Groups

Use Keychain Access Groups when sharing secrets between app and extensions.

Example use cases:

- App + widget
- App + share extension
- App suite

This requires entitlements and careful security review.

## Testing

Use protocol abstraction.

```swift
final class InMemoryKeychainStore: KeychainStoring {
    private var storage: [String: Data] = [:]

    func save(_ data: Data, service: String, account: String) throws {
        storage["\(service):\(account)"] = data
    }

    func read(service: String, account: String) throws -> Data {
        guard let data = storage["\(service):\(account)"] else {
            throw KeychainError.itemNotFound
        }
        return data
    }

    func delete(service: String, account: String) throws {
        storage.removeValue(forKey: "\(service):\(account)")
    }
}
```

## Common Mistakes

- Storing secrets in UserDefaults.
- Storing large objects in Keychain.
- No wrapper around C API.
- Ignoring accessibility level.
- Not handling `errSecItemNotFound`.
- Not considering extensions/access groups.
- Tests using real Keychain without cleanup.

## Senior iOS Engineer Perspective

Ask:

- Is this value truly secret?
- What accessibility level is needed?
- Should it sync or stay device-only?
- Does an extension need access?
- How will tests avoid real Keychain dependency?
- What happens on logout?

## Interview Notes

Junior:

Keychain stores sensitive secrets securely.

Mid-level:

Use Keychain for tokens/passwords and wrap it behind a protocol.

Senior:

I design Keychain storage around threat model, accessibility, sync behavior, app extensions, logout cleanup, and test isolation.

## Practice

1. Build a `TokenStore` using Keychain.
2. Add an in-memory fake for tests.
3. Compare `WhenUnlocked` and `AfterFirstUnlock`.
4. Explain Keychain access groups.
