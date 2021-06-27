---
title: "[iOS, Widget] a Issue about New Widget on WigetKit with TodayExtension"
excerpt: "a Issue about New Widget on WigetKit with TodayExtension"
description: "a Issue about New Widget on WigetKit with TodayExtension"
modified: 2021-06-27
categories: "iOS"
tags: [iOS, Widget, WidgetKit, TodayExtension]

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# Issue

## iOS14 ~

### 오류 케이스

- 기존 today extension이 1개 이상 설정되어 있는 상태
- 이 상태에서 new widget이 추가된 버전의 앱으로 업데이트 실행
- 기존 설정된 today extension이 모두 제거(목록에서도 삭제)되고, 새로운 widget이 자동으로 설정됨
- 앱 삭제후 재설치 or 디바이스 리부팅시, 목록에서 제거된 today extension이 다시 목록에 추가

### 정상 케이스

- 기존 today extension이 설정되어 있지 않은 상태
- new widget이 추가된 버전의 앱으로 업데이트 실행
- today extension이 목록에서 제거되지도 않고, 새로운 widget이 자동 설정되지도 않음


## iOS14.5
- iOS 14.5(?)부터 기존 위젯(TodayExtenstion)이 삭제되지 않고, unable to load 상태(?)로 전환됨
- 이전과 마찬가지로 디바이스 리부팅 or 앱 재설치시 문제 해결



## Reference Forums
### Apple Developer Forums
- [https://developer.apple.com/forums/thread/650350?answerId=614450022](https://developer.apple.com/forums/thread/650350?answerId=614450022){:target="_blank"}
> Today Extensions continue to be available, however, they are deprecated. When your app is submitted for the newer SDK the legacy today widget extension will be removed.

# Apple Feedback & Developer Technical Support(DTS)
## Feedback

### Q : (2021/3/26)
We've developed and distributed widgets using WidgetKit.

However, when updated with the app version that contains the WidgetKit, the existing today extension widget is automatically deleted and no longer displayed. And a new widget(using WidgetKit) was installed automatically.

Occurs when the App is updated when the today extension widget is displayed, and it was often resolved when the app was deleted and reinstalled or rebooted.

If I build with WidgetKit, is it right that Today Extension is deleted automatically?
Or Is there another solution?

< STEPS TO REPRODUCE >
1. Set up one or more Today Extensions
2. Update to version of app with WidgetKit
3. Confirm the existing Today Extension is deleted and a new widget is automatically installed

### A : (2021/6/9)
Thanks for your feedback.  We believe this issue is resolved.

Please test with the latest iOS / iPadOS 15 beta release and update your feedback report with your results by logging into https://feedbackassistant.apple.com or by using the Feedback Assistant app.

Betas and Release Candidates:
https://developer.apple.com/download/

GM Releases:
https://developer.apple.com/download/release/

If the issue continues, please update any logs and screen recordings:
https://developer.apple.com/bug-reporting/profiles-and-logs/

Please close this feedback report, or let us know if this is still an issue for you.  Thank you.

### Q : (2021/6/14)
Thank you for your answer.

But The issue has not yet been solved. The same thing happens in iOS15.
So I will attach a few screenshots to explain the request for the more details.
This was tested on the ios15 beta simulator.

1. I installed the today extension named 'XXX' on the simulator. (image1 and image1-1)
2. I updated to version of app with new WidgetKit.
3. As you can see in image2 and image2-1, the existing today extension named 'XXX' was automatically deleted and a new widget was automatically installed. The deleted today extension('XXX') was not found in the list(image2-1).
4. The deleted today extension is listed again when i reinstall the app or reboot the device.

Please check if this is a bug on the iOS or if it is intended.

### A :
waiting....

## DTS (Developer Technical Support)
### Q : (2021/3/26)
PLATFORM AND VERSION
iOS
- iPhone 12 mini (iOS 14.4.1)
- iPhone 12 Pro Max (iOS 14.4)

DESCRIPTION OF PROBLEM
We've developed and distributed widgets using WidgetKit.

However, when updated with the app version that contains the WidgetKit, the existing today extension widget is automatically deleted and no longer displayed. And a new widget(using WidgetKit) was installed automatically.

Occurs when the App is updated when the today extension widget is displayed, and it was often resolved when the app was deleted and reinstalled or rebooted.

If I build with WidgetKit, is it right that Today Extension is deleted automatically?

Or Is there another solution?

STEPS TO REPRODUCE
1. Set up one or more Today Extensions
2. Update to version of app with WidgetKit
3. Confirm the existing Today Extension is deleted and a new widget is automatically installed

### A : (2021/3/30)
Thank you for contacting Apple Developer Technical Support (DTS).

**The behavior and resulting limitations you describe are by design.**

If you believe an alternative approach should be considered by Apple, we encourage you to file an enhancement request with information on how this design decision impacts you, and what you’d like to see done differently.

Although there is no promise that the behavior will be changed, it is the best way to ensure your thoughts on the matter are seen by the team responsible for the decision.
