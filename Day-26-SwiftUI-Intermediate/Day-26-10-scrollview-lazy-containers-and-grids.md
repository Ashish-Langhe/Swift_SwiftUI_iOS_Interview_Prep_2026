# Day 26: ScrollView, Lazy Containers, And Grids

## Why This Topic Matters

Many SwiftUI screens are not simple `List` screens. Product galleries, dashboards, profile pages, feeds, onboarding screens, and custom layouts often use:

- `ScrollView`
- `LazyVStack`
- `LazyHStack`
- `LazyVGrid`
- `LazyHGrid`
- `Grid`
- `ScrollViewReader`

Knowing when to use `List` versus scroll/lazy containers is a real intermediate-to-senior skill.

## `ScrollView`

`ScrollView` gives you a scrolling region without the built-in row behavior of `List`.

```swift
struct ProfileScreen: View {
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 24) {
                ProfileHeader()
                StatsSummary()
                RecentActivitySection()
            }
            .padding()
        }
        .navigationTitle("Profile")
    }
}
```

Use `ScrollView` when you need custom composition more than table/list behavior.

## `ScrollView` vs `List`

Use `List` when:

- Rows are primary.
- You need built-in row styling.
- You need edit mode, selection, swipe actions, sections, or platform table behavior.

Use `ScrollView` when:

- Layout is custom.
- Content is mixed.
- You need a dashboard/feed/gallery.
- You want full control over spacing, backgrounds, and grouping.

Senior note: many apps overuse `ScrollView` and rebuild list behavior badly. Many others overuse `List` and fight its styling. Choose based on interaction model.

## Lazy Stacks

`LazyVStack` creates child views as needed.

```swift
struct FeedView: View {
    let posts: [Post]

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 16) {
                ForEach(posts) { post in
                    PostCard(post: post)
                }
            }
            .padding()
        }
    }
}
```

Use lazy containers for large content. For small static content, a normal `VStack` is simpler.

## Lazy Grids

```swift
struct ProductGridView: View {
    let products: [Product]

    private let columns = [
        GridItem(.adaptive(minimum: 160), spacing: 16)
    ]

    var body: some View {
        ScrollView {
            LazyVGrid(columns: columns, spacing: 16) {
                ForEach(products) { product in
                    ProductCard(product: product)
                }
            }
            .padding()
        }
    }
}
```

Adaptive columns are excellent for iPhone/iPad layouts.

## Fixed vs Flexible vs Adaptive Grid Items

Fixed:

```swift
GridItem(.fixed(120))
```

Flexible:

```swift
GridItem(.flexible(minimum: 100, maximum: 200))
```

Adaptive:

```swift
GridItem(.adaptive(minimum: 140))
```

Senior preference: use adaptive when content should naturally reflow across device sizes.

## `Grid`

Use `Grid` for smaller, table-like layouts where all cells exist at once.

```swift
Grid(alignment: .leading, horizontalSpacing: 16, verticalSpacing: 12) {
    GridRow {
        Text("Subtotal")
        Text(subtotalText)
    }

    GridRow {
        Text("Tax")
        Text(taxText)
    }

    Divider()
        .gridCellColumns(2)

    GridRow {
        Text("Total").bold()
        Text(totalText).bold()
    }
}
```

Use lazy grids for large scrolling collections. Use `Grid` for compact structured layout.

## Programmatic Scrolling

```swift
struct ChatView: View {
    let messages: [Message]

    var body: some View {
        ScrollViewReader { proxy in
            ScrollView {
                LazyVStack {
                    ForEach(messages) { message in
                        MessageBubble(message: message)
                            .id(message.id)
                    }
                }
            }
            .onChange(of: messages.last?.id) { _, lastID in
                guard let lastID else { return }

                withAnimation {
                    proxy.scrollTo(lastID, anchor: .bottom)
                }
            }
        }
    }
}
```

Stable IDs are required for reliable scrolling.

## Pull To Refresh

```swift
ScrollView {
    LazyVStack {
        ForEach(products) { product in
            ProductRow(product: product)
        }
    }
}
.refreshable {
    await model.reload()
}
```

Use this for user-initiated refresh, not initial loading.

## Common Mistakes

- Using `VStack` with thousands of items.
- Using `ScrollView` where `List` behavior is needed.
- Nesting vertical scroll views.
- Forgetting stable IDs with `ScrollViewReader`.
- Hard-coding grid columns for one device size.
- Making heavy row/card bodies.

## Senior iOS Engineer Perspective

Senior engineers choose containers by behavior:

- Is this row/table interaction or custom content composition?
- Is the data large enough to need lazy creation?
- Does the layout adapt across iPhone, iPad, and dynamic type?
- Are item IDs stable?
- Does scrolling preserve position correctly?
- Are expensive views created only when needed?

## Interview Notes

Junior:

`ScrollView` scrolls content. Lazy stacks and grids create items as needed.

Mid-level:

Use `LazyVGrid` for adaptive grids and `ScrollViewReader` for programmatic scrolling.

Senior:

I choose between `List`, `ScrollView`, lazy stacks, and grids based on interaction behavior, identity, performance, accessibility, and adaptive layout requirements.

## Practice

1. Build a product grid with adaptive columns.
2. Add pull-to-refresh to a lazy feed.
3. Build a chat screen that scrolls to the latest message.
4. Explain when `List` is better than `ScrollView`.
