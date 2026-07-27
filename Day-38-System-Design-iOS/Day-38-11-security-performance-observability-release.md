# Day 38: Security, Performance, Observability, And Release Design

Production iOS system design must include more than screens and APIs. Senior interviews expect you to discuss security, performance, observability, feature flags, rollout, and operational safety.

## Security Design

Include:

- App Transport Security.
- Token storage in Keychain.
- No secrets hardcoded in app.
- Certificate pinning only when justified.
- Biometric access for sensitive local data.
- Privacy manifests.
- Jailbreak/root detection as risk signal only.
- Secure logout cleanup.
- Payment tokenization.

## Sensitive Data Rules

Never store:

- Passwords.
- Raw card details.
- Long-lived secrets in code.
- Sensitive personal data in logs.

Store carefully:

- Access token: memory, short-lived.
- Refresh token: Keychain.
- User preferences: UserDefaults if non-sensitive.
- Sensitive local documents: file protection/encryption.

## Performance Design

Measure:

- App launch.
- Screen load.
- Scroll FPS.
- Memory.
- Network latency.
- Image decode time.
- Video startup time.
- Battery impact.

Design:

- Cache images.
- Downsample images.
- Avoid main-thread IO.
- Cancel stale requests.
- Use pagination.
- Avoid over-rendering SwiftUI/UIKit lists.
- Precompute display models.

## Observability

App should report:

- Crashes.
- Non-fatal errors.
- API failures.
- Decoding failures.
- Request latency.
- Playback errors.
- Checkout failures.
- Cache hit rate.
- Feature flag assignment.

Every backend error should ideally include a request ID that support/debugging can trace.

## Logging

Good logs:

```text
requestID=req_123 path=/orders status=500 retryable=true
```

Bad logs:

```text
token=abc password=secret card=4111111111111111
```

## Analytics

Analytics answer should include:

- Event schema.
- User properties.
- Funnel metrics.
- Experiment assignment.
- Privacy.
- Offline batching.
- Retry.
- Opt-out handling.

## Feature Flags

Use feature flags for:

- Gradual rollout.
- Kill switch.
- A/B tests.
- Backend compatibility.
- Region-specific features.

iOS concern:

- App should have safe defaults.
- Remote config fetch should not block launch forever.
- Flags should be typed.

## Release Strategy

Large apps release carefully:

- Internal testing.
- TestFlight.
- Phased rollout.
- Feature flags.
- Backend compatibility.
- Crash monitoring.
- Rollback through server-side kill switch when possible.

App Store releases cannot be rolled back instantly like backend deploys.

## Migration Strategy

When changing APIs:

1. Add new fields.
2. Release app that reads new fields.
3. Wait for adoption.
4. Stop depending on old fields.
5. Remove old fields much later.

Senior note: mobile clients force long backward compatibility windows.

## Incident Thinking

Example checkout failure:

- Detect spike in payment failure.
- Correlate by app version, OS, region.
- Use request IDs.
- Disable new payment experiment via flag.
- Show graceful user message.
- Monitor recovery.

## Interview Answer Example

```text
For production readiness, I would secure tokens in Keychain, avoid logging sensitive data, use typed analytics, add request IDs, track crash-free sessions and API latency, gate risky features behind flags, and support phased rollout. Since iOS rollback is slow, backend contracts must remain backward compatible.
```

## Common Mistakes

- No observability.
- No feature flag kill switch.
- App launch blocked by remote config.
- Secrets in app bundle.
- No request IDs.
- No migration story.
- No privacy discussion.
- No performance metrics.

