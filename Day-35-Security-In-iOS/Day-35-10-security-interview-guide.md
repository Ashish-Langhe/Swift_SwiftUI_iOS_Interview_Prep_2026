# Day 35: Security Interview Guide

## One-Minute Senior Answer

iOS security is layered. I rely on ATS and strong TLS for network security, Keychain for small secrets, LocalAuthentication and Keychain access control for biometric-gated access, secure storage decisions for files and caches, careful secrets handling because app binaries cannot hide static secrets, jailbreak detection only as a weak risk signal, OWASP Mobile Top 10 as a review framework, and privacy manifests/App Store privacy labels for modern privacy compliance. I pair client-side controls with server-side authorization, monitoring, and revocation.

## Core Topic Map

```text
ATS:
Secure network transport

Keychain:
Small secrets

Certificate pinning:
Extra trust restriction for high-risk domains

Biometric auth:
Local user presence, not backend login

Secure storage:
Right storage for sensitivity and size

Jailbreak detection:
Bypassable risk signal

Secrets handling:
No static mobile secrets

OWASP Mobile Top 10:
Security review framework

Privacy manifests:
Data/API transparency and App Store requirement
```

## Junior Questions

What is ATS?

App Transport Security enforces secure HTTPS/TLS connections.

What is Keychain for?

Secure storage of small sensitive values like tokens and passwords.

What is Face ID used for in apps?

Local authentication or unlocking Keychain-protected secrets.

## Mid-Level Questions

Where do you store auth tokens?

In Keychain with appropriate accessibility, never in UserDefaults.

What is certificate pinning?

Restricting trust to expected certificate or public key material for a domain.

What is a privacy manifest?

A bundled `PrivacyInfo.xcprivacy` file declaring collected data and required reason API usage.

## Senior Questions

How do you handle secrets in mobile apps?

I assume bundled secrets can be extracted. True secrets stay server-side. The app uses short-lived scoped tokens, stores runtime tokens in Keychain, redacts logs, and supports rotation/revocation.

When would you use certificate pinning?

Only for high-risk domains where threat model justifies operational risk. I require backup pins, rotation plan, monitoring, and no weakening of default TLS validation.

How do you treat jailbreak detection?

As defense-in-depth and risk telemetry, not a reliable security boundary. Sensitive decisions should involve server-side risk controls.

How do you prepare for App Store privacy review?

I audit data collection, SDK behavior, required reason APIs, tracking domains, privacy manifests, SDK signatures, and App Store privacy labels.

## Senior Artifact: iOS Security Review

```text
Feature:
Sensitive data:
Storage:
Network domains:
ATS exceptions:
Auth/session:
Token refresh:
Biometric gates:
Certificate pinning:
Secrets:
Logs:
Jailbreak/tamper response:
OWASP risks:
Privacy manifest:
SDK review:
Open risks:
```

## Real Scenario: Banking App

Security decisions:

- ATS default enabled, no broad exceptions.
- Pin high-risk API domain with backup pins.
- Refresh token in Keychain `ThisDeviceOnly`.
- Biometric unlock for viewing account details.
- Server enforces all authorization.
- Jailbreak detection contributes to risk scoring.
- Logs redact all PII and tokens.
- Privacy manifest declares required reason APIs.
- Third-party SDKs reviewed.
- Logout clears tokens and local personal cache.

## Common Interview Traps

- Saying Keychain is for all encrypted storage.
- Storing API secrets in app bundle.
- Treating Face ID as backend authentication.
- Relying on jailbreak detection as complete protection.
- Enabling `NSAllowsArbitraryLoads`.
- Pinning without rotation plan.
- Ignoring privacy manifests and SDK data collection.

## Strong Closing Answer

Security is not one API. It is threat modeling, secure defaults, minimal data, strong storage, safe networking, backend enforcement, release governance, and continuous review. Senior iOS engineers know where client-side security helps and where it cannot be trusted alone.

## Practice Prompts

1. Design secure auth storage for a finance app.
2. Review ATS exceptions for a media app.
3. Explain pinning tradeoffs.
4. Build a biometric-gated Keychain flow.
5. Map a checkout feature to OWASP risks.
6. Create a privacy manifest review checklist.
