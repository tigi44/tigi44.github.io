---
title: "MVVM Pattern"
excerpt: "About the MVVM Pattern"
description: "About the MVVM Pattern"
last_modified_at: 2021-01-18T14:00:00+09:00
categories: "ETC"
tags: [Development, MVVM, Design Pattern, Architecture]

toc: true

header:
  teaser: /assets/images/post/mvvm/MVVMPattern.png
---

![mvvm](/assets/images/post/mvvm/MVVMPattern.png)

## What is MVVM
`MVVM` 하나의 소프트웨어 아키텍쳐 패턴으로, UI를 담당하는 View의 개발을 비지니스 로직 부분과 분리시켜 종속적이지 않은 형태로 개발할 수 있도록 해준다.
MVVM은 `Model-View-ViewModel` 의 세 가지 구성요소로 되었있다.

### Model
`Model`은 View, ViewModel과는 완전히 독립적으로 존재하며, 홀로 빌드가 가능해야 한다.
즉, UI와 관련 없는 로직들이 포함되어야 한다.

### ViewModel
`ViewModel`은 View와 Model 간의 브릿지 역할을 한다.
- View를 위한 데이터 저장
- View로부터 이벤트를 수신하고, 이에 맞는 Model의 함수 호출
- View로부터 이벤트를 수신하여 상태를 업데이트하고, 데이터 바인딩을 통해 View 업데이트

### View
`View`는 UI를 랜더링하는 부분이다.
사용자의 상호작용을 수신하고, 이를 Data Bunding을 통해 ViewModel에 전달한다.

## Reference
* [https://ko.wikipedia.org/wiki/모델-뷰-뷰모델](https://ko.wikipedia.org/wiki/모델-뷰-뷰모델){: target="_blank"}
* [https://levelup.gitconnected.com/implement-mvvm-in-swiftui-47f76dc28f1a](https://levelup.gitconnected.com/implement-mvvm-in-swiftui-47f76dc28f1a){: target="_blank"}
