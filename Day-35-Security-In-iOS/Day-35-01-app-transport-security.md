# Day 35: App Transport Security

## What App Transport Security Is

App Transport Security, or ATS, is Apple's network security policy that requires secure network connections for apps and app extensions.

By default, ATS expects URL Loading System traffic, such as `URLSession`, to use HTTPS with strong TLS settings.

ATS improves:

- User privacy
- Data integrity
- Protection against passive network sniffing
- Protection against weak TLS configurations
- App Store security posture

## Default Rule

Use HTTPS.

```swift
let url = URL(string: "https://api.example.com/profile")!
let (data, response) = try await URLSession.shared.data(from: url)
```

Avoid plain HTTP.

```swift
let url = URL(string: "http://api.example.com/profile")!
```

ATS blocks insecure cleartext HTTP loads unless you configure exceptions.

## ATS Requirements

ATS requires strong server-side security, including:

- Valid certificate
- Certificate name matches server DNS name
- Certificate not expired
- Certificate chain trusted
- TLS 1.2 or later
- Strong ciphers
- SHA-256 or stronger certificate signature
- Forward secrecy in default policy

Senior note: many ATS fixes belong on the server, not in the app.

## Info.plist Configuration

ATS configuration lives under `NSAppTransportSecurity`.

Example domain-specific exception:

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

This is safer than allowing arbitrary loads globally.

## Avoid Global Arbitrary Loads

Bad:

```xml
<key>NSAllowsArbitraryLoads</key>
<true/>
```

This disables ATS broadly and weakens the app.

Better:

- Fix the server.
- Use HTTPS.
- Use domain-specific exceptions only when unavoidable.
- Document App Store justification.

## Web Content And Media Exceptions

There are narrower exceptions:

- `NSAllowsArbitraryLoadsInWebContent`
- `NSAllowsArbitraryLoadsForMedia`
- `NSAllowsLocalNetworking`

Use these only for legitimate product needs.

Example:

```xml
<key>NSAllowsLocalNetworking</key>
<true/>
```

Useful for local device communication, such as development servers or local hardware integrations, but still review carefully.

## Certificate Transparency And Pinning

ATS supports stronger policies such as:

- `NSRequiresCertificateTransparency`
- `NSPinnedDomains`

These strengthen trust rather than weaken it.

Use for high-risk domains:

- Banking
- Healthcare
- Payment
- Enterprise regulated traffic

## Debugging ATS Failures

Symptoms:

```text
App Transport Security has blocked a cleartext HTTP resource load.
```

Debug flow:

```text
1. Confirm URL uses HTTPS
2. Check certificate validity
3. Check TLS version/ciphers
4. Test with nscurl
5. Fix server if possible
6. Add narrow exception only if unavoidable
7. Document justification
```

## Senior iOS Engineer Perspective

Ask:

- Why is an exception needed?
- Can the server be fixed?
- Is exception global or domain-specific?
- Does exception require App Store justification?
- Are web content and app API traffic separated?
- Is this regulated traffic?
- Are lower-level networking APIs bypassing ATS?

## Common Mistakes

- Setting `NSAllowsArbitraryLoads` to true for convenience.
- Shipping development ATS exceptions.
- Not documenting exception reason.
- Assuming ATS applies to all low-level networking.
- Ignoring server certificate quality.
- Weakening security instead of fixing backend TLS.

## Interview Notes

Junior:

ATS enforces secure HTTPS connections in iOS apps.

Mid-level:

Use HTTPS by default, avoid broad exceptions, and configure only narrow domain-specific exceptions when necessary.

Senior:

I treat ATS exceptions as security debt. I prefer fixing server TLS, use the narrowest possible exception, document App Store justification, and strengthen high-risk traffic with certificate transparency or pinning where appropriate.

## Practice

1. Explain why HTTP API calls fail under ATS.
2. Write a narrow domain-specific ATS exception.
3. Compare `NSAllowsArbitraryLoads` with `NSExceptionDomains`.
4. Explain why fixing the server is better than weakening ATS.
