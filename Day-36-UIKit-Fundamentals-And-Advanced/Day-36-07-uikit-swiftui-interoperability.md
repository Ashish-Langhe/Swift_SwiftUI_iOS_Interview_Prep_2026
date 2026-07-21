# Day 36.7: UIKit And SwiftUI Interoperability

Most real iOS teams do not rewrite entire apps overnight. UIKit and SwiftUI often live together for years. Senior engineers need to know how to embed SwiftUI in UIKit, wrap UIKit in SwiftUI, share state safely, coordinate navigation, and avoid lifecycle or ownership confusion.

## Why Interoperability Matters

Use cases:

- Add a new SwiftUI screen inside an existing UIKit app.
- Reuse an old UIKit control in a SwiftUI feature.
- Embed SwiftUI cells inside collection views.
- Keep a UIKit coordinator while migrating screens gradually.
- Use UIKit APIs that SwiftUI does not expose directly.

## SwiftUI Inside UIKit With `UIHostingController`

`UIHostingController` is a UIKit view controller that manages a SwiftUI view hierarchy.

```swift
import SwiftUI
import UIKit

struct ProfileHeaderView: View {
    let name: String

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(name).font(.title.bold())
            Text("iOS Engineer").foregroundStyle(.secondary)
        }
        .padding()
    }
}

final class ProfileViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let header = UIHostingController(rootView: ProfileHeaderView(name: "Aarav"))
        addChild(header)
        view.addSubview(header.view)
        header.view.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            header.view.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            header.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            header.view.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])

        header.didMove(toParent: self)
    }
}
```

Senior detail: treat `UIHostingController` like any child view controller. Use correct containment.

## Presenting SwiftUI From UIKit

```swift
func showSettings() {
    let settingsView = SettingsView(
        onDone: { [weak self] in
            self?.dismiss(animated: true)
        }
    )

    let controller = UIHostingController(rootView: settingsView)
    controller.modalPresentationStyle = .formSheet
    present(controller, animated: true)
}
```

This is a common migration step.

## UIKit Inside SwiftUI With `UIViewRepresentable`

Wrap a UIKit view:

```swift
struct SearchTextField: UIViewRepresentable {
    @Binding var text: String

    func makeUIView(context: Context) -> UITextField {
        let textField = UITextField()
        textField.placeholder = "Search"
        textField.delegate = context.coordinator
        return textField
    }

    func updateUIView(_ uiView: UITextField, context: Context) {
        if uiView.text != text {
            uiView.text = text
        }
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(text: $text)
    }

    final class Coordinator: NSObject, UITextFieldDelegate {
        private var text: Binding<String>

        init(text: Binding<String>) {
            self.text = text
        }

        func textFieldDidChangeSelection(_ textField: UITextField) {
            text.wrappedValue = textField.text ?? ""
        }
    }
}
```

The coordinator bridges UIKit delegate callbacks into SwiftUI state.

## UIKit View Controller Inside SwiftUI

Use `UIViewControllerRepresentable`.

```swift
struct DocumentScannerScreen: UIViewControllerRepresentable {
    let onFinish: ([UIImage]) -> Void

    func makeUIViewController(context: Context) -> ScannerViewController {
        let controller = ScannerViewController()
        controller.delegate = context.coordinator
        return controller
    }

    func updateUIViewController(_ controller: ScannerViewController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(onFinish: onFinish)
    }

    final class Coordinator: NSObject, ScannerViewControllerDelegate {
        let onFinish: ([UIImage]) -> Void

        init(onFinish: @escaping ([UIImage]) -> Void) {
            self.onFinish = onFinish
        }

        func scanner(_ controller: ScannerViewController, didFinish images: [UIImage]) {
            onFinish(images)
        }
    }
}
```

Use cases:

- Camera flows.
- Document scanners.
- Map controls.
- Existing UIKit feature modules.
- Third-party UIKit SDKs.

## SwiftUI In Collection/Table Cells

Use `UIHostingConfiguration` for SwiftUI content in UIKit cells.

```swift
cell.contentConfiguration = UIHostingConfiguration {
    ProductRowView(product: product)
}
```

Good for:

- Gradual UI migration.
- Rich row views.
- Sharing SwiftUI row components across screens.

Senior caution:

- Keep hosted cell views lightweight.
- Make identity stable.
- Profile scrolling.
- Do not hide complex state ownership inside every cell.

## State Ownership Across Frameworks

UIKit owning state:

```swift
final class ProfileViewController: UIViewController {
    private let viewModel: ProfileViewModel

    func showHeader() {
        hostingController.rootView = ProfileHeaderView(
            state: viewModel.headerState,
            onFollow: { [weak self] in self?.viewModel.follow() }
        )
    }
}
```

SwiftUI owning state:

```swift
struct ProfileScreen: View {
    @StateObject private var viewModel = ProfileViewModel()

    var body: some View {
        LegacyChartView(data: viewModel.chartData)
    }
}
```

Pick one owner. Avoid both UIKit and SwiftUI mutating the same state independently.

## Navigation

Common production approach:

- UIKit coordinator owns navigation stack.
- SwiftUI views emit intents through closures.
- Coordinator pushes UIKit or SwiftUI screens.

```swift
struct ProductDetailsView: View {
    let onOpenReviews: () -> Void

    var body: some View {
        Button("Reviews", action: onOpenReviews)
    }
}

final class ProductCoordinator {
    let navigationController: UINavigationController

    func showProductDetails(id: Product.ID) {
        let view = ProductDetailsView(
            onOpenReviews: { [weak self] in
                self?.showReviews(productID: id)
            }
        )
        navigationController.pushViewController(UIHostingController(rootView: view), animated: true)
    }
}
```

This prevents SwiftUI screens from knowing about UIKit navigation controllers.

## Lifecycle Differences

UIKit lifecycle:

- Controller object owns a view.
- Visibility callbacks are explicit.
- View identity is mostly object identity.

SwiftUI lifecycle:

- Views are value descriptions.
- State drives rendering.
- `.onAppear` can run more often than expected.
- Identity depends on data and view structure.

Senior migration risk: do not map `viewDidLoad` directly to SwiftUI `init` or `.onAppear`. They are different models.

## Main Actor And Concurrency

UI work belongs on the main actor.

```swift
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published private(set) var name = ""

    func load() async {
        name = await service.fetchName()
    }
}
```

When UIKit calls into SwiftUI state, keep updates main-actor safe.

## Common Mistakes

- Forgetting child containment for `UIHostingController`.
- Creating two navigation owners.
- Letting SwiftUI and UIKit both own the same mutable state.
- Using `.onAppear` as if it were `viewDidLoad`.
- Retain cycles in callbacks passed to SwiftUI.
- Overusing hosting configurations in heavy scrolling cells without profiling.

## Junior-Level Interview Answer

You can put SwiftUI inside UIKit using `UIHostingController`. You can put UIKit inside SwiftUI using `UIViewRepresentable` or `UIViewControllerRepresentable`. Coordinators help bridge UIKit delegate callbacks to SwiftUI.

## Senior-Level Interview Answer

Interop is a migration and boundary design problem. I choose a single state owner, keep navigation ownership explicit, use hosting controllers with correct containment, use representables for UIKit APIs SwiftUI does not expose, and profile hosted cells. I avoid mixing lifecycle assumptions because SwiftUI view identity and UIKit object identity are different.

## Points To Remember

- `UIHostingController` embeds SwiftUI in UIKit.
- `UIViewRepresentable` wraps UIKit views for SwiftUI.
- `UIViewControllerRepresentable` wraps UIKit controllers.
- Coordinators bridge delegate callbacks.
- One owner for state and navigation is cleaner than two.
- Interop is not temporary in many real apps; design it intentionally.

