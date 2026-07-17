# Day 29: TCA Purpose, Core Concepts, And Mental Model

## What TCA Is

TCA means The Composable Architecture.

It is a Swift architecture library from Point-Free for building apps with:

- Explicit state management
- Actions
- Reducers
- Effects
- Dependencies
- Composition
- Strong testing

It works with SwiftUI, UIKit, and Apple platforms.

## Core TCA Concepts

State:

The data needed to render a feature and perform logic.

Action:

Everything that can happen in a feature.

Reducer:

The logic that mutates state for an action and returns effects.

Effect:

Work that talks to the outside world and can send actions back.

Store:

Runtime that holds state, runs reducer logic, and receives actions.

## Basic TCA Feature

```swift
import ComposableArchitecture

@Reducer
struct CounterFeature {
    @ObservableState
    struct State: Equatable {
        var count = 0
        var fact: String?
        var isLoading = false
    }

    enum Action {
        case decrementTapped
        case incrementTapped
        case factTapped
        case factResponse(Result<String, Error>)
    }

    @Dependency(\.numberFactClient) var numberFactClient

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .decrementTapped:
                state.count -= 1
                return .none

            case .incrementTapped:
                state.count += 1
                return .none

            case .factTapped:
                state.isLoading = true
                let count = state.count

                return .run { send in
                    await send(.factResponse(
                        Result { try await numberFactClient.fetch(count) }
                    ))
                }

            case .factResponse(.success(let fact)):
                state.isLoading = false
                state.fact = fact
                return .none

            case .factResponse(.failure):
                state.isLoading = false
                state.fact = "Unable to load fact."
                return .none
            }
        }
    }
}
```

View:

```swift
struct CounterView: View {
    let store: StoreOf<CounterFeature>

    var body: some View {
        Form {
            Text("\(store.count)")

            Button("Increment") {
                store.send(.incrementTapped)
            }

            Button("Decrement") {
                store.send(.decrementTapped)
            }

            Button("Number Fact") {
                store.send(.factTapped)
            }

            if store.isLoading {
                ProgressView()
            }

            if let fact = store.fact {
                Text(fact)
            }
        }
    }
}
```

## TCA Mental Model

TCA asks:

```text
What is the state?
What actions can happen?
How does each action change state?
What effects should run?
What actions can effects send back?
```

Everything important is visible in the reducer.

## Why TCA Exists

TCA helps with:

- Consistent state mutation
- Testable side effects
- Large feature composition
- Navigation state
- Dependency control
- Reducer-driven business logic
- Exhaustive user-flow tests

## When TCA Helps

Good fit:

- Complex state machines
- Many async effects
- Large modular app
- High test requirements
- Shared state across features
- Navigation modeled as data
- Team wants strong architectural consistency

## When TCA May Hurt

Weak fit:

- Tiny app
- Simple forms
- Team unfamiliar with reducer architecture
- Rapid prototype
- Low test investment
- Architecture ceremony not justified

## Senior Artifact

```text
Artifact: TCA Fit Check
Feature:
State complexity:
Actions:
Effects:
Dependencies:
Composition needs:
Testing requirements:
Team familiarity:
Boilerplate cost:
Decision:
```

## Common Mistakes

- Choosing TCA without committing to tests.
- Massive reducers.
- Actions too vague.
- State stores derived values unnecessarily.
- Effects that are hard to cancel.
- Dependencies not controlled in tests.
- Treating TCA as magic instead of architecture.

## Interview Notes

Junior:

TCA uses State, Action, Reducer, Effect, and Store.

Mid-level:

Views send actions to the store; reducers mutate state and return effects.

Senior:

I use TCA when explicit state/action/effect modeling and testability are worth the ceremony. It shines in complex flows, but it requires discipline around reducer size, dependency control, and composition.

## Practice

1. Build a counter feature with State, Action, Reducer, Store.
2. Add an async effect.
3. Identify all actions in a login flow.
4. Decide if a simple settings screen needs TCA.
