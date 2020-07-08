# Awesome Swift
A list of awesome helpers for Swift and iOS

## Generic Builder DSL

Usage:
```swift
lazy var label = UILabel()
    .. \.text <- "Hello, World!"
    .. \.font <- .systemFont(ofSize: 40)
    .. \.textColor <- .white
```

Implementation:
```swift
infix operator ..: AdditionPrecedence
infix operator <-: MultiplicationPrecedence

struct Predicate<Element> {
    let code: (Element) -> Element

    func runCode(for element: Element) -> Element {
        return code(element)
    }
}

func <- <Element, T>(_ attribute: WritableKeyPath<Element, T>,
                     _ constant: T) -> Predicate<Element> {
    return Predicate(code: { value in
        var copy = value
        copy[keyPath: attribute] = constant
        return copy
    })
}

protocol Builder {}

extension Builder {
    @discardableResult
    static func .. (_ element: Self,
                    _ predicate: Predicate<Self>) -> Self {
        return element.with(predicate)
    }

    private func with(_ predicate: Predicate<Self>) -> Self {
        return predicate.runCode(for: self)
    }
}

```
---
## Router/Coordinator pattern protocols

- [x] PresentableRouter
- [x] PushableRouter
- [ ] TabBarRouter
- [ ] TabPageRouter

### PresentableRouter

Usage:
1. Make your Router implement `PresentableRouter`
```swift
final class SomeModalRouter: PresentableRouter {
    // ...
    
}
```
2. Simply call its `start()` method
```swift
let router = SomeModalRouter(...)
router.start()
```

Implementation:
```swift
import UIKit

protocol PresentableRouter: ShowableRouter {
    var presentingViewController: UIViewController { get }
}

extension PresentableRouter {
    func show(viewController: UIViewController, animated: Bool) {
        currentViewController = viewController
        presentingViewController.present(viewController, animated: animated)
    }
}
```

Obs.: It requires `ShowableRouter` and `Router` protocols.

---

### PushableRouter

Usage:
1. Make your Router implement `PushableRouter`
```swift
final class SomeRouter: PushableRouter {
    // ...
    
}
```
2. Simply call its `start()` method
```swift
let router = SomeRouter(...)
router.start()
```

Implementation:
```swift
import UIKit

protocol PushableRouter: ShowableRouter {
    var presentingViewController: UINavigationController { get }
}

extension PushableRouter {
    func show(viewController: UIViewController, animated: Bool) {
        currentViewController = viewController
        presentingViewController.pushViewController(viewController, animated: animated)
    }
}
```

Obs.: It requires `ShowableRouter` and `Router` protocols.

--- 

### Router

A prerequisite for `ShowableRouter`.

Implementation:
```swift
import UIKit

protocol Router: class {
    var currentViewController: UIViewController? { get }
    func start()
    func start(animated: Bool)
}

extension Router {
    func start() {
        start(animated: true)
    }
}
```

### ShowableRouter

A prerequisite for `PushableRouter`, `PresentableRouter`, `RootRouter` and `TabPageRouter`

Implementation:
```swift
import UIKit

protocol ShowableRouter: Router {
    var currentViewController: UIViewController? { get set }
    func createViewController() -> UIViewController
    func show(viewController: UIViewController, animated: Bool)
}

extension ShowableRouter {
    func start(animated: Bool = true) {
        let viewController = createViewController()
        show(viewController: viewController, animated: animated)
    }
}
```
