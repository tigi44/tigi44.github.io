---
title: "[iOS, SwiftUI] onAppear called twice on a NavigationView in a TabView"
excerpt: "onAppear called twice on a NavigationView in a TabView"
description: "onAppear called twice on a NavigationView in a TabView"
modified: 2021-06-29
categories: "iOS"
tags: [iOS, SwiftUI, TabView, NavigationView]

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# iOS 14.x
- onAppear called twice on a NavigationView in a TabView

```swift
TabView {
    NavigationView {
        Color.red
            .onAppear {
                print("appear : red")
            }
            .onDisappear {
                print("disappear : red")
            }
    }
}
```

```
appear : red
disappear : red
appear : red
```

# iOS 15.0
- `onAppear called twice on a NavigationView in a TabView` issue is resolved on iOS15, but onAppear of a ZStack in a NavigationView in a TabView is called twice

```swift
TabView {
    NavigationView {
        ZStack {
            Color.red
                .onAppear {
                    print("appear : red")
                }
                .onDisappear {
                    print("disappear : red")
                }
        }
    }
}
```

```
appear : red
disappear : red
appear : red
```

# Reference
- [https://stackoverflow.com/questions/67483863/swiftui-onappear-called-twice-when-navigationview-inside-tabview](https://stackoverflow.com/questions/67483863/swiftui-onappear-called-twice-when-navigationview-inside-tabview){:target="_blank"}
