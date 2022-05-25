---
title: "[iOS, Swift] Prevent ScreenShot"
excerpt: "Prevent ScreenShot"
description: "Prevent ScreenShot"
modified: 2022-05-25
categories: "iOS"
tags: [iOS, Swift, ScreenShot, UITextField]

toc: true

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# 1. Notification
- 스크린캡쳐 관련된 노티피케이션을 등록하여, 스크린샷 동작을 감지하여 얼럿을 띄울 수 있음
    - UIApplication.userDidTakeScreenshotNotification : 스크린샷 수행된 동작 감지
    - UIScreen.capturedDidChangeNotification : 화면 녹화 상태 변경 감지

```swift
extension UIViewController {
    @objc public func addPreventScreenShotNotification() {
        NotificationCenter
            .default
            .addObserver(self,
                         selector: #selector(preventScreenShot),
                         name: UIApplication.userDidTakeScreenshotNotification,
                         object: nil)

        NotificationCenter
            .default
            .addObserver(self,
                         selector: #selector(preventScreenShot),
                         name: UIScreen.capturedDidChangeNotification,
                         object: nil)
    }

    @objc public func removePreventScreenShotNotification() {
        NotificationCenter
            .default
            .removeObserver(self,
                            name: UIApplication.userDidTakeScreenshotNotification,
                            object: nil)

        NotificationCenter
            .default
            .removeObserver(self,
                            name: UIScreen.capturedDidChangeNotification,
                            object: nil)
    }

    @objc private func preventScreenShot() {
        // show a alert
    }
}
```

# 2. Set secureTextEntry in UITextField
- UITextField의 secure 기능을 우회하여, 스크린 캡쳐 감지시 해당 UIView를 숨길 수 있음
- iOS13 부터 UITextField에 secureTextEntry 값이 true 이면 스크린샷 동작시 해당 항목이 가려지는 기능이 추가됨
- 해당 기능을 이용하여, 스크린샷 방지가 필요한 UIView에 UITextField항목을 추가하여 사용 가능

```swift
import UIKit

class ViewController: UIViewController {

    lazy var testView: UIView = {
        let testView = UIView()
        testView.translatesAutoresizingMaskIntoConstraints = false
        testView.backgroundColor = UIColor.green

        self.view.addSubview(testView)

        testView.widthAnchor.constraint(equalToConstant: 100).isActive = true
        testView.heightAnchor.constraint(equalToConstant: 100).isActive = true
        testView.centerXAnchor.constraint(equalTo: self.view.centerXAnchor).isActive = true
        testView.centerYAnchor.constraint(equalTo: self.view.centerYAnchor).isActive = true

        return testView
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.

        testView.preventScreenShot()
    }
}

extension UIView {
    @objc public func preventScreenShot() {
        DispatchQueue.main.async {
            let backView                = UIView(frame: self.frame)
            backView.backgroundColor    = UIColor.lightGray
            backView.layer.cornerRadius = 5
            self.superview?.insertSubview(backView, at: 0)

            let sPreventField           = UITextField()
            sPreventField.isSecureTextEntry = true
            self.addSubview(sPreventField)

            sPreventField.centerYAnchor.constraint(equalTo: self.centerYAnchor).isActive = true
            sPreventField.centerXAnchor.constraint(equalTo: self.centerXAnchor).isActive = true

            self.layer.superlayer?.addSublayer(sPreventField.layer)
            sPreventField.layer.sublayers?.first?.addSublayer(self.layer)
        }
    }
}
```

| 테스트화면(녹색사각형 영역이 스크린 캡쳐 방지 영역) | 스크린 캡쳐 수행 | 스크린 캡쳐된 화면(해당 영역이 회색 영역으로 캡쳐) | 뷰계층구조 |
| --- | --- | --- | --- |
| ![testview](/assets/images/post/ios/preventscreenshot/1.png) |![testviewcapture](/assets/images/post/ios/preventscreenshot/2.png) | ![capturedview](/assets/images/post/ios/preventscreenshot/3.png)| ![viewhierarchy](/assets/images/post/ios/preventscreenshot/4.png)|

# Reference
- [https://green1229.tistory.com/169](https://green1229.tistory.com/169){:target="_blank"}
- [https://stackoverflow.com/questions/18680028/prevent-screen-capture-in-an-ios-app](https://stackoverflow.com/questions/18680028/prevent-screen-capture-in-an-ios-app){:target="_blank"}
- [https://littleshark.tistory.com/77](https://littleshark.tistory.com/77){:target="_blank"}
- [https://joonhyoung.github.io/swift/PreventScreenCapture/](https://joonhyoung.github.io/swift/PreventScreenCapture/){:target="_blank"}
