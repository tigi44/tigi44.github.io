---
title: "[WWDC21] Xcode Cloud"
excerpt: "[WWDC21] Xcode Cloud"
description: "[WWDC21] Xcode Cloud"
modified: 2021-06-22
categories: "WWDC21"
tags: [WWDC21, Xcode, Xcode13, Xcode Cloud]

header:
  teaser: /assets/images/teaser/wwdc21-teaser.jpg
---

# Overview
- Xcode Cloud는 CI(continuous integration)를 기반으로 구축되었으며, Apple 개발 툴(Xcode, ...)들을 연결하여 프로젝트의 빌드, 테스트, 배포, 피드백 수집등 신속하게 반복할 수 있는 완전한 개발 파이프라인을 제공

- Xcode + AppStoreConnect + TestFlight
![overview](/assets/images/post/wwdc21/xcodecloud/overview.png)

## Xcode Cloud
- workflow를 설정해놓으면, 협업하는 개발자들이 수정 및 진행사항, 결과 확인이 가능함

![xcodecloud](/assets/images/post/wwdc21/xcodecloud/xcodecloud.png)

## App Store Connect
- Xcode Cloud 기능은 Xcode뿐만 아니라 App Store Connect 웹에서도 사용할 수 있음
- Post-actions 기능을 통해, 빌드된 앱을 TestFlight에 바로 배포 가능, 이때 앱인증서는 cloud signing을 통해 자동으로 적용됨

## Privacy
- Cloud상에서의 빌드 환경은 임시적. 빌드되는 소스코드는 cloud상에 저장되지 않으며, 워크로드는 격리되고, 빌드 환경은 빌드시마다 새롭게 생성됨

# Workflow
- Workflow는 Xcode Cloud에서 수행할 작업과 수행할 시기등을 설정하는 기본 구성
- drive the continuous integration
    - automate building, analyzing, testing, archiving, and distributing your apps and frameworks
- General, Start Condition, Environment, Actions, Post-Actions으로 구성됨

## General
- 이름 및 레파지토리등의 기본적인 설정

![general](/assets/images/post/wwdc21/xcodecloud/general.png)

## Start Condition
- Workflow의 작업 시기를 설정할 수 있음
- pull request완료시, branch내용 변경시, 주기적인 스케쥴등등으로 설정 가능

![startcondition](/assets/images/post/wwdc21/xcodecloud/startcondition.png)

## Environment
- 빌드 환경 설정
- xcode, macos등의 버전 설정도 가능, 로컬에는 없는 버전을 cloud를 통해서 빌드 테스트 가능

![environment](/assets/images/post/wwdc21/xcodecloud/environment.png)

## Actions
- Archive, Build, Analyze, Test

![actions](/assets/images/post/wwdc21/xcodecloud/actions.png)



## Post-Actions
- SendNotifications
  - 기본적으로 Slack과 email을 통해 notification 설정 가능
  - 이외의 추가적인 notification가 필요하면 Workflow Custom을 통해 webhook 이용

![postactionsnoti](/assets/images/post/wwdc21/xcodecloud/postactionsnoti.png)

- Deploying with TestFlight
  - Actions의 Archive에서 TestFlight 배포준비 설정을 해야함
  - Internal(개발 단계) 과 External(배포전 단계) TestFlight 배포 존재
  - 앱 배포 인증서는 cloud signing을 통해 자동 적용됨([https://developer.apple.com/wwdc21/10267](https://developer.apple.com/wwdc21/10204){:target="_blank"})

![postactiontestflight](/assets/images/post/wwdc21/xcodecloud/postactionstestflight.png)

# Custom Workflow
- Xcode내에서 설정할 수 있는 설정항목외에 커스텀한 항목을 추가하여 Workflow를 좀 더 확장하여 사용 가능

## Environment Varibles
   - 빌드시 필요한 환경변수 설정가능, 앱키같이 private한 변수는 secret으로 숨김처리 가능

## Custom scripts
   - shell script
   - post-clone, pre-xcodebuild, post-xcodebuild

## Additional repositories
   - access to a private extra repository

## Webhooks
   - http 통신으로 외부 서비스 호출 가능
   - Xcode cloud에 기본 설정되어 있는 Slack외의 다른 서비스에 빌드 진행사항 notification 가능

# Reference
- [https://developer.apple.com/wwdc21/10267](https://developer.apple.com/wwdc21/10267){:target="_blank"}
- [https://developer.apple.com/wwdc21/10268](https://developer.apple.com/wwdc21/10268){:target="_blank"}
- [https://developer.apple.com/wwdc21/10269](https://developer.apple.com/wwdc21/10269){:target="_blank"}
