---
title: "[iOS, SwiftUI] SiriKit (Shortcuts)"
excerpt: "SiriKit (Shortcuts) : Intents, IntentsUI, NSUserActivity"
description: "SiriKit (Shortcuts) : Intents, IntentsUI, NSUserActivity"
modified: 2021-07-08
categories: "iOS"
tags: [iOS, SwiftUI, SiriKit, Shortcuts, Intents, IntentsUI, NSUserActivity]

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# SiriKit

- Intents와 IntentsUI 프레임워크를 이용하여, 시리를 통해 디바이스와 상호작용을 할 수 있는 프레임워크

```swift
import Intents
import IntentsUI
```

## SiriKit Capability

- add SiriKit Capability into the project setting for using siri

![sirikitCapability](/assets/images/post/sirikit/sirikitcapability.png)

# Shortcuts

## NSUserActivity

- `NSUserActivity`를 이용하여 쉽게 단축어(Shortcuts)를 추가하고, 시리를 통하여 앱을 실행

![addsiri](/assets/images/post/sirikit/addsiri.png)
![shortcuts](/assets/images/post/sirikit/shortcuts.png)

### Set Shortcut to Siri Suggestions
```swift
import Intents

final class UserActivityShortcutsManager {

    public enum Shortcut: CaseIterable {
        case redview
        case blueview

        var type: String {

            switch self {
            case .redview:
                return "com.tigi44.shortcuts.redview"
            case .blueview:
                return "com.tigi44.shortcuts.blueview"
            }
        }

        var title: String {

            switch self {
            case .redview:
                return "레드뷰 실행"
            case .blueview:
                return "블루뷰 실행"
            }
        }

        var invocationPhrase: String {

            switch self {
            case .redview:
                return "레드뷰 보여줘"
            case .blueview:
                return "블루뷰 보여줘"
            }
        }

        var userActivity: NSUserActivity {

            let userActivity = NSUserActivity(activityType: self.type)
            userActivity.title = self.title
            userActivity.suggestedInvocationPhrase = self.invocationPhrase

            return userActivity
        }

        func makeShortcut() -> INShortcut {
            return INShortcut(userActivity: self.userActivity)
        }
    }

    static func setup() {

        var shortcuts: [INShortcut] = []

        for shortcut in Shortcut.allCases {
            shortcuts.append(shortcut.makeShortcut())
        }

        INVoiceShortcutCenter.shared.setShortcutSuggestions(shortcuts)
    }
}

// main

@main
struct SiriKitExampleApp: App {

    init() {
        UserActivityShortcutsManager.setup()
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// if use appDelegate..

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

  UserActivityShortcutsManager.setup()

  return true
}

```

- NSUserActivity를 이용하여 INShortcut을 만들고, 이를 INVoiceShortcutCenter.shared.setShortcutSuggestions(shortcuts)로 단축어에 설정
- 설정된 Shortcut은 단축어(Shortcuts)앱내의 Gallery에서 확인 및 사용 가능

![shortcutsgallery](/assets/images/post/sirikit/shortcutsgallery.png)

### Continue UserActivity
- 단축어(Shortcuts)로 실행된 내용을 SwiftUI에서 동작시키기 위해서는 View내의 `onContinueUserActivity`를 이용하여 쉽게 적용 가능
- Shortcut type에 따라 그에 맞는 onContinueUserActivity 설정

```swift
struct ContentView: View {

    @State var showRedView: Bool = false
    @State var showBlueView: Bool = false

    var body: some View {

        VStack(spacing: 100) {
            Button(action: {
                showRedView.toggle()
            }, label: {
                Label("Show a RedView", systemImage: "eye.circle.fill")
            })
            .foregroundColor(.red)

            Button(action: {
                showBlueView.toggle()
            }, label: {
                Label("Show a BlueView", systemImage: "eye.circle.fill")
            })
            .foregroundColor(.blue)
        }
        .sheet(isPresented: $showRedView, content: {
            RedView()
        })
        .sheet(isPresented: $showBlueView, content: {
            BlueView()
        })
        .onContinueUserActivity(UserActivityShortcutsManager.Shortcut.redview.type, perform: { userActivity in
            showRedView.toggle()
        })
        .onContinueUserActivity(UserActivityShortcutsManager.Shortcut.blueview.type, perform: { userActivity in
            showBlueView.toggle()
        })
    }
}
```

- 만약 `appDelegate` 혹은 `sceneDelegate`를 사용한다면 아래 함수에서 shortcut 동작을 실행 시키면 됨
- sceneDelegate를 이용하는 경우, 앱이 백그라운드로 실행중이지 않을 경우를 대비하여 `scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions)`에 동작을 추가해야 함

```swift
// appDelegate
func application(UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: [AnyObject]? -> Void) -> Bool {
  switch userActivity.activityType {
    ...
  }
}

// sceneDelegate
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
  guard let userActivity = connectionOptions.userActivities.first else { return }
  self.scene(scene, continue: userActivity)
}

func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
  switch userActivity.activityType {
    ...
  }
}
```

## SiriButton (INUIAddVoiceShortcutButton)
![blueview](/assets/images/post/sirikit/blueview.png)
![blueviewsiri](/assets/images/post/sirikit/blueviewsiri.png)

- Shortcuts 추가는 앱내의 해당 화면에서 SiriButton(INUIAddVoiceShortcutButton)을 통하여 가능
- SwiftUI에서는 UIViewControllerRepresentable를 이용하여 INUIAddVoiceShortcutButton 기능 추가

```swift
import SwiftUI
import IntentsUI

struct SiriButton: UIViewControllerRepresentable {
    public let shortcut: INShortcut

    func makeUIViewController(context: Context) -> SiriUIViewController {
        return SiriUIViewController(shortcut: shortcut)
    }

    func updateUIViewController(_ uiViewController: SiriUIViewController, context: Context) {
    }
}

class SiriUIViewController: UIViewController {
    let shortcut: INShortcut

    init(shortcut: INShortcut) {
        self.shortcut = shortcut
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        let button = INUIAddVoiceShortcutButton(style: .blackOutline)
        button.shortcut = shortcut

        self.view.addSubview(button)
        view.centerXAnchor.constraint(equalTo: button.centerXAnchor).isActive = true
        view.centerYAnchor.constraint(equalTo: button.centerYAnchor).isActive = true
        button.translatesAutoresizingMaskIntoConstraints = false

        button.delegate = self
    }
}

extension SiriUIViewController: INUIAddVoiceShortcutButtonDelegate {
    func present(_ addVoiceShortcutViewController: INUIAddVoiceShortcutViewController, for addVoiceShortcutButton: INUIAddVoiceShortcutButton) {
        addVoiceShortcutViewController.delegate = self
        addVoiceShortcutViewController.modalPresentationStyle = .formSheet
        present(addVoiceShortcutViewController, animated: true)
    }

    func present(_ editVoiceShortcutViewController: INUIEditVoiceShortcutViewController, for addVoiceShortcutButton: INUIAddVoiceShortcutButton) {
        editVoiceShortcutViewController.delegate = self
        editVoiceShortcutViewController.modalPresentationStyle = .formSheet
        present(editVoiceShortcutViewController, animated: true)
    }
}

extension SiriUIViewController: INUIAddVoiceShortcutViewControllerDelegate {
    func addVoiceShortcutViewController(_ controller: INUIAddVoiceShortcutViewController, didFinishWith voiceShortcut: INVoiceShortcut?, error: Error?) {
        controller.dismiss(animated: true)
    }

    func addVoiceShortcutViewControllerDidCancel(_ controller: INUIAddVoiceShortcutViewController) {
        controller.dismiss(animated: true)
    }
}

extension SiriUIViewController: INUIEditVoiceShortcutViewControllerDelegate {
    func editVoiceShortcutViewController(_ controller: INUIEditVoiceShortcutViewController, didUpdate voiceShortcut: INVoiceShortcut?, error: Error?) {
        controller.dismiss(animated: true)
    }

    func editVoiceShortcutViewController(_ controller: INUIEditVoiceShortcutViewController, didDeleteVoiceShortcutWithIdentifier deletedVoiceShortcutIdentifier: UUID) {
        controller.dismiss(animated: true)
    }

    func editVoiceShortcutViewControllerDidCancel(_ controller: INUIEditVoiceShortcutViewController) {
        controller.dismiss(animated: true)
    }
}

// add a siributton into a blueview

import SwiftUI
import Intents

struct BlueView: View {

    let shortcut: INShortcut = UserActivityShortcutsManager.Shortcut.blueview.makeShortcut()

    var body: some View {
        ZStack {
            Color.blue
                .ignoresSafeArea()

            SiriButton(shortcut: shortcut).frame(height: 34)
        }
    }
}
```

## Intents


![testshortcuts](/assets/images/post/sirikit/testshortcuts.png)
![intentview](/assets/images/post/sirikit/intentview.png)
![shortcutsapp](/assets/images/post/sirikit/shortcutsapp.png)

- Intents를 사용할 경우, Shortcuts 앱내의 `Add Action` 메뉴에서 사용이 가능해짐
- Intents를 통해 추가 파라미터등을 입력받을 수 있는 등, 다양한 방식으로 Shortcut 활용이 가능

### Create a SiriKit Intent Definition File

![siriintentdefinition](/assets/images/post/sirikit/siriintentdefinition.png)

- Add `ShowIntentViewIntent`
![showintentviewinent](/assets/images/post/sirikit/showintentviewinent.png)
- Custom Intent (Category)
[https://developer.apple.com/design/human-interface-guidelines/siri/overview/custom-intents/](https://developer.apple.com/design/human-interface-guidelines/siri/overview/custom-intents/){:target="_blank"}

- Add Text Parameter into the intent
![intentparam](/assets/images/post/sirikit/intentparam.png)

- Set a Parameter into Shortcuts App and Siri Suggestions
![intentshortcut](/assets/images/post/sirikit/intentshortcut.png)

### Continue UserActivity
- UserActivityType은 Intent Definition 생성시 info.plist 파일에 자동 생성
- `ShowIntentViewIntent`에서 text 파라미터를 받을 수 있기때문에, userActivity에서 text 파라미터를 가져와 사용

```swift
struct ContentView: View {

    ...

    var body: some View {

        ...

        .onContinueUserActivity("ShowIntentViewIntent", perform: { userActivity in

            if let intent = userActivity.interaction?.intent
                as? ShowIntentViewIntent {
                intentViewText = intent.text
            }

            showIntentView.toggle()
        })
    }
}
```


# Source Code
- [https://github.com/tigi44/SiriKitExample](https://github.com/tigi44/SiriKitExample){:target="_blank"}

# Reference
- [https://zeddios.tistory.com/1289](https://zeddios.tistory.com/1289){:target="_blank"}
- [https://swiftui-lab.com/nsuseractivity-with-swiftui/](https://swiftui-lab.com/nsuseractivity-with-swiftui/){:target="_blank"}
- [https://blog.devgenius.io/how-to-add-siri-shortcuts-in-your-app-in-swift-7afb61934c4e](https://blog.devgenius.io/how-to-add-siri-shortcuts-in-your-app-in-swift-7afb61934c4e){:target="_blank"}
- [https://toolboxpro.app/blog/adding-shortcuts-to-an-app-1](https://toolboxpro.app/blog/adding-shortcuts-to-an-app-1){:target="_blank"}
