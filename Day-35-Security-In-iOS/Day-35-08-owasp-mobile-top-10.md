# Day 35: OWASP Mobile Top 10

## What OWASP Mobile Top 10 Is

OWASP Mobile Top 10 is a list of major mobile app security risks.

The 2024 list includes:

```text
M1: Improper Credential Usage
M2: Inadequate Supply Chain Security
M3: Insecure Authentication/Authorization
M4: Insufficient Input/Output Validation
M5: Insecure Communication
M6: Inadequate Privacy Controls
M7: Insufficient Binary Protections
M8: Security Misconfiguration
M9: Insecure Data Storage
M10: Insufficient Cryptography
```

Use it as an interview and security-review map.

## M1: Improper Credential Usage

Examples:

- Hardcoded secrets
- Tokens in UserDefaults
- Long-lived credentials
- Tokens logged
- Credentials sent to wrong endpoint

iOS mitigation:

- Keychain
- Token refresh
- Redacted logs
- Server-side secrets
- Least privilege tokens

## M2: Inadequate Supply Chain Security

Examples:

- Untrusted SDKs
- Outdated dependencies
- Unsigned binary frameworks
- Dependency confusion
- Third-party SDK privacy risk

iOS mitigation:

- Review dependencies
- Pin versions
- Use SPM responsibly
- Audit SDK privacy manifests
- Monitor CVEs/releases
- Prefer trusted vendors

## M3: Insecure Authentication/Authorization

Examples:

- Client-side-only authorization
- Weak session handling
- Refresh token misuse
- No logout revocation

iOS mitigation:

- Server-enforced authorization
- Secure token storage
- Token refresh actor
- Reauth for sensitive actions
- Session expiry handling

## M4: Insufficient Input/Output Validation

Examples:

- Trusting deep-link parameters
- Unsafe file paths
- Rendering untrusted HTML
- API response assumptions

iOS mitigation:

- Validate deep links
- Sanitize filenames
- Validate DTO mapping
- Avoid unsafe WebView content
- Handle unexpected server values

## M5: Insecure Communication

Examples:

- HTTP APIs
- Weak TLS
- Disabled ATS
- No pinning for high-risk apps

iOS mitigation:

- ATS
- HTTPS
- Strong TLS
- Certificate transparency/pinning where justified
- No broad exceptions

## M6: Inadequate Privacy Controls

Examples:

- Excess data collection
- Tracking without disclosure
- Missing privacy manifests
- Overbroad permissions

iOS mitigation:

- Data minimization
- Purpose strings
- Privacy manifest
- App Store privacy labels
- Permission gating

## M7: Insufficient Binary Protections

Examples:

- Easy reverse engineering
- No tamper detection
- Debug code shipped
- Sensitive strings in binary

iOS mitigation:

- Remove debug features
- Avoid static secrets
- Obfuscation only as defense-in-depth
- Jailbreak/tamper risk signals
- Server-side controls

## M8: Security Misconfiguration

Examples:

- Debug endpoints in production
- ATS disabled
- Verbose logs
- Misconfigured entitlements

iOS mitigation:

- Build config review
- Entitlement review
- Release checklist
- Logging controls
- Environment separation

## M9: Insecure Data Storage

Examples:

- Tokens in UserDefaults
- Sensitive files unprotected
- Plaintext caches
- Logs with PII

iOS mitigation:

- Keychain
- File protection
- Encryption
- Logout cleanup
- Cache policy

## M10: Insufficient Cryptography

Examples:

- Custom crypto
- Weak algorithms
- Hardcoded keys
- Poor random generation

iOS mitigation:

- Use CryptoKit/Security
- System random APIs
- Keychain for keys
- Avoid custom crypto
- Rotate keys

## Senior Artifact

```text
Artifact: OWASP Mobile Review
Feature:
M1 credential risk:
M2 supply chain risk:
M3 auth risk:
M4 validation risk:
M5 communication risk:
M6 privacy risk:
M7 binary risk:
M8 config risk:
M9 storage risk:
M10 crypto risk:
Mitigations:
Open risks:
```

## Interview Notes

Junior:

OWASP Mobile Top 10 lists common mobile security risks.

Mid-level:

Map app features to risks like insecure storage, insecure communication, weak auth, and privacy controls.

Senior:

I use OWASP as a review framework, connecting each risk to iOS-specific mitigations, backend controls, privacy requirements, dependency governance, and release checklists.

## Practice

1. Map login feature to OWASP risks.
2. Review checkout against M1, M3, M5, M9.
3. Explain why supply chain security matters.
4. Create an OWASP review checklist for a banking app.
