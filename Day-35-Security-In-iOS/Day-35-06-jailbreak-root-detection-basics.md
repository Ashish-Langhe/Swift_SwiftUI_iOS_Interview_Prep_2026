# Day 35: Jailbreak And Root Detection Basics

## What Jailbreak Detection Means

Jailbreak/root detection tries to identify whether the iOS device security model has been weakened.

Signals may include:

- Suspicious file paths
- Ability to write outside sandbox
- Suspicious URL schemes
- Loaded dynamic libraries
- Debugger/tampering signs
- Abnormal sandbox behavior

## Important Reality

Jailbreak detection is not a guarantee.

An attacker controlling the device can often bypass detection.

Use it as:

- Risk signal
- Defense-in-depth layer
- Fraud/security telemetry
- Feature gating for high-risk operations

Do not treat it as perfect protection.

## Basic File Path Check

```swift
let suspiciousPaths = [
    "/Applications/Cydia.app",
    "/Library/MobileSubstrate/MobileSubstrate.dylib",
    "/bin/bash",
    "/usr/sbin/sshd",
    "/etc/apt"
]

let isSuspicious = suspiciousPaths.contains {
    FileManager.default.fileExists(atPath: $0)
}
```

This is easy to bypass and should not be your only check.

## Sandbox Write Check

```swift
func canWriteOutsideSandbox() -> Bool {
    let path = "/private/security-test.txt"

    do {
        try "test".write(toFile: path, atomically: true, encoding: .utf8)
        try FileManager.default.removeItem(atPath: path)
        return true
    } catch {
        return false
    }
}
```

If write succeeds, sandbox may be compromised.

## URL Scheme Check

```swift
if let url = URL(string: "cydia://package/com.example"),
   UIApplication.shared.canOpenURL(url) {
    // suspicious
}
```

Requires `LSApplicationQueriesSchemes` for queried schemes. Also easy to bypass.

## Debugger Detection

Debugger checks can identify casual tampering, but must be used carefully. They can interfere with development/testing and can be bypassed.

Senior strategy:

- Disable or soften behavior in debug/internal builds.
- Treat as risk score.
- Avoid breaking legitimate users without strong evidence.

## Response Strategy

Options:

- Log risk event.
- Disable sensitive features.
- Require reauthentication.
- Block high-risk actions.
- Show generic security message.

Avoid revealing exact detection reason to attackers.

## Server-Side Signals

Device-side detection should be paired with server-side controls:

- Risk scoring
- Device attestation where available
- Rate limiting
- Transaction monitoring
- Fraud detection
- Session revocation

## Common Mistakes

- Relying only on jailbreak detection.
- Revealing detection details in UI.
- Blocking all users based on weak signal.
- Breaking debug/test builds.
- No server-side risk model.
- Hardcoded checks never updated.
- Thinking detection equals prevention.

## Senior Artifact

```text
Artifact: Jailbreak Risk Review
Threat model:
Detection signals:
Bypass risk:
False positive risk:
User impact:
Server-side controls:
Sensitive features gated:
Telemetry:
Release behavior:
```

## Interview Notes

Junior:

Jailbreak detection checks whether device security may be compromised.

Mid-level:

Use multiple signals, but understand they can be bypassed.

Senior:

I treat jailbreak detection as defense-in-depth and risk telemetry, not a security boundary. Sensitive decisions should combine server-side controls, attestation/risk scoring, and careful user impact analysis.

## Practice

1. List common jailbreak signals.
2. Explain why detection is bypassable.
3. Design a response for banking app risk.
4. Explain false positive risks.
