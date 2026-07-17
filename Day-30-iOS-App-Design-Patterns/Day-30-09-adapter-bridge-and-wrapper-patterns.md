# Day 30: Adapter, Bridge, And Wrapper Patterns

## Why Adapter Patterns Matter

iOS apps often integrate:

- Third-party SDKs
- Legacy UIKit code
- Platform APIs
- Vendor analytics
- Payment SDKs
- Old services

Adapter/wrapper patterns protect your app from external API shapes.

## Adapter Pattern

Adapter converts one interface into another.

Third-party SDK:

```swift
final class VendorAnalytics {
    func logEvent(_ name: String, parameters: [String: Any]) {}
}
```

App protocol:

```swift
protocol AnalyticsClient {
    func track(_ event: AnalyticsEvent)
}
```

Adapter:

```swift
struct VendorAnalyticsAdapter: AnalyticsClient {
    let vendor: VendorAnalytics

    func track(_ event: AnalyticsEvent) {
        vendor.logEvent(event.name, parameters: event.parameters)
    }
}
```

Now the app does not depend directly on vendor API.

## Wrapper Pattern

Wrapper hides platform details.

```swift
protocol KeychainStoring {
    func save(_ value: String, key: String) throws
    func value(for key: String) throws -> String?
}
```

Implementation wraps Keychain APIs.

ViewModels depend on `KeychainStoring`, not Security framework calls.

## Bridge Pattern

Bridge separates abstraction from implementation.

```swift
protocol ImageLoading {
    func image(from url: URL) async throws -> UIImage
}

struct ProductImageProvider {
    let imageLoader: ImageLoading

    func thumbnail(for product: Product) async throws -> UIImage {
        try await imageLoader.image(from: product.thumbnailURL)
    }
}
```

You can switch image loading implementations without changing provider logic.

## UIKit Adapter In SwiftUI

```swift
struct LegacyRatingControl: UIViewRepresentable {
    @Binding var rating: Int

    func makeUIView(context: Context) -> UISegmentedControl {
        let control = UISegmentedControl(items: ["1", "2", "3", "4", "5"])
        control.addTarget(
            context.coordinator,
            action: #selector(Coordinator.changed(_:)),
            for: .valueChanged
        )
        return control
    }

    func updateUIView(_ uiView: UISegmentedControl, context: Context) {
        uiView.selectedSegmentIndex = rating - 1
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(rating: $rating)
    }

    final class Coordinator: NSObject {
        var rating: Binding<Int>

        init(rating: Binding<Int>) {
            self.rating = rating
        }

        @objc func changed(_ sender: UISegmentedControl) {
            rating.wrappedValue = sender.selectedSegmentIndex + 1
        }
    }
}
```

`UIViewRepresentable` is an adapter from UIKit to SwiftUI.

## When To Use Adapter

Use when:

- Vendor API should not leak.
- Legacy interface differs from new app interface.
- Tests need fake implementations.
- You expect SDK replacement.
- Platform API is verbose or unsafe.

Skip when:

- You only use API once.
- Wrapper adds no simplification.
- Adapter hides important behavior.

## Common Mistakes

- Vendor SDK types leaking into domain.
- Adapter becomes a god service.
- Wrapper hides errors too aggressively.
- Direct SDK calls scattered across app.
- No tests around mapping.
- Adapter performs unrelated business logic.

## Senior Artifact

```text
Artifact: Adapter Boundary Review
External API:
App protocol:
Adapter:
Mapped types:
Error mapping:
Vendor leakage:
Tests:
Replacement risk:
```

## Interview Notes

Junior:

Adapter lets incompatible interfaces work together.

Mid-level:

Use adapters to wrap SDKs and platform APIs behind app protocols.

Senior:

I use adapters to prevent vendor/platform APIs from leaking into domain and presentation layers. I keep mapping explicit and test replacement-sensitive boundaries.

## Practice

1. Wrap an analytics SDK behind `AnalyticsClient`.
2. Wrap Keychain behind a protocol.
3. Create a SwiftUI wrapper for UIKit control.
4. Explain adapter vs facade.
