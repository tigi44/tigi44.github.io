---
title: "[iOS, SwiftUI] Local Notification on SwiftUI"
excerpt: "Local Notification on SwidtUI"
description: "Local Notification on SwiftUI"
modified: 2022-01-25
categories: "iOS"
tags: [iOS, Swift, SwiftUI, Notification]

toc: true

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# Request Authorization

```swift
UNUserNotificationCenter.current()
    .requestAuthorization(options: [.alert, .sound, .badge]) { (granted, error) in
        if let error = error {
        }
    }
```
![alert](/assets/images/post/ios/notification/alert.png)

# Add a NotificationRequest

```swift
UNUserNotificationCenter.current()
    .getNotificationSettings { settings in
        guard settings.authorizationStatus == .authorized else {
            return
        }

        let content         = UNMutableNotificationContent()
            content.title   = "Test Title"
            content.body    = "test message"
            content.sound   = UNNotificationSound.default
            content.badge   = 1
        let dateInfo        = Calendar.current.dateComponents([.weekday, .hour, .minute, .second], from: Date()) // Repeating every week by week
        let trigger         = UNCalendarNotificationTrigger(dateMatching: dateInfo, repeats: true)
        let request         = UNNotificationRequest(identifier: "identifier", content: content, trigger: trigger)

        UNUserNotificationCenter.current().add(request) { error in
            if let error = error {
            }
        }
    }
```

## Notification Trigger
- [UNPushNotificationTrigger](https://developer.apple.com/documentation/usernotifications/unpushnotificationtrigger){:target="_blank"}
- [UNTimeIntervalNotificationTrigger](https://developer.apple.com/documentation/usernotifications/UNTimeIntervalNotificationTrigger){:target="_blank"}
- [UNCalendarNotificationTrigger](https://developer.apple.com/documentation/usernotifications/UNCalendarNotificationTrigger){:target="_blank"}
- [UNLocationNotificationTrigger](https://developer.apple.com/documentation/usernotifications/UNLocationNotificationTrigger){:target="_blank"}

# Remove Pending NotificationRequests

```swift
UNUserNotificationCenter.current()
    .removePendingNotificationRequests(withIdentifiers: "identifier")

UNUserNotificationCenter.current()
    .removeAllPendingNotificationRequests()
```

# UNUserNotificationCenterDelegate

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter,
                            openSettingsFor notification: UNNotification?) {
}

func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {    
    completionHandler()
}

func userNotificationCenter(_ center: UNUserNotificationCenter,
                            willPresent notification: UNNotification,
                            withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    return [.sound, .banner]
}
```

# Reset the badge number
- must be in main thread

```swift
UIApplication.shared.applicationIconBadgeNumber = 0
```

# Source Code
<script src="https://gist.github.com/tigi44/5a457075bde06a79ccab617441331596.js"></script>

# Reference
- [https://medium.com/doyeona/scheduled-user-notifications-local-notification-ios-70c81aab8c54](https://medium.com/doyeona/scheduled-user-notifications-local-notification-ios-70c81aab8c54){:target="_blank"}
