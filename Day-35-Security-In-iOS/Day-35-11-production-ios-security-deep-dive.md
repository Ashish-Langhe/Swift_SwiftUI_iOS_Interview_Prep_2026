# Day 35: Production iOS Security Deep Dive

Security is not just about hiding data. In production iOS apps, security means protecting user trust, reducing fraud, preventing account takeover, avoiding sensitive data leaks, and designing systems that remain safe even when the device, network, app binary, or backend response is under attack.

This file is a focused iOS security preparation guide for junior to senior interviews. It covers practical Apple frameworks, mobile attack surfaces, secure coding decisions, production banking flow, and the kind of tradeoffs senior iOS engineers are expected to explain clearly.

## Topics Covered

- Keychain Services vs UserDefaults
- App Transport Security
- HTTPS, TLS, and SSL pinning
- Certificate pinning vs public key pinning
- Face ID and Touch ID with LocalAuthentication
- Secure Enclave architecture
- JWT authentication and token security
- CryptoKit with SHA256 and AES-GCM
- Secure networking with URLSession
- Jailbreak detection
- Reverse engineering protection
- Secure coding best practices
- OWASP Mobile Top 10 2024
- Production banking app security flow
- Top iOS security interview questions and answers
- MASVS security mindset
- Secure logging and analytics
- Dependency and supply-chain security
- Deep link and URL scheme security
- App Attest and DeviceCheck overview

## 1. Security Mindset For iOS Developers

A mobile app is a hostile environment compared with backend code. The app binary runs on a device the user controls. Attackers can inspect traffic, reverse engineer binaries, patch runtime behavior, hook methods, inspect local storage, or run the app on compromised devices.

Senior principle:

```text
The client can improve security and user experience, but the server must remain the final authority for sensitive business decisions.
```

The iOS app should protect:

- Authentication tokens.
- Personal data.
- Financial data.
- Health data.
- Private messages.
- Session state.
- API integrity.
- App configuration.
- Local files.
- Logs and analytics.

The iOS app should not be the only authority for:

- Payment approval.
- Banking transfer approval.
- Authorization rules.
- Subscription entitlement.
- Inventory reservation.
- Fraud decisions.
- Admin access.

## 2. Keychain Services Vs UserDefaults

`UserDefaults` is for small, non-sensitive preferences. It is not secure storage.

Use `UserDefaults` for:

- Theme preference.
- Last selected tab.
- Onboarding completed flag.
- Feature preference.
- Non-sensitive cached settings.

Do not use `UserDefaults` for:

- Access tokens.
- Refresh tokens.
- Passwords.
- PINs.
- Private keys.
- OTPs.
- Payment data.

Use Keychain for:

- Refresh tokens.
- Session secrets.
- Device-bound credentials.
- Symmetric encryption keys.
- Small sensitive values.

Comparison:

| Area | UserDefaults | Keychain |
| --- | --- | --- |
| Purpose | Preferences | Secrets |
| Encryption | Not meant as secure vault | Protected by iOS security services |
| Survives reinstall | Usually no | Can survive depending access group/configuration |
| Best for | Small settings | Tokens, credentials, keys |
| Interview answer | Not secure | Correct default for sensitive secrets |

Simple Keychain wrapper shape:

```swift
protocol SecureTokenStore {
    func saveRefreshToken(_ token: String) throws
    func refreshToken() throws -> String?
    func deleteRefreshToken() throws
}
```

Senior guidance:

```text
I would keep access tokens short-lived in memory where possible and store refresh tokens in Keychain. On logout, I would clear tokens, sensitive caches, and revoke the refresh token server-side if supported.
```

## 3. Keychain Accessibility Choices

Keychain access level changes security behavior.

Common choices:

- `kSecAttrAccessibleWhenUnlocked`: available only when device is unlocked.
- `kSecAttrAccessibleAfterFirstUnlock`: available after first unlock, useful for background tasks.
- `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly`: stronger local protection, not migrated to another device.

Senior decision:

```text
For banking or high-risk apps, I would prefer device-only accessibility for sensitive secrets when business requirements allow it, because it reduces migration and backup exposure.
```

Tradeoff:

- Stronger device-only settings improve security.
- They may affect restore, migration, and background behavior.

## 4. App Transport Security

App Transport Security, or ATS, is Apple's default policy that pushes apps toward secure network connections.

ATS expects:

- HTTPS instead of HTTP.
- Strong TLS configuration.
- Valid certificates.
- Modern cryptography.

Avoid broad exceptions like:

```xml
<key>NSAllowsArbitraryLoads</key>
<true/>
```

Better:

```text
If an exception is unavoidable for a legacy endpoint, scope it to a specific domain, document why it exists, and remove it as soon as possible.
```

Interview answer:

```text
ATS reduces insecure network communication by enforcing secure defaults. In production, I would avoid global ATS exceptions and require HTTPS for API and media endpoints.
```

## 5. HTTPS, TLS, And SSL Pinning

HTTPS protects data in transit using TLS. It provides encryption, server authentication, and integrity.

TLS protects against:

- Passive network sniffing.
- Basic man-in-the-middle attacks.
- Credential theft on open Wi-Fi.

TLS alone may not protect against:

- A malicious certificate authority.
- User-installed root certificates.
- Compromised trust stores.
- Corporate proxy interception.

That is where pinning may help for high-risk apps.

## 6. Certificate Pinning Vs Public Key Pinning

Pinning means the app validates more than normal certificate trust. It checks whether the server certificate or public key matches expected values.

### Certificate Pinning

You pin the exact certificate.

Pros:

- Strong binding to a known certificate.
- Simple to reason about.

Cons:

- Certificate rotation can break the app.
- Requires app update or remote pin set if certificate changes.

### Public Key Pinning

You pin the public key or subject public key info.

Pros:

- Survives certificate renewal if the same key pair is used.
- Usually more flexible than exact certificate pinning.

Cons:

- Still requires careful rotation planning.
- Key compromise is serious.

Senior answer:

```text
For banking, I would consider public key pinning with backup pins and an operational rotation plan. Pinning without backup and monitoring can create self-inflicted outages.
```

## 7. URLSession Pinning Example

Pinning is usually implemented through `URLSessionDelegate`.

Simplified shape:

```swift
final class PinnedSessionDelegate: NSObject, URLSessionDelegate {
    private let pinnedKeyHashes: Set<String>

    init(pinnedKeyHashes: Set<String>) {
        self.pinnedKeyHashes = pinnedKeyHashes
    }

    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge
    ) async -> (URLSession.AuthChallengeDisposition, URLCredential?) {
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust,
              let certificateChain = SecTrustCopyCertificateChain(serverTrust) as? [SecCertificate]
        else {
            return (.performDefaultHandling, nil)
        }

        let isPinned = certificateChain.contains { certificate in
            let keyHash = PublicKeyHasher.hashPublicKey(from: certificate)
            return pinnedKeyHashes.contains(keyHash)
        }

        if isPinned {
            return (.useCredential, URLCredential(trust: serverTrust))
        } else {
            return (.cancelAuthenticationChallenge, nil)
        }
    }
}
```

This is intentionally simplified. A production implementation needs tested key extraction, backup pins, observability, failure handling, and rotation planning.

## 8. Face ID And Touch ID With LocalAuthentication

Use `LocalAuthentication` for biometric authentication.

Common use cases:

- Unlock app.
- Re-authenticate before sensitive data display.
- Confirm sensitive local action.
- Step-up UX before server-side high-risk action.

Example:

```swift
import LocalAuthentication

final class BiometricAuthenticator {
    func authenticate(reason: String) async throws -> Bool {
        let context = LAContext()
        var error: NSError?

        guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
            throw BiometricError.notAvailable
        }

        return try await context.evaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            localizedReason: reason
        )
    }
}

enum BiometricError: Error {
    case notAvailable
}
```

Important:

- Biometrics prove local user presence.
- Biometrics do not replace server authorization.
- Always handle denied, unavailable, lockout, and fallback states.
- Do not show sensitive information before authentication succeeds.

Senior banking answer:

```text
For a transfer, biometrics can be used as a step-up local authentication factor, but the server must still validate session, device risk, account limits, beneficiary state, and transaction authorization.
```

## 9. Secure Enclave Architecture

Secure Enclave is a hardware-backed security component available on modern Apple devices. It protects sensitive cryptographic operations and biometric data.

Key ideas:

- Face ID and Touch ID biometric templates are protected by Secure Enclave.
- The app does not receive raw biometric data.
- Secure Enclave can protect keys so private key material does not leave protected hardware.
- Access Control flags can require user presence or biometrics before key usage.

Use cases:

- Device-bound private key.
- High-risk authentication.
- Signing a challenge.
- Protecting local encryption keys.

Senior explanation:

```text
I would not store biometric data. iOS handles biometric matching. The app requests authentication through LocalAuthentication and can protect keys with access control policies tied to user presence.
```

## 10. JWT Authentication And Token Security

JWT is a token format commonly used for API authentication. It often contains claims like subject, issuer, expiry, audience, and scopes.

Important JWT points:

- A JWT is usually signed, not encrypted.
- Do not store sensitive PII inside JWT payload unless encrypted.
- Always validate expiry server-side.
- Client can decode JWT for convenience, but should not trust it as final authority.
- Access tokens should be short-lived.
- Refresh tokens need stronger protection.

Token storage strategy:

```text
Access token: memory or Keychain depending app/session model
Refresh token: Keychain
Logout: clear local tokens and revoke refresh token server-side
```

Refresh flow:

```text
API returns 401 -> Auth manager refreshes token -> retry original request once -> fail/logout if refresh fails
```

Avoid refresh stampede:

```swift
actor AuthTokenManager {
    private var refreshTask: Task<AuthTokens, Error>?

    func validTokens() async throws -> AuthTokens {
        if let refreshTask {
            return try await refreshTask.value
        }

        if let current = cachedTokens, current.isValid {
            return current
        }

        let task = Task { try await refreshTokensFromServer() }
        refreshTask = task
        defer { refreshTask = nil }
        return try await task.value
    }

    private var cachedTokens: AuthTokens?

    private func refreshTokensFromServer() async throws -> AuthTokens {
        // Call refresh endpoint and persist new refresh token securely.
        throw AuthError.notImplemented
    }
}
```

Senior point:

```text
Refresh token rotation and single-flight refresh protect against race conditions and reduce accidental account logout storms.
```

## 11. CryptoKit: SHA256

Use SHA256 for hashing, not encryption.

Good use cases:

- Integrity checks.
- Public key hash comparison.
- File checksum.
- Non-secret identifiers with salt where appropriate.

Do not use plain SHA256 for:

- Password storage.
- Reversible encryption.
- Secure token generation.

Example:

```swift
import CryptoKit
import Foundation

func sha256Hex(_ data: Data) -> String {
    let digest = SHA256.hash(data: data)
    return digest.map { String(format: "%02x", $0) }.joined()
}
```

Password note:

```text
Password hashing belongs on the server and should use dedicated password hashing algorithms such as Argon2id, bcrypt, or scrypt, not plain SHA256.
```

## 12. CryptoKit: AES-GCM Encryption

Use symmetric encryption when you must protect local data before storing it.

Example:

```swift
import CryptoKit
import Foundation

struct LocalEncryptor {
    let key: SymmetricKey

    func encrypt(_ data: Data) throws -> Data {
        let sealedBox = try AES.GCM.seal(data, using: key)
        guard let combined = sealedBox.combined else {
            throw CryptoError.encryptionFailed
        }
        return combined
    }

    func decrypt(_ data: Data) throws -> Data {
        let box = try AES.GCM.SealedBox(combined: data)
        return try AES.GCM.open(box, using: key)
    }
}

enum CryptoError: Error {
    case encryptionFailed
}
```

Key storage:

- Do not hardcode encryption keys.
- Store symmetric keys in Keychain.
- For high-risk apps, consider hardware-backed key protection.
- Rotate keys only with a clear migration plan.

Senior warning:

```text
Encryption is only as strong as key management. Hardcoded keys inside the app binary are recoverable by reverse engineering.
```

## 13. Secure Networking With URLSession

A secure networking layer should handle:

- HTTPS only.
- ATS compliance.
- Auth headers.
- Token refresh.
- TLS/pinning if required.
- Request timeout.
- Retry policy.
- Response validation.
- Typed errors.
- Request IDs.
- Sensitive log redaction.

Example request builder:

```swift
struct SecureEndpoint<Response: Decodable> {
    let path: String
    let method: HTTPMethod
    let requiresAuth: Bool
    let body: Data?
}

enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case delete = "DELETE"
}
```

Do not log:

- Authorization headers.
- Cookies.
- OTP.
- Card numbers.
- Full request bodies with PII.
- Raw banking transaction details.

## 14. Jailbreak Detection

Jailbreak detection attempts to identify compromised devices.

Common signals:

- Suspicious file paths.
- Ability to write outside sandbox.
- Suspicious URL schemes.
- Debugger or dynamic library injection signs.
- Runtime tampering indicators.

Important:

- Jailbreak detection can be bypassed.
- It should be treated as a risk signal, not a perfect guarantee.
- Avoid relying on it as the only protection.
- Server-side risk scoring is stronger.

Senior answer:

```text
For banking, jailbreak detection can increase risk score, limit high-risk actions, require step-up authentication, or block certain flows. I would not treat client-side jailbreak detection as unbypassable.
```

## 15. Reverse Engineering Protection

Attackers can inspect the app binary, symbols, strings, API endpoints, and runtime behavior.

Defenses:

- Avoid hardcoded secrets.
- Strip debug symbols for release.
- Use compiler optimizations.
- Obfuscate only where justified.
- Detect debugger/tampering as risk signals.
- Keep business-critical decisions server-side.
- Use App Attest or DeviceCheck where appropriate.
- Use certificate/public key pinning for high-risk APIs.

What not to promise:

```text
No iOS app can be made impossible to reverse engineer. The goal is to reduce useful secrets in the binary and move critical trust decisions to the server.
```

## 16. Secure Coding Best Practices

Practical checklist:

- Validate inputs from API, deep links, push payloads, and pasteboard.
- Treat server responses as untrusted until decoded and validated.
- Avoid force-unwrapping security-sensitive data.
- Avoid logging secrets.
- Use least privilege permissions.
- Keep dependencies updated.
- Avoid broad ATS exceptions.
- Use secure random APIs for tokens/nonces.
- Clear sensitive data on logout.
- Avoid putting secrets in Info.plist.
- Avoid shipping debug menus in production.
- Protect WebViews if used.

Example secure random:

```swift
import Security
import Foundation

func secureRandomBytes(count: Int) throws -> Data {
    var data = Data(count: count)
    let result = data.withUnsafeMutableBytes {
        SecRandomCopyBytes(kSecRandomDefault, count, $0.baseAddress!)
    }
    guard result == errSecSuccess else {
        throw RandomError.failed
    }
    return data
}

enum RandomError: Error {
    case failed
}
```

## 17. OWASP Mobile Top 10 2024

The official OWASP Mobile Top 10 2024 categories are:

| ID | Risk | iOS Example |
| --- | --- | --- |
| M1 | Improper Credential Usage | Tokens in UserDefaults, hardcoded API keys |
| M2 | Inadequate Supply Chain Security | Unverified third-party SDK, compromised dependency |
| M3 | Insecure Authentication/Authorization | Client-only authorization checks |
| M4 | Insufficient Input/Output Validation | Unsafe deep link parameters, trusting push payloads |
| M5 | Insecure Communication | HTTP, weak TLS, missing validation |
| M6 | Inadequate Privacy Controls | Logging PII, excessive tracking |
| M7 | Insufficient Binary Protections | Easy tampering, no reverse engineering resistance |
| M8 | Security Misconfiguration | ATS exceptions, debug flags in release |
| M9 | Insecure Data Storage | Sensitive data in files/UserDefaults |
| M10 | Insufficient Cryptography | Hardcoded keys, weak algorithms, poor key management |

Senior interview line:

```text
I would use OWASP Mobile Top 10 and MASVS as review frameworks: storage, crypto, auth, network, platform interaction, code quality, resilience, and privacy.
```

## 18. MASVS Mapping For iOS

OWASP MASVS groups mobile security controls into major areas:

- Storage: protect sensitive data at rest.
- Crypto: use cryptography correctly.
- Auth: authentication and authorization.
- Network: secure data in transit.
- Platform: safe use of OS features.
- Code: secure coding and updates.
- Resilience: tamper and reverse engineering resistance.
- Privacy: protect user privacy.

Interview answer:

```text
For a senior security review, I would map the app against MASVS categories so we do not only focus on tokens and pinning while missing privacy, platform, dependency, and binary-resilience risks.
```

## 19. Secure Logging And Analytics

Logs are a common source of data leakage.

Safe to log:

- Endpoint family, not full sensitive URL.
- Status code.
- Error family.
- Duration.
- Request ID.
- App version.
- OS version.
- Feature flag state.

Avoid logging:

- Authorization headers.
- Cookies.
- Tokens.
- OTP.
- PIN.
- Full card number.
- Raw private messages.
- Exact address/location unless required and consented.

Example:

```swift
struct NetworkLogEvent {
    let endpointName: String
    let statusCode: Int
    let durationMS: Int
    let errorFamily: String?
    let requestID: String?
}
```

Senior point:

```text
Security observability should help debug production issues without creating a privacy incident.
```

## 20. Dependency And Supply-Chain Security

Modern iOS apps depend on Swift packages, CocoaPods, binary SDKs, analytics SDKs, payment SDKs, and internal frameworks.

Risks:

- Compromised dependency.
- Malicious post-install script.
- Abandoned package.
- Over-permissioned SDK.
- SDK collecting excessive data.
- Binary framework without review.

Controls:

- Pin dependency versions.
- Review dependency ownership and activity.
- Prefer trusted packages.
- Audit privacy manifests and SDK data collection.
- Use SPM package resolution files.
- Keep CI reproducible.
- Remove unused SDKs.
- Monitor CVEs and release notes.

Senior answer:

```text
Supply-chain security matters because third-party SDKs run inside the same app process and can access data, lifecycle events, and network behavior depending on integration.
```

## 21. Deep Link And URL Scheme Security

Deep links are external input.

Risks:

- Unauthorized navigation.
- Parameter injection.
- Open redirect style behavior.
- Triggering sensitive actions.
- URL scheme hijacking.
- Bypassing auth gates.

Safe design:

```text
Deep link -> Parse -> Validate -> Auth gate -> Permission check -> Fetch server state -> Navigate
```

Never:

- Execute money movement directly from a link.
- Trust user ID from a link.
- Skip auth because a URL has a token.
- Put long-lived secrets in links.

Senior answer:

```text
A deep link should route the user to a flow, not become authorization. The server still decides whether the user can access the target resource.
```

## 22. App Attest And DeviceCheck Overview

App Attest and DeviceCheck help backend systems reason about whether requests are likely coming from a legitimate app/device.

Use cases:

- Fraud reduction.
- Abuse prevention.
- Protecting high-risk APIs.
- Verifying app instances.
- Adding device risk signals.

Important:

- They complement authentication.
- They do not replace user authorization.
- They need backend integration.
- They should be part of layered defense.

Senior line:

```text
For high-risk flows, I would combine user auth, server-side risk checks, device attestation signals, and step-up verification rather than trusting only one control.
```

## 23. Production Banking App Security Flow

Prompt:

```text
Design security for a banking transfer flow.
```

Senior solution:

```text
The banking app should treat the server as authoritative. The iOS app stores refresh tokens in Keychain, uses HTTPS with ATS, may use public key pinning with backup pins, and requires local biometric unlock for sensitive screens. For transfer, the app collects input, validates basic format locally, requests server validation, shows verified beneficiary and fees, then requires step-up authentication such as biometrics plus OTP depending on risk. The final transfer is submitted with an idempotency key. The server checks limits, fraud, device risk, session validity, account state, and authorization. The app shows pending or confirmed only based on server response.
```

Flow:

```text
Login -> Token stored in Keychain
Open transfer -> Biometric local gate
Enter beneficiary/amount -> Local validation
Server validate -> Show risk/fees
Step-up auth -> OTP/biometric/device signal
Submit transfer with idempotency key
Server authorizes -> Pending/Success/Failure state
Receipt -> Secure display and share controls
```

iOS details:

- Keychain token storage.
- Session timeout.
- Screen hiding in app switcher if policy requires.
- No screenshots for sensitive views if using custom protection strategy.
- Redacted logs.
- No sensitive data in analytics.
- Server-authoritative balance.
- Clear sensitive cache on logout.
- Jailbreak/tamper signal can increase risk.
- App version and request ID included in telemetry.

State machine:

```swift
enum TransferState: Equatable {
    case enteringDetails
    case validating
    case review(TransferReview)
    case stepUpRequired(StepUpMethod)
    case submitting
    case pending(referenceID: String)
    case completed(receiptID: String)
    case failed(reason: String)
}
```

Tradeoff:

```text
Banking flows should optimize for correctness and trust over speed. We can make the UI responsive, but final success must come from server confirmation.
```

## 24. Secure Architecture Checklist

Use this when reviewing an app:

- Are tokens stored in Keychain?
- Are secrets absent from source code and app binary?
- Is ATS enabled without broad exceptions?
- Is pinning used only with rotation readiness?
- Is auth refresh single-flight and safe?
- Are sensitive APIs server-authoritative?
- Is local sensitive data minimized?
- Are logs redacted?
- Are analytics privacy-safe?
- Are deep links validated and auth-gated?
- Is user input validated?
- Are dependencies reviewed?
- Is jailbreak/tamper detection treated as a signal?
- Is there a kill switch for risky security rollout?
- Are old app versions supported safely?
- Are OWASP/MASVS controls considered?

## 25. Junior-Level Interview Questions And Answers

### What is the difference between Keychain and UserDefaults?

`UserDefaults` stores preferences and is not secure storage. Keychain is designed for secrets like tokens and credentials. For interview purposes, say: use UserDefaults for non-sensitive preferences and Keychain for sensitive values.

### What is ATS?

ATS is Apple's secure networking policy that encourages HTTPS and modern TLS. In production, avoid broad ATS exceptions.

### What is SSL pinning?

SSL pinning means the app validates that the server certificate or public key matches an expected pinned value instead of relying only on normal trust evaluation.

### Why should tokens not be stored in UserDefaults?

Because UserDefaults is not a secure vault. Tokens can grant account access, so they belong in Keychain with suitable accessibility settings.

### What is Face ID used for in apps?

Face ID can verify local user presence for unlocking the app or approving sensitive local steps. It does not replace server-side authorization.

## 26. Senior-Level Interview Questions And Answers

### How would you design token refresh safely?

Use short-lived access tokens and Keychain-stored refresh tokens. When a request receives 401, coordinate refresh through an actor or single-flight mechanism so multiple requests do not trigger multiple refresh calls. Retry the original request once. If refresh fails, clear session and route to login.

### Certificate pinning vs public key pinning?

Certificate pinning binds the app to an exact certificate and can break during certificate rotation. Public key pinning binds to the key and can survive certificate renewal if the key remains the same. Public key pinning is often more rotation-friendly, but both require backup pins and operational planning.

### How do you protect against reverse engineering?

Do not put secrets in the binary. Keep critical business rules on the server. Strip debug symbols, avoid debug features in release, use tamper/debugger signals where appropriate, protect network communication, and consider App Attest or DeviceCheck for high-risk APIs.

### How would you secure a banking transfer?

Use Keychain, ATS, optional pinning, biometric local gates, server-side authorization, step-up auth, idempotency keys, audit logs, privacy-safe telemetry, and server-confirmed final states. Do not allow final transfer offline.

### Is jailbreak detection enough?

No. It can be bypassed. Treat it as a risk signal and combine it with server-side checks, step-up authentication, device attestation, and limited capabilities for high-risk actions.

### What is the biggest mistake in mobile security?

Trusting the client too much. The client should improve UX and add layers of defense, but server-side authorization and validation must protect critical decisions.

## 27. Common Mistakes

- Storing tokens in UserDefaults.
- Hardcoding API keys as if they are secret.
- Using broad ATS exceptions.
- Pinning certificates without backup or rotation plan.
- Logging sensitive request/response bodies.
- Trusting JWT claims on the client as final authorization.
- Treating biometrics as server authentication.
- Allowing deep links to bypass auth.
- Using weak or custom cryptography.
- Keeping sensitive debug screens in release builds.
- Ignoring third-party SDK privacy behavior.
- Treating jailbreak detection as complete protection.
- Caching sensitive data without product/security approval.

## 28. Points To Remember

- Security is layered defense.
- Keychain is for secrets; UserDefaults is for preferences.
- HTTPS is mandatory; pinning is a high-risk-app enhancement with operational cost.
- Certificate pinning pins certificates; public key pinning pins keys.
- Face ID and Touch ID verify local user presence.
- Secure Enclave protects biometric and key operations; apps do not receive raw biometric data.
- JWTs are usually signed, not encrypted.
- CryptoKit helps with correct cryptographic primitives, but key management remains critical.
- Jailbreak detection is bypassable and should be a risk signal.
- Reverse engineering cannot be fully prevented; do not ship secrets.
- OWASP Mobile Top 10 2024 and MASVS are strong interview frameworks.
- Banking flows must be server-authoritative.
- Secure coding includes privacy, logging, dependencies, deep links, and rollout.

## References

- OWASP Mobile Top 10 2024: https://owasp.org/www-project-mobile-top-10/
- OWASP MASVS: https://mas.owasp.org/MASVS/
- OWASP MAS Checklist: https://mas.owasp.org/checklists/
