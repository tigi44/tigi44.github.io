---
title: "[WWDC18] What`s New in User Notifications"
excerpt: "[WWDC18] Behind What`s New in User Notifications"
description: "[WWDC18] What`s New in User Notifications"
modified: 2018-08-31
categories: "WWDC18"
tags: [WWDC18, Notifications]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/wwdc18-teaser.jpg
---

# What\`s New in User Notifications

## Grouped notifications
![스크린샷 2018-09-07 오전 11.20.49.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-09-07 오전 11.20.49.png)
![스크린샷 2018-08-29 오후 5.50.49.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-29 오후 5.50.49.png)

### Automatic grouping
- 기본적으로 별다른 설정이 없다면 같은 앱을 기준으로 그룹핑

### Thread identifier
- 특정 threadIdentifier 설정으로 그룹핑할 수 있음

```swift
// Grouped Notifications
// Thread identifier

let content = UNMutableNotificationContent()
content.title = "New Photo"
content.body = "Jane Doe posted a new photo"
content.threadIdentifier = "thread-identifier"

// PUSH
{
    "aps" : {
        "alert" : {
            "title" : "New Photo",
            "body" : "Jane Doe posted a new photo",
            "thread-id" : "thread-identifier",
        },
    },
}
```

### User setting
- 그룹핑은 각앱의 알림 설정에서 지정 가능(Automatic / ByApp / Off)
- Automatic 설정시 threadIdentifier 설정이 있다면 해당 threadIdentifier에 따른 그룹핑, 그외에는 기본적으로 앱에 따른 그룹핑

### Summary text
- 알림의 그룹핑 내용을 요약해서 보여주는 텍스트 부분 설정 가능

![스크린샷 2018-08-29 오후 6.01.28.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-29 오후 6.01.28.png)

## Notification content extensions
- Make New a Project : notification content extension

```swift
// Notification Content Extensions

import UserNotifications
import UserNotificationsUI

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    func didReceive(_ notification: UNNotification) {
        // Set up content extension view
    }
}
```
![스크린샷 2018-08-29 오후 6.04.19.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-29 오후 6.04.19.png)

### Actions
- 사용자가 응답할 수 있는 액션영역 추가

![스크린샷 2018-08-29 오후 6.05.44.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-29 오후 6.05.44.png)

```swift
// Notification Content Extensions
// Actions

let likeAction = UNNotificationAction(identifier: "like-action",
                                                                        title: "Like",
                                                                options: [])

let commentAction = UNTextInputNotificationAction(identifier: "comment-action",
                                                                                            title: "Comment",
                                                                                            options: [],
                                                                                            textInputButtonTitle: "Comment",
                                                                                            textInputPlaceholder: "Type here...")

let category = UNNotificationCategory(identifier: "extension-example",
                                                                    actions: [ likeAction, commentAction ],
                                                                    intentIdentifiers: [],
                                                                    options: [])


UNUserNotificationCenter.current().setNotificationCategories([ category ])
```

```swift
// Notification Content Extensions
// Actions handling

import UserNotificationsUI

class NotificationViewController: UIViewController, UNNotificationContentExtension {
        @IBOutlet var likeLabel: UILabel?
        func didReceive(_ response: UNNotificationResponse, completionHandler completion:
        (UNNotificationContentExtensionResponseOption) -> Void) {
            if response.actionIdentifier == "like-action" {
                likeLabel?.text = "You liked this photo"
                likedPhoto()
            }
            completion(.doNotDismiss)
        }
}
```

### User interations
- 알림 화면 터치등의 사용자 인터랙션 추가
![스크린샷 2018-08-29 오후 6.14.48.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-29 오후 6.14.48.png)
![스크린샷 2018-11-26 오후 2.52.13.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-11-26 오후 2.52.13.png)

```swift
// Notification Content Extensions
// User interaction

import UserNotificationsUI

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    @IBOutlet var likeButton: UIButton?
    ...
    likeButton?.addTarget(self, action: #selector(likeButtonTapped), for: .touchUpInside)
    ...
    @objc func likeButtonTapped() {
        likeButton?.setTitle("♥", for: .normal)
        likedPhoto()
    }
}
```

### Launch application
 - 알림에 대한 앱을 실행할 수 있는 기본 액션 제공

```swift
// Notification Content Extensions
extension NSExtensionContext {

    @available(iOS 12.0, *)
    func performNotificationDefaultAction()
}

...


// Notification Content Extensions
// Default action

import UserNotificationsUimport UserNotificationsU`I

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    @IBOutlet var allCommentsButton: UIButton?
    ...
    allCommentsButton?.addTarget(self, action: #selector(launchApp), for: .touchUpInside)
    ...
    @objc func launchApp() {
        extensionContext?.performNotificationDefaultAction()
    }
}
```

### Dismiss content extension view
- 알림은 dismiss할 수 있는 기본 액션 제공

```swift
// Notification Content Extensions

extension NSExtensionContext {

    @available(iOS 12.0, *)
    func dismissNotificationContentExtension()
}

...

// Notification Content Extensions
// User interaction

import UserNotificationsUimport UserNotificationsU`I

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    @IBOutlet var likeButton: UIButton?
    ...
    likeButton?.addTarget(self, action: #selector(likeButtonTapped), for: .touchUpInside)
    ...
    @objc func likeButtonTapped() {
        likedPhoto()
        extensionContext?.dismissNotificationContentExtension()
    }
}
```

## Notification management
- 알림센터에서 알림 관리를 할 수 있다.
- 알림을 켜고 끄고는 물론이고, 조용히(?)모드 앱내의 개별 알림 설정등등의 메뉴 제공

| management  |turn off  |setting  | setting  notifications on app |
| --- | --- | --- | --- |
| ![스크린샷 2018-08-30 오전 10.13.10.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.13.10.png) | ![스크린샷 2018-08-30 오전 10.13.28.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.13.28.png) |![스크린샷 2018-08-30 오전 10.13.40.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.13.40.png)  | ![스크린샷 2018-08-30 오전 10.45.24.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.45.24.png)|

```swift
// Custom Settings Link

 import UIKit
 import UserNotifications

class AppDelegate: UIApplicationDelegate, UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                    openSettingsFor notification: UNNotification? ) {
    }
}
```

## Provisional authorization
- 알림센터에서 해당 앱의 알림에 대한 허가 옵션을 관리할 수 있다.

| as-is |new  |keep actions  |turn off actions  |
| --- | --- | --- | --- |
| ![스크린샷 2018-08-30 오전 10.16.38.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.16.38.png) | ![스크린샷 2018-08-30 오전 10.17.08.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.17.08.png) | ![스크린샷 2018-08-30 오전 10.17.20.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.17.20.png) | ![스크린샷 2018-08-30 오전 10.17.27.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.17.27.png) |

## Critical alerts
- 긴급상황시 별도의 얼럿이 필요하다.
- 사용자의 액션이 필요한 상황의 얼럿 : medical and health , home and security , public safety
- 방해금지모드나 음소거모드등에 상관 없어야하고, 소리가 나야하며, 다른업무중일때도 알아차릴 수 있어야 함


|new alert| notification | setting |
| --- | --- | --- |
|![스크린샷 2018-08-30 오전 10.48.09.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.48.09.png)| ![스크린샷 2018-08-30 오전 10.29.50.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.29.50.png) |  ![스크린샷 2018-08-30 오전 10.29.58.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.29.58.png)|

```swift
// Critical alert with default sound

let content = UNMutableNotificationContent()
content.title = "WARNING: LOW BLOOD SUGAR"
content.body = "Glucose level at 57."
content.categoryIdentifier = "low-glucose—alert"
content.sound = UNNotificationSound.defaultCritical
//content.sound = UNNotificationSound.criticalSoundNamed(@"warning-sound" withAudioVolume: 1.00)
```

```swift
// Critical alert push payload

{
    "aps" : {
        "sound" : {
            "critical": 1,
            "name": "warning-sound.aiff",
            "volume": 1.0
        }
    }
}
```

# Using Grouped Notifications

## Notification groups

|  |  |  |
| --- | --- | --- |
| ![스크린샷 2018-08-30 오전 10.59.45.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.59.45.png) | ![스크린샷 2018-08-30 오전 10.59.53.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.59.53.png) | ![스크린샷 2018-08-30 오전 10.59.59.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 10.59.59.png) |

## App grouping
- 앱 기준 그룹핑

## Custom grouping
- 쓰레드 설정으로 앱내의 커스텀 그룹핑

![스크린샷 2018-08-30 오전 11.02.36.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 11.02.36.png)

- 캘린더 : 중요하고 별도의 동작이 필요한 알림은 정보성 알림과 구분
- 메세지 : 의미있고 개인적인 그룹을 별도로 구분
- 메일 : 사용자의 개인적인 우선순위와 조직구조등을 존중하여 구분

## Group summaries
- 그룹핑된 항목의 요약 정보를 별도로 설정 할 수 있다.

```swift
// Simple Notification Group Summary

let summaryFormat = "%u more messages"

return UNNotificationCategory(identifier: "category-identifier",
                                                    actions: [],
                                                    intentIdentifiers: [],
                                                    hiddenPreviewsBodyPlaceholder: nil,
                                                    categorySummaryFormat: summaryFormat,
                                                    options: [])
```

```swift
// Notification Group Summary with Arguments

let summaryFormat = "%u more messages from %@"

return UNNotificationCategory(identifier: "group-messages",
                                                    actions: [],
                                                    intentIdentifiers: [],
                                                    hiddenPreviewsBodyPlaceholder: nil,
                                                    categorySummaryFormat: summaryFormat,
                                                    options: [])

...

// Notification Group Summary Argument

let content = UNMutableNotificationContent()

content.body = "..."
content.threadIdentifier = "notifications-team"
content.summaryArgument = "Kritarth"

// Notification Group Summary Argument
{
    "aps" : {
        "alert" : {
            "body" : "...",
            "summary-arg" : "Kritarth"
        },
        "thread-id" : "notifications-team"
    }
}
```

### Summary Plurals and Localization
- 요약정보에 노출할 단수/복수 및 지역화를 설정할 수 있다.

|  |  |  |
| --- | --- | --- |
| ![스크린샷 2018-08-30 오후 12.18.38.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오후 12.18.38.png) | ![스크린샷 2018-08-30 오후 12.18.44.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오후 12.18.44.png) | ![스크린샷 2018-08-30 오후 12.18.52.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오후 12.18.52.png) |

```swift
// Summary Localization

let summaryFormat = NSString.localizedUserNotificationString(forKey: "NOTIFICATION_SUMMARY",
arguments: nil)

return UNNotificationCategory(identifier: "category-identifier",
                                                    actions: [],
                                                    intentIdentifiers: [],
                                                    hiddenPreviewsBodyPlaceholder: nil,
                                                    categorySummaryFormat: summaryFormat,
                                                    options: [])
```

```swift
<!-- Summary Localization -->
<plist version="1.0">
<dict>
    <key>NOTIFICATION_SUMMARY</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%#@notifications@</string>
        <key>notifications</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>u</string>
            <key>one</key>
            <string>%u more notification</string>
            <key>other</key>
            <string>%u more notifications</string>
        </dict>
    </dict>
</dict>
</plist>
```

# Designing Notifications
- 알림은 모두에게 great 해야한다.
- 사람들은 연결하고, 정보를 전달해야한다.

## First run
![스크린샷 2018-08-30 오전 11.27.17.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 11.27.17.png)
![스크린샷 2018-08-30 오전 11.58.44.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 11.58.44.png)
- 최초 앱설치 및 실행시 바로 알림에 대한 허가 미요청
- 왜 알림이 필요한지에 대한 설명
- 적절한 상황에 요구

## Provide value
![스크린샷 2018-08-30 오전 11.27.35.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 11.27.35.png)
![스크린샷 2018-08-30 오후 12.04.21.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오후 12.04.21.png)
- 의미있는 컨텐츠와 특별한 정보를 주어야 한다.
- 알림은 앱을 시작하는 이유가 되어서는 안된다.
- 전달을 고려해야 한다.
- 사용자가 스마트하게 사용할 수 있어야 한다.(앱 개별 알림 설정 옵션 제공)

## Notification Grouping
![스크린샷 2018-08-30 오전 11.27.40.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 11.27.40.png)
![스크린샷 2018-08-30 오후 12.05.43.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오후 12.05.43.png)

- 기본적으로 앱기준 그룹핑
- 쓰레드 그룹핑은 필요한 경우에 적용되야 한다. (하나의 앱에 너무 많은 쓰레드 그룹핑은 피해야 한다)

## Riche Rich Notifications
![스크린샷 2018-08-30 오전 11.27.44.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오전 11.27.44.png)
![스크린샷 2018-08-30 오후 12.11.01.png](/assets/images/post/wwdc18/notifications/스크린샷 2018-08-30 오후 12.11.01.png)
- 각 알림은 하나의 개별적인 업무로 처리되야 한다.
- 이미지, 비디오, 오디오, 커스텀 컨텐츠등이 추가 될 수 있다.

# Reference
- [https://developer.apple.com/videos/play/wwdc2018/710/](https://developer.apple.com/videos/play/wwdc2018/710/){:target="_blank"}
- [https://developer.apple.com/videos/play/wwdc2018/711/](https://developer.apple.com/videos/play/wwdc2018/711/){:target="_blank"}
- [https://developer.apple.com/videos/play/wwdc2018/806/](https://developer.apple.com/videos/play/wwdc2018/806/){:target="_blank"}
