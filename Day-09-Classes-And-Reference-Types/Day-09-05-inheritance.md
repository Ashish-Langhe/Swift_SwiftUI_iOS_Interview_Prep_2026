# Day 9: Inheritance

## Core Idea

Inheritance lets a class reuse and specialize another class.

```swift
class Animal {
    func speak() {
        print("Sound")
    }
}

class Dog: Animal {
    override func speak() {
        print("Bark")
    }
}
```

## Real iOS Use Cases

- UIKit subclasses
- Custom view controllers
- Custom views

## Interview Levels

Junior:

Inheritance lets one class inherit from another.

Senior:

Use inheritance when there is a true subtype relationship or framework requirement. Prefer composition for app architecture when behavior can be assembled without rigid hierarchy.

## Quick Notes

- Classes support inheritance
- Structs do not
- Use `override`
- Prefer composition unless inheritance fits

## Interview Depth

Junior answer: Inheritance lets one class get properties and methods from another class.

Mid-level answer: Subclasses can specialize superclass behavior. UIKit uses inheritance heavily through view controllers and views.

Senior answer: Inheritance creates strong coupling. Use it for true subtype relationships or framework contracts. Prefer composition for flexible app architecture.

iOS use case:

```swift
class BaseViewController: UIViewController {
    func applyTheme() { }
}
```

Common mistakes:

- Deep inheritance trees.
- Overriding without understanding superclass contract.
- Using inheritance for code reuse when composition is clearer.

Practice:

1. Create superclass/subclass.
2. Override method.
3. Explain inheritance vs composition.
