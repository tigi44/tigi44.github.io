---
title: "[iOS] Code Signing"
excerpt: "iOS Code Signing"
description: "iOS Code Signing"
modified: 2022-12-08
categories: "iOS"
tags: [iOS, CodeSigning, Certificate, ProvisioningProfile]

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# 개요
* iOS 환경에서의 앱 서명키 발급과 관리방법, 빌드서버에서 앱 빌드시 서명하는 절차에 대한 정리

# iOS Code Signing
> 오직 애플만이 자신들의 하드웨어에서 어떤 소프트웨어가 동작하도록 허락할 수 있다.
- 애플을 통해 발급된 인증서 및 프로비저닝 프로파일등을 통해 앱의 제작자가 서명한 개발자임을 확인하고 서명 후 내용의 변경이 없음을 보장하는 작업

## Certificate
- 애플쪽 인증을 통하여 애플을 대신하여 앱을 동작시킬 수 있는 권한 제공

### 인증서 발급 절차
1. 키체인앱에서 [CSR(Certificate Signing Request)](https://en.wikipedia.org/wiki/Certificate_signing_request)인증서 발급
    - 프라이빗 키가 만들어지고 키는 Keychain에 저장
2. CSR인증서를 이용하여 개발자센터를 통해 Apple인증서 발급 및 다운로드
3. 다운로드한 Certificate를 실행하여 Keychain에 저장
    -  Code Signing Identity 발급 : 이 과정에서 Keychain에 저장되어 있던 프라이빗 키와 Certificate가 매치되어 Code Signing Identity(코드 사이닝 아이디)가 생성됨
    -  Certificate는 인증서의 소유자 이름, 인증서 소유자의 공개 키 (비밀 키는 소유자가 가지고 있다), 인증서의 유효 기간, 고유한 UID, 인증서의 기타 모든 값들을 해시화한 값을 갖음
4. 이 인증서를 통해 빌드시 Code Signing 진행

### 인증서 종류
- 배포(Distribution, Product) 인증서
    - AppStore 및 ad-hoc, In-House 등 배포용
- 개발(Development) 인증서
    - pc와 연결된 디바이스에 앱 설치를 위한 개발용(디버그 모드)

## Provisioning Profile
- 프로비저닝 프로파일은 App ID, Certificate, Device 정보등을 포함함
    - App ID : 앱 Bundle ID
    - Certificate : 애플 인증서
    - Device : 디바이스의 UDID
- 앱설치 및 실행시, 프로파일에 등록된 내용을 통해 인증서와 개발된 앱, 기기등을 연결

![1*5iJ7UXMuPcjFjwaE4jayUg.webp](/files/3427563572719222933)

### 프로파일 종류
- Development Profile: 허가받은 디바이스 아이디가 명시되어있으며, 디버그 모드에서 실행
- In-house Distribution Profile: 엔터프라이즈 개발자 계정에서만 사용 가능하며 회사 내 배포. 사용 허가 받은 디바이스 아이디를 관리하지 않으며, 모든 디바이스에서 사용 가능하도록 배포
- Ad-hoc Distribution Profile: 허가받은 디바이스 아이디가 명시되어있으며, 정해진 대수만큼만 배포
- App Store Distribution Profile: 판매를 위해 앱스토어 등록용 배포

## App ID (Bundle ID)
- 앱 구별을 위한 ID 값을 개발자센터에 등록
- 해당 앱에서 사용할 기능권한들(Capabilities)도 같이 등록
    - ex : push, nfc, wallet 등등

## Device
- 앱 설치시 사용된 기기의 UDID값을 개발자센터에 등록
- Development Profile 이나 Ad-hoc Profile 은 미리 등록된 특정 기기에서만 설치 및 실행 가능

# 빌드 및 배포 앱서명 과정
1. 위 과정들을 통해 앱 서명을 위해 준비 되는 항목
- 개발자를 증명하는 공개키와 비밀키 (KeyChain을 통해 생성 및 사용)
- 애플이 인정해준 앱 서명 허락 인증서 (KeyChain에 등록하여 사용)
- 디바이스에 설치 가능한 프로비저닝 프로파일

2. 위의 정보들을 통해 `*.mobileprovision` 을 생성하게 되며, 이를 컴파일 과정에서 사용하여 서명을 하게됨
3. 앱을 빌드(아카이빙)하게 되면 .app 패키지가 생성되면서 패키지 내부에 서명과 관련된 파일들이 생성됨
    - `embedded.mobileprovision` : 컴파일시 사용된 프로비저닝 프로파일
        - 해당 파일 안에는 인증서와 기기 목록, Entitlements 항목, 유효기간 등이 명시되어 있으며, 만약 앱을 실행하는 환경이 명시된 항목들과 일치 하지 않는 다면 앱을 실행할 수 없음
    - _CodeSignature 폴더 : `CodeResources` 란 파일(.plist)을 담고 있고, 패키지에 있는 모든 파일의 암호화된 해쉬정보를 담고 있음

4. 최종적으로 앱 실행시 아래 항목으로 체크된 후 앱을 실행하게 됨
    - .app 에 포함된 프로비저닝 프로파일이 애플에서 서명된 것인지 확인
    - `CodeResources` 란 파일에 기록된 각 파일의 해쉬 정보를 실제의 파일들과 확인하여 빌드 후 수정이 되지 않았음을 확인
    - 디바이스에 .app 에 포함된 프로비저닝 프로파일이 있는지 확인

## 프로파일별 앱 배포
### Development, Ad-hoc 앱실행
- 해당 프로파일은 허용된 디바이스 목록을 갖고 있기때문에, `해당 기기에서만 앱이 실행`됨
### In-house(Enterprise) 앱실행
- Enterprise 멤버십에 가입된 `회사를 대상으로 별도의 인증서가 발급`되고, 해당 인증서는 애플의 인증서와 같이 별도의 기기 확인없이 `모든 기기에서 앱이 설치 가능`하도록 함
### App Store 배포
- 해당 프로파일로는 `모든 디바이스에서 설치할 수 없는 상태`가 됨
- 오직 `애플쪽 제출만 가능`한 상태로, 애플측에서 앱배포를 승인하면서 애플에서 자신들의 서명을 다시함으로써 모든 iOS 디바이스에서 실행될 수 있도록 해줌

# Reference
- https://support.apple.com/ko-kr/guide/security/sec7c917bf14/web
- https://developer.apple.com/documentation/appstoreconnectapi/certificates
- https://velog.io/@enebin777/Apple-iOS-앱-배포-과정코드-사이닝-프로비저닝-프로필
- https://engineering.linecorp.com/ko/blog/ios-code-signing/
