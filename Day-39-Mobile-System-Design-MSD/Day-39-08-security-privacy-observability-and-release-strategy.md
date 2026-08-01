# Day 39: Security, Privacy, Observability, And Release Strategy

Senior MSD answers must include how the app is operated safely in production. Security, privacy, observability, and release strategy are not afterthoughts; they decide whether the app can survive real users, fraud, crashes, and bad deployments.

## Security Design

Common iOS security areas:

- Authentication.
- Token storage.
- Secure API transport.
- Certificate pinning where appropriate.
- Biometric gates.
- Keychain.
- Sensitive data redaction.
- Jailbreak/root detection as risk signal.
- Runtime tampering awareness.

## Token Handling

Access token:

- Short-lived.
- Attached to API requests.
- Refreshed when expired.

Refresh token:

- Longer-lived.
- Stored in Keychain.
- Rotated if backend supports it.
- Cleared on logout.

Do not store tokens in:

- UserDefaults.
- Plain files.
- Logs.
- Analytics properties.

Senior note:

```text
The app can protect tokens and reduce exposure, but server-side authorization remains the final security boundary.
```

## Biometric Auth

Use biometrics for:

- App unlock.
- Sensitive action confirmation.
- Showing private data.

Do not treat biometrics as:

- Replacement for server auth.
- Proof that a transaction is allowed.
- A guarantee that the device is uncompromised.

Example:

```text
For banking transfer, biometrics can unlock the local session or approve UX step, but the server must validate transaction limits, risk rules, OTP requirements, and account state.
```

## Privacy

Privacy areas:

- Data minimization.
- Permission purpose strings.
- Privacy manifests.
- Tracking transparency where applicable.
- PII redaction in logs.
- Data retention.
- User deletion/export flows.

Senior answer:

```text
I would log operational metadata like status code, endpoint family, duration, app version, and request ID, but never raw tokens, card numbers, OTPs, exact message content, or unnecessary PII.
```

## Observability

Mobile observability must answer:

- Which app version is crashing?
- Which OS/device has the issue?
- Which feature flag was enabled?
- Which API failed?
- Did launch or screen load regress?
- Is the issue regional?
- Is it tied to memory pressure?

Events to track:

- App launch.
- Screen load success/failure.
- API latency buckets.
- Error categories.
- Sync queue size.
- Retry count.
- Crash-free sessions.
- Checkout/payment funnel.

Example event:

```swift
struct AnalyticsEvent: Sendable {
    let name: String
    let properties: [String: String]
}

analytics.track(
    AnalyticsEvent(
        name: "checkout_failed",
        properties: [
            "stage": "payment_authorization",
            "error_family": "network_timeout",
            "app_version": "5.8.0"
        ]
    )
)
```

## Release Strategy

Mobile release is slower than backend release.

Use:

- Feature flags.
- Remote config.
- Phased rollout.
- Kill switches.
- Backward-compatible APIs.
- App version checks.
- Server-side capability negotiation.

Example:

```text
For a new checkout flow, I would ship code dark, enable it for internal users, then 1 percent, 10 percent, and gradually increase while watching crash rate, payment failure rate, and conversion.
```

## Backward Compatibility

Rules:

- Add fields instead of removing fields.
- Make new fields optional where possible.
- Keep old endpoints until old app versions are below acceptable threshold.
- Use enum decoding with unknown fallback.
- Avoid client-breaking server assumptions.

Swift example:

```swift
enum PaymentMethodType: Decodable, Equatable {
    case card
    case upi
    case wallet
    case unknown(String)

    init(from decoder: Decoder) throws {
        let value = try decoder.singleValueContainer().decode(String.self)
        switch value {
        case "card": self = .card
        case "upi": self = .upi
        case "wallet": self = .wallet
        default: self = .unknown(value)
        }
    }
}
```

## Security Tradeoffs

### Certificate Pinning

Pros:

- Reduces man-in-the-middle risk.

Cons:

- Operational risk if certificates rotate incorrectly.
- Can break users if not shipped carefully.

Senior answer:

```text
I would use pinning for high-risk apps like banking, with backup pins and remote kill strategy if supported by policy. I would not casually add pinning without operational readiness.
```

### Local Data

Pros:

- Better UX.
- Offline access.

Cons:

- Privacy and breach risk.

Decision:

```text
Cache product catalog aggressively. Cache bank balance cautiously with refresh time and protection. Never store OTP or CVV.
```

## Interview Notes

- Mention security and privacy before the interviewer asks.
- Observability is a senior differentiator.
- Always consider old app versions.
- Feature flags reduce release risk.
- Security choices need operational backup plans.
