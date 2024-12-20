---
title: "[iOS] Communication and Time Sensitive Notifications"
excerpt: "[iOS] Communication and Time Sensitive Notifications”
description: "Communication and Time Sensitive Notifications"
modified: 2024-12-20
categories: "iOS"
tags: [iOS, Notification, Communicatin, TimeSensitive, interruptionLevel]

toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# Communication and Time Sensitive notifications
![notification_setting.png](/assets/images/post/notification/notification_setting.png)

- 시간 지정 요약 설정시 해당 시간의 푸시(notifications)들이 바로 노출되지 않게됨
- 시간 지정 요약 설정을 무시하고 푸시를 바로 노출 시키기 위해서는 `TimeSensitiveNotifications`와 `CommunicationNotifications` 등의 설정이 필요함

# Time Sensitive Notifications
- iOS15 부터 Interruption levels ([UNNotificationInterruptionLevel](https://developer.apple.com/documentation/usernotifications/unnotificationinterruptionlevel)) 항목이 추가됨
- Passive (NEW), Active (default level), Time Sensitive (NEW), Critical

| |sound or vibration  | screen light up | system breakthrough | bypass ringer switch |
| — | — | — | — | — |
|Passive |❌  |❌  | ❌ | ❌ |
|Active | ✅ |✅  |  ❌| ❌ |
|Time Sensitive | ✅ |✅  | ✅ |❌  |
|Critical | ✅ | ✅ |  ✅|  ✅|

## 적용방법
- 이를 적용하기 위해서는 `Xcode의 프로젝트 설정에서 Capability(Time Sensitive Notifications)를 추가` 해야 함
- 추가로 Notification 호출시 `interruptionLevel` 값을 설정해 주면 적용 됨

### Local notification
- interruptionLevel
  
```swift 
let content = UNMutableNotificationContent()
content.title = “Urgent”
content.body = “Your account requires attention.”
content.interruptionLevel = .timeSensitive


let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 0, repeats: false)


let request = UNNotificationRequest(identifier: “time-sensitive—example”,
                                       content: content,
                                       trigger: trigger)
```

### Remote Notification (Push)
- interruption-level 필드 추가
 
```json
{
  “aps” : {
    “alert” : {
      “title” : “Urgent”,
      “body” : “Your account requires attention.”
    },
    “interruption-level” : “time-sensitive”
  }
}
```

# Communication Notifications
- Communication Notifications은 기본적으로 시간 지정 요약 기능을 무시하면서 사용자에게 전달됨
    - 이를 이용하여 푸시를 사용자에게 전달 할 수 있음
- Communication Notifications은 Siri intents(INStartCallIntent, INSendMessageIntent) 등을 사용해야 함

## 적용방법
- `Xcode의 프로젝트 설정에서 Capability(Communication Notifications)를 추가` 해야 함

### Info.plist
- INStartCallIntent, INSendMessageIntent 를 적용하기 위해, Info.plist에 `NSUserActivityTypes` 항목에 사용할 intents를 추가해야 함
 
```
<key>NSUserActivityTypes</key>
<array>
    <string>INSendMessageIntent</string>
</array>
``` 

### Add Notification Service Extension
- 다이렉트 메세지용으로 푸시로 업데이트시키기 위해, `Notification Service Extension`을 이용하여 SiriKit Intents를 추가해줘야 함
- 해당 프로젝트에 Notification Service Extension target을 추가
- `UNNotificationServiceExtension` 를 상속받는 클래스에서 didReceive 함수내에서 Intents 사용 부분 작성
 
```swift
import UserNotifications
import Intents
import UIKit

class NotificationService: UNNotificationServiceExtension {    
    override func didReceive(_ request: UNNotificationRequest,
                    withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        let incomingMessageIntent: INSendMessageIntent = // ...
        //let interaction = INInteraction(intent: incomingMessageIntent, response: nil)
        //interaction.direction = .incoming
        //interaction.donate(completion: nil)
        do {
            let messageContent = try request.content.updating(from: incomingMessageIntent)
            contentHandler(messageContent)
        } catch {
           // Handle error
        }
    }
}
``` 

### Remote Notification (Push)
- 해당 푸시가 notification service app extension 을 동작하려면, 푸시의 payload에 아래 항목이 추가되야 함
    - `mutable-content` 필드값이 `1`로 설정되야 함
    - `alert` 항목에 title, subtitle, or body 값이 설정되야 함

```json
{
   “aps” : {
      “category” : “SECRET”,
      “mutable-content” : 1,
      “alert” : {
         “title” : “Secret Message!”,
         “body”  : “(Encrypted)”
     }
   }
}
```

# Reference
- [Generating a remote notification](https://developer.apple.com/documentation/usernotifications/generating-a-remote-notification)
- [Send communication and Time Sensitive notifications](https://developer.apple.com/videos/play/wwdc2021/10091)
- [Implementing communication notifications](https://developer.apple.com/documentation/usernotifications/implementing-communication-notifications)
- [Modifying content in newly delivered notifications](https://developer.apple.com/documentation/UserNotifications/modifying-content-in-newly-delivered-notifications)
- https://docs.smartsupp.com/mobile-sdk/ios/communication-notifications/
- https://wwdcnotes.com/documentation/wwdcnotes/wwdc21-10091-send-communication-and-time-sensitive-notifications/
- https://ios-development.tistory.com/1283