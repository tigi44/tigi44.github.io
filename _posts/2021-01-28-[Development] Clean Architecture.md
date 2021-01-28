---
title: "[Development] Clean Architecture"
excerpt: "About Clean Architecture"
description: "About Clean Architecture"
last_modified_at: 2021-01-28T14:00:00+09:00
categories: "Development"
tags: [Development, Clean Architecture, Architecture]

toc: true

header:
  teaser: /assets/images/swift-logo.png
---

왜 architecture가 중요한가요?

팀이 여러 프로그램으로 구성된 모든 앱에서 아키텍처가 정말 중요합니다. 많은 것들이 개발해야 할 새로운 기능을 바꿀 것입니다. 아키텍처를 설정하지 않은 경우 각각의 구현 방식이 다르면 다른 패턴과 다른 명명 규칙을 사용하게 됩니다. 이는 프로젝트의 확장성 및 유지보수성에 영향을 미칩니다. 간단한 변경이나 새로운 기능에 대해 여러 가지 부작용이 발생할 수 있습니다. 다음은 아키텍처에 의해 영향을 받는 몇 가지 중요한 사항입니다.

유지보수성 : 유지보수성은 모바일 애플리케이션 아키텍처의 속성 중 매우 중요합니다. 일부 제어 변경이나 버그 수정 또는 기타 변경과 같은 환경으로 인해 요구 사항이 변경되었다고 가정해 보겠습니다. 항상 앱에서 이러한 종류의 변화는 생기기 마련입니다. 따라서 앱에서 수정하려면 좋은 아키텍처가 필요합니다.
가독성 : 코드 가독성에는 여러 측면이 있으며 사람들은 이에 대해 다른 의견을 가지고 있습니다. 주요 측면은 명명 규칙 (클래스, 메서드 및 변수 이름) 주석 및 문서화입니다.
테스트용이성 : 잠재적인 오류를 방지하기 위해 앱이 다른 조건에서 작동하는지 확인하려면 모바일 앱을 철저히 테스트해야합니다. 좋은 애플리케이션 아키텍처는 앱이 각 모듈에서 독립적으로 테스트 할 수 있는지도 측정 항목입니다.
확장성 : 애플리케이션 아키텍처가 좋지 않은 경우 새로운 기능 구현은 정말 고통 스럽습니다. 우수한 애플리케이션 아키텍처는 최소한의 노력으로 애플리케이션을 확장하는 데 도움이 됩니다.
이식성 : 모바일 애플리케이션에서는 환경 변화가 자주 발생합니다. 이식성은 환경의 변화에 ​​따라 앱을 처리하는 것입니다. 이러한 변경에는 구성 요소, 데이터베이스, 서비스, API 등이 포함됩니다. 우수한 모바일 앱 아키텍처는 이식성을 보장합니다.
성능 : 성능은 웹 및 데스크톱 응용 프로그램과 비교할 때 모바일 응용 프로그램 개발에서 중요한 역할을합니다. 모바일 앱 사용자는 응답을 빨리 받기를 원합니다. 그렇지 않으면 사용자가 앱을 삭제하게 됩니다. 좋은 모바일 애플리케이션 아키텍처는 앱의 성능을 달성하는 데 정말 중요합니다.

---

Clean architecture 소개
밥 삼촌이 클린 아키텍처를 제안했습니다. 클린 아키텍처는 유연성 확장성 및 유지보수성으로 인해 널리 사용됩니다. 클린 아키텍처는 앱을 빌드 할 수 있는 일련의 규칙 및 권장 사항입니다.

앱 아키텍쳐의 가이드 라인은 다음과 같습니다.

프레임워크로부터의 독립성 : 아키텍쳐는 라이브러리, 프레임워크에 독립적이여야 합니다.
테스트 가능성 : 앱은 독립적으로 테스트 가능해야 합니다.
UI는 비즈니스 로직으로부터 독립적으로 존재 : 비즈니스로직은 UI와 약한 결합을 유지해야 합니다.
데이터 레이어는 비즈니스 로직에 독립적이어야 합니다: API, coredata, Sqllite 등 우리가 사용하는 모든 데이터는 비즈니스 로직과 독립적이어야합니다.

---

1. Independent of Frameworks. 아키텍쳐는 소프트웨어 라이브러리의 존재에 의존하지 않음.
2. Testable. 비즈니스 로직은 UI, DB, 웹 서버 또는 기타 외부 요소 없이 테스트 할 수 있음.
3. Independent of UI. UI는 시스템을 변경하지 않고도 쉽게 변경 가능. (ex. 비즈니스 로직을 바꾸지 않고 웹 UI를 콘솔 UI로 변경가능.)
4. Independent of Database. 비즈니스 로직이 DB에 바인딩 되지 않음.
5. Independent of any external agency. 비즈니스 로직은 외부 세계;;(outside world에 대해 전혀 알지 못함

---

## Reference
https://zeddios.tistory.com/1065
http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
https://k-elon.tistory.com/38
https://woowabros.github.io/tools/2019/10/02/clean-architecture-experience.html
https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37
https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3
- [https://developer.apple.com/documentation/swift_packages/bundling_resources_with_a_swift_package](https://developer.apple.com/documentation/swift_packages/bundling_resources_with_a_swift_package){: target="_blank"}
