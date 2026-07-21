# Day 35: Secrets Handling

## What Secrets Are

Secrets include:

- API keys
- Auth tokens
- Refresh tokens
- Private keys
- Client secrets
- Encryption keys
- Signing secrets
- Passwords

Some values are public identifiers, not true secrets. Many mobile "API keys" are not secret because they ship inside the app binary.

## Rule: Mobile Apps Cannot Hide Static Secrets

Anything bundled in the app can potentially be extracted.

Do not embed:

- Backend client secret
- Private signing key
- Admin API key
- Payment secret key
- Database root credential

Use server-side token exchange instead.

## Bad Example

```swift
enum Secrets {
    static let paymentSecret = "sk_live_..."
}
```

This can be extracted from the binary.

## Better Design

```text
iOS app -> your backend -> payment provider
```

The backend stores true secrets. The app receives short-lived scoped tokens when needed.

## Build Configuration Secrets

Use build settings or `.xcconfig` for non-secret configuration:

```text
API_BASE_URL = https://api.example.com
```

This is configuration, not secret.

## Runtime Tokens

Store runtime tokens in Keychain.

```swift
protocol TokenStoring {
    func save(_ session: AuthSession) throws
    func session() throws -> AuthSession
    func clear() throws
}
```

Clear on logout.

## Logging Redaction

```swift
struct Redactor {
    func headers(_ headers: [String: String]) -> [String: String] {
        headers.mapValues { _ in "<redacted>" }
    }
}
```

Always redact:

- Authorization
- Cookie
- Set-Cookie
- X-API-Key
- Password fields
- Card details

## Source Control

Never commit:

- `.env` with secrets
- Production certificates/private keys
- Provisioning private materials
- Service account keys
- Exported Keychain dumps

Use secret scanning in CI where possible.

## CI/CD Secrets

Store CI secrets in secure secret storage:

- GitHub Actions secrets
- Xcode Cloud environment secrets
- Fastlane match repository with encryption
- Cloud provider secret managers

Restrict access and rotate periodically.

## Rotation

Every secret needs a rotation plan.

```text
Who owns it?
Where is it stored?
Where is it used?
How is it rotated?
What breaks during rotation?
How is old value revoked?
```

## Common Mistakes

- Believing app-bundled keys are secret.
- Logging tokens.
- Long-lived tokens with broad scope.
- Secrets in source control.
- No rotation plan.
- No logout cleanup.
- Backend trusts mobile app static key as identity.

## Senior Artifact

```text
Artifact: Secrets Handling Review
Secret:
Is it truly secret?
Where generated:
Where stored:
Lifetime:
Scope:
Rotation:
Revocation:
Logging risk:
CI/CD exposure:
Mobile extraction risk:
```

## Interview Notes

Junior:

Do not hardcode passwords or secret keys in the app.

Mid-level:

Store runtime tokens in Keychain and keep real backend secrets on the server.

Senior:

I assume app binaries can be inspected. I avoid static secrets in mobile apps, use scoped short-lived tokens, redact logs, secure CI secrets, and design rotation/revocation.

## Practice

1. Identify which values in an app are real secrets.
2. Replace hardcoded payment secret with backend token flow.
3. Add log redaction.
4. Write a rotation plan.
