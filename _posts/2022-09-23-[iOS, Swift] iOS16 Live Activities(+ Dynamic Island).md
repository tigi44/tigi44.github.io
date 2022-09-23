---
title: "[iOS, SwiftUI] iOS16 Live Activities(+ Dynamic Island)"
excerpt: "iOS16 ActivityKit - Live Activities + Dynamic Island"
description: "iOS16 ActivityKit - Live Activities + Dynamic Island"
modified: 2022-09-23
categories: "iOS"
tags: [iOS, Swift, SwiftUI, ActivityKit, LiveActivities, DynamicIsland]

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# Live Activities(+ Dynamic Island) (ActivityKit)
![ios-live-activities-examples](/assets/images/post/ios/activitykit/ios-live-activities-examples.png)
![iPhone-14-Pro-Dynamic-Island](/assets/images/post/ios/activitykit/iPhone-14-Pro-Dynamic-Island.png)

- iOS16.1 beta 버전부터 개발 지원
- Live Activity는 Dynamic Island와 LockScreen에서 하나의 위젯처럼 실시간 정보를 확인할 수 있음
- 하나의 Live Activity가 생성되면, Dynamic Island를 지원하는 기기에서는 Dynamic Island 영역에도 나타남
- 앱에서 ActivityKit을 사용하여 Live Activity을 configure, start, update 및 end하며, 앱의 위젯 확장 프로그램인 SwiftUI와 WidgetKit을 사용하여 Live Activity의 UI를 만듬
- 기존의 위젯과 다르게 빈번한 업데이트가 이뤄질 수 있기때문에, timeline대신 Remote Notification을 이용하여 업데이트된 데이터를 수신할 수 있음

# Example SourceCode
- [https://github.com/tigi44/LiveActivitiesExample](https://github.com/tigi44/LiveActivitiesExample){:target="_blank"}

# Live Activities Example

| LockScreen | DynamicIsland Compact View | DynamicIsland Minimal View | DynamicIsland Expanded View |
|---|---|---|---|
| ![live_activity_lockscreen_example](/assets/images/post/ios/activitykit/live_activity_lockscreen_example.png) | ![live_activity_dicompact_example](/assets/images/post/ios/activitykit/live_activity_dicompact_example.png) | ![live_activity_diminimal_example](/assets/images/post/ios/activitykit/live_activity_diminimal_example.png) | ![live_activity_diexpand_example](/assets/images/post/ios/activitykit/live_activity_diexpand_example.png)  |

## 1. Add a WidgetExtension
- Live Activity는 Widget을 이용하기때문에, 앱에 WidgetExtension을 추가해야 함
  - [WidgetKitExample](https://github.com/tigi44/WidgetKitExample){:target="_blank"}

## 2. Setting Info.plist
- Supports Live Activities(NSSupportsLiveActivities)항목을 추가하고, YES로 값을 변경

## 3. Define a set of static and dynamic data (ActivityAttributes)
```swift
public enum OrderStatus: CaseIterable, Codable, Equatable {
    case received
    case progress
    case ready
}

struct OrderAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var status: OrderStatus = .received
    }

    var orderNumber: Int
    var orderItems: [String]
}
```
- ActivityAttributes를 통해 Activity를 구성할 static 데이터 구현
- Activity.ContentState를 사용하여, Live Activity의 dynamic 데이터를 구현함
- 위 예제 코드에서는 orderNumber, orderItems라는 변경되지 않는 정적 데이터와 함께, status라는 동적 데이터를 이용하여 Live Activity의 데이터를 구성하게 됨

## 4. ActivityConfiguration (Widget)
```swift
import WidgetKit
import SwiftUI
import Intents
import ActivityKit

@main
struct OrderCoffee: Widget {
    var body: some WidgetConfiguration {
      ActivityConfiguration(for: OrderAttributes.self) { context in
            // Create the view that appears on the Lock Screen and as a
            // banner on the Home Screen of devices that don't support the
            // Dynamic Island.
            // ...
        } dynamicIsland: { context in
            // Create the views that appear in the Dynamic Island.
            // ...
        }
    }
}
```
- ActivityConfiguration를 이용하여 Live Activity의 ui가 될 위젯을 구성

### Create the Lock Screen view
```swift
@main
struct OrderCoffee: Widget {
    var body: some WidgetConfiguration {
      ActivityConfiguration(for: OrderAttributes.self) { context in
            LockScreenLiveActivityView(context: context)
        } dynamicIsland: { context in
            ...
        }
    }
}

struct LockScreenLiveActivityView: View {
    let context: ActivityViewContext<OrderAttributes>

    var body: some View {
        ...
    }
}
```  
- SwiftUI를 이용하여, 위젯처럼 Live Activity 화면 구현

### Create the DynamicIsland view
- Dynamic Island에는 여러 영역이 있으며, 해당 영역별로 View들을 구현해야 함
- Dynamic Island 뷰
![dynamicisland_views](/assets/images/post/ios/activitykit/dynamicisland_views.png)
- Dynamic Island Expanded view
![dynamicisland_expand_view](/assets/images/post/ios/activitykit/dynamicisland_expand_view.png)

```swift
@main
struct OrderCoffee: Widget {
    var body: some WidgetConfiguration {

        ActivityConfiguration(for: OrderAttributes.self) { context in
            LockScreenLiveActivityView(context: context)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading) {
                    ...
                }
                DynamicIslandExpandedRegion(.trailing) {
                    ...
                }
                DynamicIslandExpandedRegion(.center) {
                    ...
                }
                DynamicIslandExpandedRegion(.bottom) {
                    ...
                }
            } compactLeading: {
                HStack(spacing: -8) {
                    ForEach(context.attributes.orderItems, id: \.self) { item in
                        Image(systemName: item)
                            .resizable()
                            .aspectRatio(contentMode: .fit)
                            .frame(width: 30, height: 30)
                    }
                }
                .foregroundColor(.purple)
            } compactTrailing: {
                Image(systemName: context.state.status.image)
                    .font(.title3)
            } minimal: {
                Image(systemName: context.state.status.image)
                    .font(.title3)
            }
            .keylineTint(Color.purple)
        }
    }
}
```
- keylineTint(_:)를 통해 DynamicIsland에서 컬러값을 줄 수 있음

### Create a deep link into your app
- widgetURL(_:)
  - the Lock Screen, compact leading, compact trailing, and minimal views
- SwiftUI’s Link.
  - Expanded View

## 5. Start, Update, End
```swift
// end the Live Activity
public static func endLiveActivity(id: String) {
    guard let activity = Activity<OrderAttributes>.activities.first(where: { $0.id == id }) else {
        return
    }

    Task {
        await activity.end(using: activity.contentState, dismissalPolicy: .immediate)
    }
}

// update the Live Activity
public static func updateLiveActivity(id: String, updatedStatus: OrderStatus) {
    guard let activity = Activity<OrderAttributes>.activities.first(where: { $0.id == id }) else {
        return
    }

    var updatedState = activity.contentState
    updatedState.status = updatedStatus

    Task {
        await activity.update(using: updatedState)
    }
}

// Start a Live Activity
public static func addLiveActivity() -> String? {
    let orderAttributes = OrderAttributes(orderNumber: 123, orderItems: ["cup.and.saucer.fill", "fork.knife.circle.fill"])
    let initialContentState = OrderAttributes.ContentState()

    do {
        let activity = try Activity<OrderAttributes>.request(attributes: orderAttributes,
                                                             contentState: initialContentState,
                                                             pushType: nil)

        print("Request a Live Activity \(String(describing: activity.id)).")

        return activity.id
    } catch {
        print(error.localizedDescription)

        return nil
    }
}
```

# Update or end your Live Activity with a remote push notification
- [https://developer.apple.com/documentation/activitykit/update-and-end-your-live-activity-with-remote-push-notifications](https://developer.apple.com/documentation/activitykit/update-and-end-your-live-activity-with-remote-push-notifications){:target="_blank"}
- 앱에서 Remote Notification을 수신할 수 있도록 설정이 필요함
- Live Activity생성시, Activity 인스턴스에서 pushToken값을 얻을 수 있음
- 해당 pushToken값을 이용하여 APNs를 통해 푸쉬 전송시, 해당 Live Activity에서 데이터를 수신함
- apns payload
  - event: 해당 필드에 update/end 값을 통해, Live Activity를 업데이트하거나 종료 시킬수 있음
  - content-state: 앱에서 구현한 Activity.ContentState와 같은 포맷으로, 해당 필드를 통해 dynamic 데이터를 업데이트 시킬 수 있음
```json
{
    "aps": {
        "timestamp": 1168364460,
        "event": "update",
        "content-state": {
            "status": "progress"
        },
        "alert": {
            "title": "Order in progress",
            "body": "Your order is in progress.",
            "sound": "example.aiff"
        }
    }
}
```

# Reference
- [https://developer.apple.com/widgets/](https://developer.apple.com/widgets/){:target="_blank"}
- [https://developer.apple.com/documentation/activitykit](https://developer.apple.com/documentation/activitykit){:target="_blank"}
- [https://developer.apple.com/documentation/activitykit/displaying-live-data-with-live-activities](https://developer.apple.com/documentation/activitykit/displaying-live-data-with-live-activities){:target="_blank"}
- [https://developer.apple.com/documentation/activitykit/update-and-end-your-live-activity-with-remote-push-notifications](https://developer.apple.com/documentation/activitykit/update-and-end-your-live-activity-with-remote-push-notifications){:target="_blank"}
- [Kavsoft](https://youtu.be/gEWvV-TmjqE){:target="_blank"}
