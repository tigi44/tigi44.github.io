---
title: "[iOS, SwiftUI] Spotlight"
excerpt: "Spotlight : Searchable Item"
description: "Spotlight : Searchable Item"
modified: 2021-07-15
categories: "iOS"
tags: [iOS, SwiftUI, Swift, Spotlight]

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# Spotlight
- 앱내 정보를 iOS상에 등록하여, iOS의 spotlight에서 검색 가능하도록 하며, 검색결과 클릭으로 해당 앱의 특정 동작을 시킬수 있는 기능
![spotlight](/assets/images/post/ios/spotlight/spotlight.png)

## SwiftUI
### Add
```swift
...
.userActivity(aType) { userActivity in
    userActivity.isEligibleForSearch = true
    userActivity.title = "\(device.name)"
    userActivity.userInfo = ["deviceId": device.id]

    let attributes = CSSearchableItemAttributeSet(itemContentType: kUTTypeItem as String)

    attributes.contentDescription = "Get a Greate Device!"
    attributes.thumbnailData = UIImage(systemName: device.image)?.pngData()
    userActivity.contentAttributeSet = attributes

    print("Advertising: \(device.name)")
}
```

### Delete
```swift
Button("AllDelete") {
    NSUserActivity.deleteAllSavedUserActivities {
        print("done!")
    }
}
```

### ContinueUserActivity
```swift
...
.onContinueUserActivity(aType, perform: { userActivity in
    if let deviceId = userActivity.userInfo?["deviceId"] as? NSNumber {
        selection = deviceId.intValue
    }
})
```

## UIKit
### Add

```swift
import CoreSpotlight
import CoreServices

...

private func addItem() {
    let attributeSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeText as String)
    attributeSet.title = "Macpro"
    attributeSet.contentDescription = "Get a Greate Device!"
    attributeSet.thumbnailData = UIImage(systemName: "macpro.gen1")?.pngData()

    let item = CSSearchableItem(uniqueIdentifier: "macpro", domainIdentifier: "com.tigi44.spotlightExample", attributeSet: attributeSet)
    CSSearchableIndex.default().indexSearchableItems([item]) { error in
        if let error = error {
            print("Indexing error: \(error.localizedDescription)")
        } else {
            print("Search item successfully indexed!")
        }
    }
}
```

### Delete
```swift
// delete a item
@objc private func deleteItem() {
    CSSearchableIndex.default().deleteSearchableItems(withIdentifiers: ["macpro"]) { error in
        if let error = error {
            print("Deindexing error: \(error.localizedDescription)")
        } else {
            print("Search item successfully removed!")
        }
    }
}

// delete all item
@objc func allDelete() {
    NSUserActivity.deleteAllSavedUserActivities {
        print("done!")
    }
}    
```

### Move into the app
```swift

// SceneDelegate

func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {

    guard let _ = (scene as? UIWindowScene) else { return }

    // 앱이 실행중이지 않을 경우, 해당 scene을 찾을수 없기에 꼭 추가해야 함
    guard let userActivity = connectionOptions.userActivities.first else { return }
    self.scene(scene, continue: userActivity)
}


func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {

    if userActivity.activityType == CSSearchableItemActionType {

        if let uniqueIdentifier = userActivity.userInfo?[CSSearchableItemActivityIdentifier] as? String {

           // uniqueIdentifier에 따른 앱 동작 추가
        }
    }
}   
```

# Source Code
- [https://github.com/tigi44/SpotlightExample](https://github.com/tigi44/SpotlightExample){:target="_blank"}

# Reference
- [https://swiftui-lab.com/nsuseractivity-with-swiftui/](https://swiftui-lab.com/nsuseractivity-with-swiftui/){:target="_blank"}
- [https://www.hackingwithswift.com/example-code/system/how-to-use-core-spotlight-to-index-content-in-your-app](https://www.hackingwithswift.com/example-code/system/how-to-use-core-spotlight-to-index-content-in-your-app){:target="_blank"}
