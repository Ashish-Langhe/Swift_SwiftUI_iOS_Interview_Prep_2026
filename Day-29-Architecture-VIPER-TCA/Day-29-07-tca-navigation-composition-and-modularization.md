# Day 29: TCA Navigation, Composition, And Modularization

## Why Composition Matters

TCA is built for composing small features into larger ones.

Large apps need:

- Parent-child features
- Optional destinations
- Stack navigation
- Shared dependencies
- Modular reducers
- Testable integration flows

## Parent And Child Feature

Child:

```swift
@Reducer
struct ProductDetailsFeature {
    @ObservableState
    struct State: Equatable {
        let productID: Product.ID
        var product: Product?
    }

    enum Action {
        case task
        case productResponse(Result<Product, Error>)
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .task:
                return .none
            case .productResponse:
                return .none
            }
        }
    }
}
```

Parent:

```swift
@Reducer
struct ProductsFeature {
    @ObservableState
    struct State: Equatable {
        var products: [Product] = []
        var path = StackState<Path.State>()
    }

    enum Action {
        case productTapped(Product.ID)
        case path(StackAction<Path.State, Path.Action>)
    }

    @Reducer
    enum Path {
        case details(ProductDetailsFeature)
    }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .productTapped(let id):
                state.path.append(.details(ProductDetailsFeature.State(productID: id)))
                return .none

            case .path:
                return .none
            }
        }
        .forEach(\.path, action: \.path)
    }
}
```

## View Navigation

```swift
NavigationStack(
    path: $store.scope(state: \.path, action: \.path)
) {
    ProductsView(store: store)
} destination: { store in
    switch store.case {
    case .details(let store):
        ProductDetailsView(store: store)
    }
}
```

Exact syntax can evolve with TCA releases, but the model stays the same: navigation is state.

## Optional Destinations

Use optional state for sheets/dialog flows.

```swift
@Presents var destination: Destination.State?

enum Action {
    case addButtonTapped
    case destination(PresentationAction<Destination.Action>)
}

@Reducer
enum Destination {
    case addProduct(AddProductFeature)
}
```

Presentation becomes testable state.

## Modularization

Feature modules can own:

- State
- Action
- Reducer
- View
- Tests
- Test dependencies

Example:

```text
ProductsFeature/
  ProductsFeature.swift
  ProductsView.swift
  ProductsFeatureTests.swift
```

Larger apps may split features into Swift packages.

## Shared State

Avoid one giant root state. Share carefully through:

- Parent ownership
- Dependencies
- Identified arrays
- Shared persistence
- Dedicated app/session features

Senior warning: global shared mutable state can make TCA less predictable.

## Composition Benefits

- Child features are isolated.
- Parent controls navigation.
- Tests can target one feature or integrated flow.
- Dependencies are centrally controlled.
- Complex apps stay structured.

## Common Mistakes

- Massive root reducer.
- Child feature knows too much about parent.
- Navigation state duplicated outside TCA.
- Shared state mutated from many places.
- Over-splitting tiny components.
- No integration tests for composed flows.

## Senior Artifact

```text
Artifact: TCA Composition Plan
Root feature:
Child features:
Navigation stack:
Sheets/destinations:
Shared state:
Dependencies:
Package/module boundaries:
Unit tests:
Integration tests:
Risks:
```

## Interview Notes

Junior:

TCA can combine small features into bigger features.

Mid-level:

Parent reducers can compose child reducers and model navigation with state.

Senior:

I use TCA composition to keep domains isolated, navigation state explicit, and integration flows testable. I avoid massive roots and uncontrolled shared state.

## Practice

1. Create a parent product list and child details feature.
2. Model details navigation as stack state.
3. Add optional sheet destination.
4. Explain why navigation as state is testable.
