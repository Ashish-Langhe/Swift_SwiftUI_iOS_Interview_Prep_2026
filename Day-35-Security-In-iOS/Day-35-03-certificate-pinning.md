# Day 35: Certificate Pinning

## What Certificate Pinning Is

Certificate pinning restricts server trust to specific certificates or public keys expected by your app.

Default TLS trust asks:

```text
Is this certificate valid and trusted by the system?
```

Pinning adds:

```text
Is this certificate or public key one my app expects?
```

## Why Pinning Exists

Pinning can reduce risk from:

- Compromised certificate authorities
- Misissued certificates
- Enterprise proxy trust injection
- Some man-in-the-middle scenarios

It is most useful for high-risk apps:

- Banking
- Payments
- Healthcare
- Enterprise confidential apps

## Pinning Tradeoff

Pinning can break your app if:

- Certificate rotates.
- Pin expires.
- Backend changes CA/provider.
- Emergency cert replacement happens.
- App version with old pins remains installed.

Senior rule:

```text
Pinning must include rotation and backup-pin strategy.
```

## ATS Pinning

Apple supports certificate pinning through ATS configuration using `NSPinnedDomains`.

This lets you declare expected certificate/public-key material in Info.plist.

Use this when possible because it integrates with system trust behavior.

## Manual Pinning With URLSessionDelegate

Conceptual flow:

```swift
final class PinningDelegate: NSObject, URLSessionDelegate {
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.performDefaultHandling, nil)
            return
        }

        if trustEvaluator.isPinned(serverTrust, host: challenge.protectionSpace.host) {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

Do not bypass default trust accidentally. Pinning should tighten trust, not replace basic validation with weaker custom checks.

## Certificate vs Public Key Pinning

Certificate pinning:

- Pins exact certificate.
- Easier to reason about.
- Breaks on certificate renewal.

Public key pinning:

- Pins public key.
- Survives certificate renewal with same key.
- Requires careful extraction/validation.

## Backup Pins

Always plan backup pins.

```text
Current production pin
Next certificate/public-key pin
Emergency backup pin
```

Without backup pins, backend certificate incidents can force urgent app releases.

## Testing Pinning

Test:

- Valid production certificate accepted.
- Invalid certificate rejected.
- Expired/rotated certificate behavior.
- Backup pin accepted.
- Captive portal/proxy behavior.
- Staging domain separate from production.

## When Not To Pin

Avoid pinning when:

- Team cannot manage rotation.
- Backend cert ownership is not controlled.
- App depends on many third-party hosts.
- Availability risk outweighs MITM risk.
- Standard ATS/TLS is sufficient.

## Common Mistakes

- Pinning without backup pins.
- No rotation plan.
- Pinning third-party domains.
- Bypassing default TLS validation.
- Treating pinning as replacement for ATS.
- No monitoring for pin failures.
- Shipping debug pins to production.

## Senior Artifact

```text
Artifact: Certificate Pinning Review
Domain:
Threat model:
Pin type:
Current pin:
Backup pins:
Rotation plan:
Failure behavior:
Monitoring:
Testing:
Rollback plan:
```

## Interview Notes

Junior:

Certificate pinning makes the app trust only expected certificates or keys for a server.

Mid-level:

Pinning can protect against some MITM risks but needs certificate rotation and backup pins.

Senior:

I use pinning only for high-risk domains with a real threat model, backup pins, monitoring, and operational rotation plan. I never weaken default TLS validation.

## Practice

1. Explain certificate vs public key pinning.
2. Design backup pin strategy.
3. Review why pinning third-party hosts is risky.
4. Explain how pinning can break production.
