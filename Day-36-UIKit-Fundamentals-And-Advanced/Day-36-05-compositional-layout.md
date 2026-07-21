# Day 36.5: Compositional Layout

Compositional layout is the modern layout system for collection views. It lets you build adaptive, section-based layouts by composing items, groups, sections, supplementary views, and layout configuration. Apple describes it as flexible and fast, with sections made from groups and groups made from items.

## Why Compositional Layout Matters

Before compositional layout, many complex screens required:

- Custom `UICollectionViewLayout`.
- Multiple nested collection views.
- Manual layout math.
- Fragile size calculations.

Compositional layout makes screens like App Store, dashboards, media grids, and carousels much easier to build.

## Core Concepts

- Item: the smallest visual unit.
- Group: arranges items horizontally, vertically, or custom.
- Section: contains groups and section behavior.
- Layout: combines sections.
- Supplementary item: header/footer/badge.
- Decoration item: background decoration managed by layout.

## Basic Grid

```swift
func makeGridLayout() -> UICollectionViewLayout {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .estimated(180)
    )
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = NSDirectionalEdgeInsets(top: 6, leading: 6, bottom: 6, trailing: 6)

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .estimated(180)
    )
    let group = NSCollectionLayoutGroup.horizontal(
        layoutSize: groupSize,
        subitem: item,
        count: 2
    )

    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 16, leading: 16, bottom: 16, trailing: 16)

    return UICollectionViewCompositionalLayout(section: section)
}
```

## Dimension Types

`NSCollectionLayoutDimension` supports:

- `.absolute(44)`: fixed size.
- `.estimated(120)`: content-driven size.
- `.fractionalWidth(0.5)`: relative to container width.
- `.fractionalHeight(1.0)`: relative to container height.

Examples:

```swift
let sidebarWidth: NSCollectionLayoutDimension = .absolute(320)
let cardHeight: NSCollectionLayoutDimension = .estimated(180)
let halfWidth: NSCollectionLayoutDimension = .fractionalWidth(0.5)
```

Senior thinking:

- Use estimated height for text-heavy self-sizing content.
- Use fractional width for adaptive grids.
- Use absolute sizes for controls or small badges.
- Be careful with estimated sizes in very large lists because measurement has a cost.

## Different Layout Per Section

```swift
enum HomeSection: Int, CaseIterable {
    case hero
    case categories
    case products
}

func makeLayout() -> UICollectionViewLayout {
    UICollectionViewCompositionalLayout { sectionIndex, environment in
        guard let section = HomeSection(rawValue: sectionIndex) else {
            return nil
        }

        switch section {
        case .hero:
            return Self.makeHeroSection()
        case .categories:
            return Self.makeHorizontalCategorySection()
        case .products:
            return Self.makeProductGridSection(environment: environment)
        }
    }
}
```

This is the main power of compositional layout: one collection view, multiple section designs.

## Adaptive Layout With Environment

```swift
static func makeProductGridSection(environment: NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection {
    let width = environment.container.effectiveContentSize.width
    let columns = width > 700 ? 4 : 2

    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1),
        heightDimension: .estimated(220)
    )
    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1),
        heightDimension: .estimated(220)
    )
    let group = NSCollectionLayoutGroup.horizontal(
        layoutSize: groupSize,
        subitem: item,
        count: columns
    )

    let section = NSCollectionLayoutSection(group: group)
    section.interGroupSpacing = 12
    section.contentInsets = NSDirectionalEdgeInsets(top: 12, leading: 16, bottom: 24, trailing: 16)
    return section
}
```

This helps support iPhone, iPad, Stage Manager, split view, and rotation.

## Orthogonal Scrolling

Horizontal sections inside a vertical collection view:

```swift
section.orthogonalScrollingBehavior = .groupPagingCentered
```

Options include:

- `.continuous`
- `.continuousGroupLeadingBoundary`
- `.paging`
- `.groupPaging`
- `.groupPagingCentered`

Use cases:

- Featured products.
- Media carousels.
- Category chips.
- Onboarding pages inside a larger screen.

Senior caution: nested horizontal scrolling can hurt discoverability and accessibility if overused.

## Headers And Supplementary Views

```swift
let headerSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1.0),
    heightDimension: .estimated(44)
)
let header = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: headerSize,
    elementKind: UICollectionView.elementKindSectionHeader,
    alignment: .top
)
section.boundarySupplementaryItems = [header]
```

Register supplementary view:

```swift
let headerRegistration = UICollectionView.SupplementaryRegistration<HeaderView>(
    elementKind: UICollectionView.elementKindSectionHeader
) { header, _, indexPath in
    header.title = "Recommended"
}
```

## List Layout

Modern collection views can render table-like lists.

```swift
var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.showsSeparators = true

let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

Use this when you want modern list behavior with collection view APIs.

## Badges

```swift
let badgeSize = NSCollectionLayoutSize(
    widthDimension: .absolute(24),
    heightDimension: .absolute(24)
)
let badge = NSCollectionLayoutSupplementaryItem(
    layoutSize: badgeSize,
    elementKind: "badge",
    containerAnchor: NSCollectionLayoutAnchor(
        edges: [.top, .trailing],
        fractionalOffset: CGPoint(x: 0.3, y: -0.3)
    )
)
item.supplementaryItems = [badge]
```

Use for unread counts, sale markers, or status indicators.

## Compositional Layout With Diffable

The strongest UIKit list architecture is:

- Display state produces rows.
- Diffable data source applies snapshot.
- Compositional layout decides visual arrangement.
- Cell registration configures views.

This keeps data, layout, and rendering separate.

## Performance Considerations

- Prefer one collection view over nested collection views.
- Use estimated dimensions only where content needs it.
- Avoid heavy work in layout section provider.
- Cache expensive layout decisions if needed.
- Test rotation and iPad multitasking widths.
- Use prefetching for images and data.

## Common Interview Traps

- Confusing item/group/section.
- Hard-coding columns without considering iPad.
- Using estimated dimensions everywhere.
- Nesting collection views unnecessarily.
- Forgetting supplementary registration.
- Overusing horizontal carousels.

## Junior-Level Interview Answer

Compositional layout builds collection view layouts from items, groups, and sections. It helps create grids, lists, and horizontal sections without writing a custom layout.

## Senior-Level Interview Answer

I use compositional layout to model each screen section independently. I adapt columns using `NSCollectionLayoutEnvironment`, combine it with diffable data source, use list configuration for settings-style screens, add supplementary headers through registrations, and profile estimated-size layouts in large feeds.

## Points To Remember

- Item goes inside group.
- Group goes inside section.
- Section goes inside layout.
- Use environment for adaptive layout.
- Orthogonal scrolling is powerful but easy to overuse.
- Modern UIKit collection views can replace many table view screens.

