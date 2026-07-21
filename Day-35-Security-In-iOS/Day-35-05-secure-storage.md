# Day 35: Secure Storage

## What Secure Storage Means

Secure storage means choosing the correct storage based on sensitivity, size, lifetime, and threat model.

Storage is not secure just because it is local.

## Storage Decision Table

```text
Theme preference:
UserDefaults

Access token:
Keychain

Refresh token:
Keychain with deliberate accessibility

Large encrypted document:
FileManager + encryption key protected by Keychain

Image cache:
Caches directory, no secrets

User notes with sensitive content:
Encrypted file/database, key management required

Biometric-gated secret:
Keychain + access control
```

## UserDefaults Is Not Secure

Do not store:

- Tokens
- Passwords
- PINs
- Secret API keys
- Sensitive user data

UserDefaults is for preferences.

## Keychain For Small Secrets

```swift
protocol SecretStore {
    func save(_ value: Data, key: String) throws
    func read(key: String) throws -> Data
    func delete(key: String) throws
}
```

Use Keychain for small sensitive values.

## Encrypted Files

For large sensitive files:

```text
1. Generate symmetric encryption key
2. Store key in Keychain
3. Encrypt file data
4. Save encrypted file with FileManager
5. Clear plaintext buffers as soon as possible
```

Conceptual API:

```swift
protocol SecureFileStore {
    func save(_ data: Data, name: String) async throws
    func read(name: String) async throws -> Data
    func delete(name: String) async throws
}
```

## File Protection

iOS file protection can restrict file access based on device lock state.

```swift
try FileManager.default.setAttributes(
    [.protectionKey: FileProtectionType.complete],
    ofItemAtPath: fileURL.path
)
```

Common protection levels:

- `.complete`
- `.completeUnlessOpen`
- `.completeUntilFirstUserAuthentication`
- `.none`

Choose based on background access needs.

## Caches And Temporary Files

Sensitive temporary files are easy to forget.

Risk points:

- Exported PDFs
- Uploaded image temp files
- Crash logs
- Debug logs
- Downloaded attachments

Clean up:

```swift
try? FileManager.default.removeItem(at: temporaryURL)
```

## Logging Secrets

Do not log:

- Authorization headers
- Tokens
- Passwords
- Full request bodies with PII
- Payment details
- Keychain values

Use redaction:

```swift
logger.info("Request headers: \(redactedHeaders)")
```

## Clipboard Risk

Clipboard can expose copied values to other contexts.

Avoid copying:

- OTP codes after use
- Passwords
- Tokens
- Private keys

If your app lets users copy sensitive values, consider expiration/clear behavior and user education.

## Common Mistakes

- Token in UserDefaults.
- Sensitive cache files unencrypted.
- Plaintext temp files not deleted.
- Debug logs with secrets.
- File protection ignored.
- Keychain used for large blobs.
- No logout cleanup.

## Senior Artifact

```text
Artifact: Secure Storage Review
Data:
Sensitivity:
Size:
Lifetime:
Storage:
Encryption:
Key management:
File protection:
Backup/sync:
Logout cleanup:
Logging risk:
Tests:
```

## Interview Notes

Junior:

Sensitive data should not be stored in UserDefaults.

Mid-level:

Use Keychain for small secrets and file protection/encryption for sensitive files.

Senior:

I classify data by sensitivity, size, lifetime, backup/sync behavior, and threat model. I secure keys, files, logs, temp data, and logout cleanup as one storage design.

## Practice

1. Classify app data into storage choices.
2. Add file protection to a saved document.
3. Design encrypted file storage with Keychain key.
4. Review logs for accidental secrets.
