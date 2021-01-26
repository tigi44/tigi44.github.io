---
title: "[Jenkins] Parameterized Remote"
excerpt: "Jenkins Parameterized Remote Trigger"
description: "Jenkins Parameterized Remote Trigger"
modified: 2021-01-26
categories: "Jenkins"
tags: [Jenkins, Remote, Back-end, Server]

toc: true
toc_sticky: true

header:
  teaser: /assets/images/jenkins-logo.png
---

젠킨스 빌드 서버 여러대를 remote로 연결하여 한 곳에서 실행가능하도록 설정

## 1. Install Parameterized Remote Trigger PlugIn
- [Parameterized Remote Trigger](https://plugins.jenkins.io/Parameterized-Remote-Trigger/){:target="_blank"}

## 2. Setting System configuration options
- 젠킨스관리 > 시스템설정 > Parameterized Remote Trigger Configuration
- 원격으로 연결할 젠킨스서버 정보 입력

![setting_config](/assets/images/post/jenkins/remote_setting.png)

## 3. Setup Job
- 원격으로 연결할 job에 대한 정보 입력
- Server Info: 젠킨스 설정에서 입력한 정보 선택
- Job Info: 원격으로 실행할 잡이름 및 필요한 파라미터정보등 입력

![setup_job_1](/assets/images/post/jenkins/setup_job1.png)
![setup_job_2](/assets/images/post/jenkins/setup_job2.png)

## 4. build the Job
- 설정을 완료한 잡을 실행할 경우, 위 설정들로 연견된 젠킨스의 잡이 실행.
