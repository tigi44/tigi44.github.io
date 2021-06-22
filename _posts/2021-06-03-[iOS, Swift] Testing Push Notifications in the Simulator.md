---
title: "[iOS, Swift] Testing Push Notifications in a Simulator"
excerpt: "Testing Push Notifications in the Simulator"
description: "Testing Push Notifications in the Simulator"
modified: 2021-06-03
categories: "iOS"
tags: [iOS, Apns, Swift, Simulator]

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# 시뮬레이터에서 푸쉬 테스트하기

- XCode 11.4 부터 로컬 시뮬레이터에서 push notification을 쉽게 테스트할 수 있게 되었습니다.
- 애플 개발자 사이트에서 별도의 푸시 인증서를 발급받지 않아도 되고, XCode 설정에서 push notification 관련 capability를 추가하지 않아도 푸쉬 테스트가 가능합니다.

## 앱 권한 요청 준비
- 기본적으로 앱에서 푸쉬를 받기 위해서는, 푸쉬 관련 권한이 설정되어야 함
- 앱 실행시, appDelegate내에서 푸쉬 권한 요청하는 코드 준비
```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    UNUserNotificationCenter.current()
      .requestAuthorization(options: [.alert, .sound, .badge]) {(granted, error) in

    }

    return true
}
```

## payload file 생성
- 로컬 시뮬레이터 테스트에서는 APNs 인증서와 token등의 확인이 필요없고, `Json형식의 payload file`만 준비하면 테스트 가능
    - 파일 확장자 : `*.apns`
```json
{
  "aps": {
    "alert": {
      "title": "Test",
      "body": "test message",
      "sound": "default"
    },
    "badge": 1
  },
  "Simulator Target Bundle": "com.app.bundle.id"
}
```
- `aps` : apns 사용을 위한 payload 값들
    - 사용 가능한 payload 키값  : https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification
-  `Simulator Target Bundle` : 앱 번들 id
    -  원격 APNs를 사용할 경우는 필요 없는 항목이지만, payload file을 이용해 로컬 시뮬레이터에서 사용시 앱을 구분하기위해 필요한 값

## 푸쉬 기능 동작 - 1. 파일 드래그
- 앞서 준비한 `*.apns` 파일을 시뮬레이터에 드래그하여, 푸쉬 기능을 동작 시킴
![ex](https://jusung.github.io/images/2020/remotepush%20test%20drag.gif)

## 푸쉬 기능 동작 - 2. CLI (Command Line Interface) 이용
- 터미널과 같은 `Command Line Interface`를 통해서 동작도 가능
- CLI를 이용하는 경우, payload file내의 Simulator Target Bundle 항목 삭제가능, CLI명령어에서 이미 {app bundle id} 값을 이용
```shell
$ xcrun simctl push {device id} {app bubdle id} {payload file}
$ xcrun simctl push 6AE60A8A-2142-43D7-8F66-D26358BF3E0E com.app.bundle.id test.apns
```

- 실행중인 시뮬레이터 디바이스 id 찾는 법
```shell
$ xcrun simctl list devices | grep Booted
    iPhone 12 Pro Max (6AE60A8A-2142-43D7-8F66-D26358BF3E0E) (Booted)
```

# Reference
- [https://medium.com/swlh/simulating-push-notifications-in-ios-simulator-9e5198bed4a4](https://medium.com/swlh/simulating-push-notifications-in-ios-simulator-9e5198bed4a4){:target="_blank"}
- [https://jusung.github.io/apns-test/](https://jusung.github.io/apns-test/){:target="_blank"}
