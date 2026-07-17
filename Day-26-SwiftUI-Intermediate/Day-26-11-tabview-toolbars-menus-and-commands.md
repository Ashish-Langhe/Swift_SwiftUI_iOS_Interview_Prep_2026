# Day 26: TabView, Toolbars, Menus, And Commands

## Why This Topic Matters

Real SwiftUI apps need app structure and actions:

- Tabs for major areas
- Toolbars for screen actions
- Menus for secondary actions
- Context menus for item-level actions
- Commands for keyboard/menu integration on iPad and macOS

These APIs shape how users move and act inside the app.

## `TabView`

Use `TabView` for top-level app sections.

```swift
enum AppTab: Hashable {
    case home
    case search
    case account
}

struct RootTabView: View {
    @State private var selectedTab: AppTab = .home

    var body: some View {
        TabView(selection: $selectedTab) {
            NavigationStack {
                HomeView()
            }
            .tabItem {
                Label("Home", systemImage: "house")
            }
            .tag(AppTab.home)

            NavigationStack {
                SearchView()
            }
            .tabItem {
                Label("Search", systemImage: "magnifyingglass")
            }
            .tag(AppTab.search)

            NavigationStack {
                AccountView()
            }
            .tabItem {
                Label("Account", systemImage: "person")
            }
            .tag(AppTab.account)
        }
    }
}
```

Senior tip: give each tab its own navigation stack if each tab has independent navigation history.

## Tab Selection as State

```swift
selectedTab = .account
```

Programmatic tab switching is useful after login, checkout, or deep links.

Avoid making every child mutate tab selection directly. Prefer intent callbacks or a router when the app grows.

## Toolbars

```swift
struct ProductsScreen: View {
    var body: some View {
        List(products) { product in
            ProductRow(product: product)
        }
        .navigationTitle("Products")
        .toolbar {
            ToolbarItem(placement: .topBarTrailing) {
                Button {
                    addProduct()
                } label: {
                    Image(systemName: "plus")
                }
            }
        }
    }
}
```

Toolbars should contain primary screen actions.

## Multiple Toolbar Items

```swift
.toolbar {
    ToolbarItem(placement: .topBarLeading) {
        Button("Cancel") {
            cancel()
        }
    }

    ToolbarItem(placement: .topBarTrailing) {
        Button("Save") {
            save()
        }
        .disabled(!canSave)
    }
}
```

This is common in modal edit flows.

## Menus

Use `Menu` for secondary or grouped actions.

```swift
Menu {
    Button {
        duplicate()
    } label: {
        Label("Duplicate", systemImage: "plus.square.on.square")
    }

    Button(role: .destructive) {
        delete()
    } label: {
        Label("Delete", systemImage: "trash")
    }
} label: {
    Image(systemName: "ellipsis.circle")
}
```

Do not hide the primary action in a menu.

## Context Menus

```swift
ProductRow(product: product)
    .contextMenu {
        Button {
            favorite(product)
        } label: {
            Label("Favorite", systemImage: "heart")
        }

        Button(role: .destructive) {
            delete(product)
        } label: {
            Label("Delete", systemImage: "trash")
        }
    }
```

Context menus are useful accelerators, but actions should usually also be discoverable elsewhere.

## Commands

On iPad/macOS style apps, commands expose keyboard and menu actions.

```swift
struct NotesApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
        }
        .commands {
            CommandMenu("Notes") {
                Button("New Note") {
                    NotificationCenter.default.post(name: .newNoteRequested, object: nil)
                }
                .keyboardShortcut("n", modifiers: [.command])
            }
        }
    }
}
```

Senior note: prefer a clean command routing mechanism over random notifications in production code.

## Latest SwiftUI Notes

Recent SwiftUI updates continue improving toolbar behavior, menu/dialog APIs, and platform-adaptive interaction. The practical point is that top-level structure and action placement should adapt to platform, size class, keyboard, and pointer contexts.

## Common Mistakes

- Using tabs for temporary flows.
- Sharing one navigation stack across unrelated tabs.
- Putting too many actions in the toolbar.
- Hiding primary actions in menus.
- Making context menu actions unavailable elsewhere.
- Forgetting keyboard users on iPad/macOS.

## Senior iOS Engineer Perspective

Senior engineers think about information architecture:

- What are the top-level destinations?
- Which actions are primary enough for toolbar?
- Which actions belong in menu/context menu?
- Should each tab preserve navigation history?
- Are commands testable and discoverable?
- Does the action model work across iPhone, iPad, and macOS?

## Interview Notes

Junior:

`TabView` creates tabs. `.toolbar` adds navigation bar actions. `Menu` groups actions.

Mid-level:

Each tab often owns a navigation stack, and toolbar placement should match platform conventions.

Senior:

I design tabs, toolbars, menus, context menus, and commands around app structure, discoverability, platform adaptation, keyboard support, and clean action routing.

## Practice

1. Build a three-tab app with independent navigation stacks.
2. Add cancel/save toolbar items to an edit sheet.
3. Move secondary actions into a `Menu`.
4. Explain why context menu actions should not be the only way to perform an important task.
