---
title: "[iOS, Swift] URL issue for decodable struct in iOS16 and below"
excerpt: "iOS16이하 버전에서 한글이 포함된 URL을 사용할 때 Decodable URL 이슈 발생"
description: "URL issue for decodable struct in iOS16 and below"
modified: 2024-03-04
categories: "iOS"
tags: [iOS, Swift, Decodable, URL, Struct]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

- iOS16이하 버전에서 한글이 포함된 URL을 사용할 때 Decodable URL 이슈 발생 (iOS17 부터는 한글 이슈 없음), iOS16이하에서는 URL 대신 EncodingURL로 사용

<script src="https://gist.github.com/tigi44/66f4d6ac7bf91a8fe1b85559eaf6ba2a.js"></script>
