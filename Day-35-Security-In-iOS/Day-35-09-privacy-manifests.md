# Day 35: Privacy Manifests

## What Privacy Manifests Are

A privacy manifest is a property list file named:

```text
PrivacyInfo.xcprivacy
```

It describes:

- Data your app or SDK collects
- Required reason APIs your app or SDK uses
- Tracking domains when applicable

Apple uses privacy manifests to improve transparency and software supply-chain privacy.

## Where It Lives

For iOS apps, the file is placed at the root of the app bundle:

```text
Sample.app/PrivacyInfo.xcprivacy
```

In Xcode, add:

```text
File -> New File -> App Privacy File
```

## Key Fields

Important keys:

```text
NSPrivacyCollectedDataTypes
NSPrivacyAccessedAPITypes
NSPrivacyTracking
NSPrivacyTrackingDomains
```

## Required Reason APIs

Some APIs can be misused for fingerprinting. If your app or SDK uses required reason APIs, you must declare approved reasons.

Apple states that since May 1, 2024, apps that do not describe required reason API usage in their privacy manifest are not accepted by App Store Connect.

Examples of required-reason categories can include:

- File timestamp APIs
- System boot time APIs
- Disk space APIs
- Active keyboard APIs
- User defaults APIs

Always check Apple's current list because it can change.

## Third-Party SDKs

Third-party SDKs may need their own privacy manifest and signature.

Senior review:

- Does SDK collect data?
- Does SDK use required reason APIs?
- Is SDK on Apple's commonly used SDK list?
- Is manifest valid?
- Is SDK signed if required?
- Does App Store Connect report match actual behavior?

## App Privacy Details vs Privacy Manifest

App Privacy Details:

- Entered in App Store Connect.
- Describes data collection shown on App Store product page.

Privacy Manifest:

- Bundled in app/SDK.
- Declares collected data and required reason API use.

Both must be accurate.

## Example Conceptual Manifest

```xml
<dict>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
    </array>
</dict>
```

Use only approved reason strings that match your actual app behavior.

## Privacy Review Questions

Ask:

- What data leaves device?
- Is data linked to user?
- Is data used for tracking?
- Which SDKs collect data?
- Which required reason APIs are used?
- Are reasons accurate?
- Are privacy labels up to date?
- Is data minimized?

## Common Mistakes

- Missing `PrivacyInfo.xcprivacy`.
- Invalid manifest keys/values.
- SDK manifest missing.
- Declared reasons do not match behavior.
- App privacy labels outdated.
- Assuming on-device data is "collected."
- Not reviewing new SDKs.

## Senior Artifact

```text
Artifact: Privacy Manifest Review
Target:
PrivacyInfo.xcprivacy present:
Collected data:
Required reason APIs:
Approved reasons:
Tracking:
Tracking domains:
Third-party SDK manifests:
App Store privacy labels:
Open risks:
```

## Interview Notes

Junior:

Privacy manifests describe data collection and required API reasons.

Mid-level:

Add `PrivacyInfo.xcprivacy`, declare collected data and required reason APIs, and keep App Store privacy labels accurate.

Senior:

I treat privacy manifests as part of release governance. I audit app and SDK data collection, required reason APIs, tracking domains, manifests, SDK signatures, and App Store privacy labels.

## Practice

1. Explain `PrivacyInfo.xcprivacy`.
2. List privacy review questions for analytics SDK.
3. Compare privacy manifest and App Store privacy labels.
4. Explain required reason APIs.
