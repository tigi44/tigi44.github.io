---
title: "The Clean Architecture"
excerpt: "About The Clean Architecture"
description: "About The Clean Architecture"
last_modified_at: 2021-01-18T14:00:00+09:00
categories: "ETC"
tags: [Development, Clean Architecture, Architecture]

toc: true

header:
  teaser: /assets/images/post/cleanArchitecture/theCleanArchitecture.jpg
---

## Architecture?
여러 개발자들로 구성된 팀내에서 만드는 앱은 Architecture가 굉장히 중요합니다.
Architecture가 잘 설계되지 않았다면, 앱을 구성하는 기능들은 각각의 구현방식이 달라질 수 있으며, 각각의 다른 패턴과 규칙들을 사용하게 됩니다.
이는 앱의 유지보수 및 확장성등에 제한을 생기게 합니다.
Architecture가를 잘 구성하게 된다면 다음과 같은 항목에서 많은 이점을 갖을 수 있습니다.

- 유지보수성(Maintainability)
- 가독성(Readability)
- 테스트용이성(Testability)
- 확장성(Scalability)
- 이식성(Portability)
- 성능(Performance)

여러종류의 시스템 아키텍쳐들이 있지만, 결국 아키텍쳐가 지향하는 바는 `관심사의 분리`입니다. 비슷한 관심사를 갖는 기능들을 계층(layer)으로 나누는 것 입니다.
이렇게 관심사들을 분리하게되면, 다음과 같은 구조를 갖는 시스템으로 만들어지게 됩니다.

- 프레임워크 독립적(Independent of Frameworks) : System Architecture는 소프트웨어의 라이브러리, 프레임워크등에 의존하지 않습니다.
- 테스트 용이(Testable) : 비지니스 로직은 UI, DB, Web Server 등 기타 외부 요인과 관계없이 테스트 가능합니다.
- UI 독립적(Independent of UI) : 시스템의 다른 부분을 고려하지 않고 UI를 변경할 수 있습니다.
- Database 독립적(Independent of Database) : DB 또한 독립적으로 변경할 수 있으며, 이는 비지니스 로직에 얽매이지 않습니다.
- 외부 기능 독립적(Independent of any external agency) : 비지니스 로직은 외부 기능들(DB, UI)에 대해서 아무것도 모릅니다.

이런 다양한 시스템 아키텍쳐들의 아이디어를 정리하여, `Uncle Bob`이라는 사람이 `Clean architecture` 라는 아키텍쳐를 소개하게 되었습니다.

## About Clean architecture

![TheCleanArchitecture](/assets/images/post/cleanArchitecture/theCleanArchitecture.jpg)

### The Dependency Rule
`Clean architecture`에서 중요한 부분이 바로 `The Dependency Rule` 입니다.
간단히 정리하자면 아래와 같습니다.
- 모든 소스코드의 의존성(Dependency)는 반드시 outer에서 inner로 향해야 한다.
- 원안에 있는 것들은, 그보다 원 밖에 있는 것들에 대해 아무것도 알 수 없다.(클래스명, 함수명 등등)
  - 여기서 예외(?)적으로 `Dependency Inversion`를 이용하여 원밖에 상의계층과 소통을 할 수 있습니다. 이에 대한 자세한 내용은 아래 `Crossing boundaries`에서 살펴보겠습니다.

### Entities
`Entity` 는 애플리케이션에서 핵심적인 기능 Business Rule을 담게 됩니다. 외부에 변화에 대해 변경될 가능성이 가장 적고, 가장 높은 수준의 규칙을 갖게 됩니다.
즉, 어떤 기능에도 종속적이지 않고, 독립적으로 비지니스 기능을 수행할 수 있는 데이터 구조 및 함수들이라고 생각하면 됩니다.

### Use cases
`Use Case`는 특정한 application에 대한 비지니스 룰입니다. 쉽게 말해, 시스템안에서 특정 application의 기능을 동작하는 비지니스 로직입니다. 특정 기능의 동작을 위한 로직이 필요하다면, 이곳에 구현을 하면 됩니다.
이는 `Entity`와의 데이터 흐름을 정의하고, 상호작용하게 됩니다.

### Interface Adapters
`Interface Adapters`는 원밖의 Frameworks and Drivers들과 원내부의 DomainLayer 사이에서 포맷을 변경해주는 번역기와 같은 역할을 합니다.
예로 View에서 Input 데이터를 입력받아, use case와 entity에서 사용하는 데이터 포맷으로 변경해주는 역할을 할 수 있습니다.
반대로 use case에서 처리한 Output 데이터를 받아, View나 DB와 같은 곳에서 사용하기 편한 데이터 포맷으로 변경하기도 합니다.

### Frameworks and Drivers
`Frameworks and Drivers` 는 UI, DB, Framework 등와 같은 외부 요소들이 포함됩니다.

### Only Four Circles?
지금까지 Clean Architecture내의 4가지 원(Entities, Use cases, Interface Adapters, Frameworks and Drivers)에 대해 설명하였습니다.
하지만 Clean Architecture에는 꼭 4까지 원만 존재해야 되는 것은 아닙니다. 처음 설명드린 `The Dependency Rule`만 잘 지켜진다면, 원의 갯수는 줄여도, 늘려도 상관 없습니다.

### Crossing boundaries
위의 Clean Architecture의 Dependency Rule에 따르면 제어의 흐름은 원밖에서 원안으로 향하는 것만 가능해보입니다.
하지만 상황에 따 각 계층을 교차해야 되는 경우가 생기게 됩니다.

![Crossing boundaries](/assets/images/post/cleanArchitecture/crossingBoundaries.png)

위 예시 이미지를 보면 `Controller → Use cases → Presenter` 와 같은 형태로 제어가 흐를 수 있음을 알 수 있습니다.
하위 계층의 `Use cases`가 상위 계층인 `Presenter(Interface Adapters)`를 제어하는 형태를 볼 수 있는데, 이런 흐름을 가져갈 수 있는 것은 `Dependency Inversion`이라는 원칙을 사용했기 때문입니다.
Dependency Inversion은 상위 계층인 `Presenter`의 `Output Port`인터페이스를, 하위 계층인 `Use cases`내부에 정의 함으로써, `Use cases`는 상위 계층(Presenter)의 존재를 모르더라도, 내부에 정의된 인터페이스를 통해 상위 계층을 호출할 수 있게 되는 것 입니다.

### What data crosses the boundaries.
의존성 규칙을 지키기 위해서는 단순하고, 고립된 형태의 데이터 구조를 사용해야 합니다.
외부 Framework에 종속적인 데이터 구조들을 계층 사이에서 사용한다면, 의존성을 해치게 됩니다.

### Conclusion
`Clean Architecture`는 의존성 규칙과 관심사 분리를 통해, 테스트하기 쉽고 구현에 많은 이점을 갖는 시스템을 만들어줍니다.

## Reference
* [https://medium.com/@afsara.504/clean-architecture-swift-8a5834aecf6](https://medium.com/@afsara.504/clean-architecture-swift-8a5834aecf6){: target="_blank"}
* [http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html){: target="_blank"}
