---
title: "[Jenkins] Build with Parameter(git branches)"
excerpt: "Jenkins Build with Parameter(git branches)"
description: "Jenkins Build with Parameter(git branches)"
modified: 2021-01-26
categories: "Jenkins"
tags: [Jenkins, Build, git, Back-end, Server]

toc: true
toc_sticky: true

header:
  teaser: /assets/images/jenkins-teaser.png
---

젠킨스 잡 실행시, git 브랜치들을 파라미터로 선택하여 실행하도록 설정

## 1. Install List Git Branches Parameter PlugIn
- [List Git Branches Parameter](https://plugins.jenkins.io/list-git-branches-parameter/){:target="_blank"}

## 2. Setup Job
- 깃 브랜치 파라미터가 필요한 잡에 설정

![git_param](/assets/images/post/jenkins/git_param.png)

## 3. Build with Parameter
- 잡 실행시 브랜치 리스트를 받아와 선택할 수 있음

![git_param_job](/assets/images/post/jenkins/git_param_job.png)

## 4. Source code
- 아래와 같은 방법으로 선택된 파라미터 값으로 해당 브랜치의 소스코드를 사용할 수 있음

![git_param_source](/assets/images/post/jenkins/git_param_source.png)
