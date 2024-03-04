---
title: "[iOS, Swift] Prevent ScreenShot on iOS17"
excerpt: "Prevent ScreenShot on iOS17"
description: "Prevent ScreenShot on iOS17"
modified: 2024-02-22
categories: "iOS"
tags: [iOS, Swift, ScreenShot, UITextField, iOS17]

toc: false

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# Prevent ScreenShot ~ iOS16
- [\[iOS, Swift\] Prevent ScreenShot](/ios/iOS,-Swift-Prevent-Screen-Capture/){: target="_blank"}

# on iOS17
```swift
extension UIView {
    @available(iOS 17.0, *)
    @objc public func preventScreenShot_IOS17() {
        DispatchQueue.main.async {
          let sPreventField               = UITextField()
          sPreventField.isSecureTextEntry = true
          self.addSubview(sPreventField)

          self.layer.superlayer?.addSublayer(sPreventField.layer)
          sPreventField.layer.sublayers?.last?.addSublayer(self.layer)
        }
    }
}
```

# as UIView
```swift
private class SecureField : UITextField {
    override init(frame: CGRect) {
        super.init(frame: .zero)
        self.isSecureTextEntry = true
        self.translatesAutoresizingMaskIntoConstraints = false
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    weak var secureContainer: UIView? {
        let secureView = self.subviews.filter({ subview in
            type(of: subview).description().contains("CanvasView")
        }).first
        secureView?.translatesAutoresizingMaskIntoConstraints = false
        secureView?.isUserInteractionEnabled = true
        return secureView
    }

    override var canBecomeFirstResponder: Bool {false}
    override func becomeFirstResponder() -> Bool {false}
}

open class SecureView: UIView {
    private lazy var secureField: UIView = {
        var secureField: UIView = UIView()
        secureField.translatesAutoresizingMaskIntoConstraints = false

        if let secureContainer = SecureField().secureContainer {
            secureField = secureContainer
        }

        addSubview(secureField)
        NSLayoutConstraint.activate([
            secureField.topAnchor.constraint(equalTo: self.topAnchor),
            secureField.leadingAnchor.constraint(equalTo: self.leadingAnchor),
            secureField.bottomAnchor.constraint(equalTo: self.bottomAnchor),
            secureField.trailingAnchor.constraint(equalTo: self.trailingAnchor),
        ])

        return secureField
    }()

    private lazy var preventLabel: UILabel = {
        let preventLabel = PreventScreenShotLabel()
        preventLabel.translatesAutoresizingMaskIntoConstraints = false

        addSubview(preventLabel)
        NSLayoutConstraint.activate([
            preventLabel.topAnchor.constraint(equalTo: self.topAnchor),
            preventLabel.leadingAnchor.constraint(equalTo: self.leadingAnchor),
            preventLabel.bottomAnchor.constraint(equalTo: self.bottomAnchor),
            preventLabel.trailingAnchor.constraint(equalTo: self.trailingAnchor),
        ])

        return preventLabel
    }()

    public required init() {
        super.init(frame: .zero)

        _ = preventLabel
        _ = secureField
    }

    public required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

public extension SecureView {
    func addSecureSubview(_ view: UIView) {
        secureField.addSubview(view)
    }

    var secureSubviews: [UIView] {
        return secureField.subviews
    }
}

// let view = SecureView()
// view.translatesAutoresizingMaskIntoConstraints = false
// view.addSecureSubview(subView)
```

# Reference
- [https://medium.com/@lakshimi.cg/screenshot-prevention-in-ios-f059dc82b046](https://medium.com/@lakshimi.cg/screenshot-prevention-in-ios-f059dc82b046){: target="_blank"}
