---
title: "[iOS] about dSYM"
excerpt: "about dSYM"
description: "about dSYM"
modified: 2024-02-22
categories: "iOS"
tags: [iOS, dSYM]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# dSYM?
- [https://jjhyuk15.medium.com/ios-dsym-이란-무엇인가-69516fa7ce99](https://jjhyuk15.medium.com/ios-dsym-이란-무엇인가-69516fa7ce99){: target="_blank"}

## Framework에 따른 dSYM
- Static Framework의 경우, 최종 결과물인 앱의 바이너리 안에 포함되기에 dSYM 파일이 별도로 생성되지 않는다. 하지만 Dynamic Framework의 경우, dSYM 파일이 생성된다.
- 외부 Dynamic Framework을 가져다 사용할 경우, 별도의 dSYM이 있어야 확인 가능

# Bitcode
- [https://hcn1519.github.io/articles/2020-05/bitcode_implementation](https://hcn1519.github.io/articles/2020-05/bitcode_implementation){: target="_blank"}

## Bitcode & dSYM
- Bitcode로 생성된 dSYM 파일과 최종적으로 앱스토어에 dSYM은 전혀 다른 파일이다
- Bitcode가 설정된 상태로 앱스토어에 업로드 할 경우, 아카이빙시 생성된 dSYM파일은 사용하지 못하고, 앱스토어내부에서 다시 컴파일된 dSYM를 다운 받아 사용해야 한다.(Xcode Organizer나 iTunes Connect에서..)

# Mach-O
- [https://zeddios.tistory.com/908](https://zeddios.tistory.com/908){: target="_blank"}
- Static Library
- Dynamic Library
