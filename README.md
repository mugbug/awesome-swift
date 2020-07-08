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
