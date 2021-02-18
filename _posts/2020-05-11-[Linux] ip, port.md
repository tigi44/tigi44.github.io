---
title: "[Linux] IP, Port, Server, Firewall ..."
excerpt: "[Linux] IP, Port, Server, Firewall ..."
description: "[Linux] IP, Port, Server, Firewall ..."
last_modified_at: 2020-05-11T14:00:00+09:00
categories: "ETC"
tags: [Development, Linux, IP, Port, Firewall, Server]
---

## 열려있는 포트 프로세스 닫기
```shell
$ sudo lsof -i :portNumber
```

## 현재 열려있는 포트를 확인
```shell
$ netstat -tulpn | grep LISTEN
$ netstat -anp | grep LISTEN | grep :80
$ lsof -i -n -P | grep TCP | grep LISTEN
```

## firewall 오픈
```shell
$ systemctl start firewalld
```

## firewall 허용 port 리스트
```shell
$ sudo firewall-cmd --zone=public --list-ports
```

## 포트가 외부에서 접속되지 않는다면 포트를 방화벽에 추가합니다.
```shell
$ sudo firewall-cmd --zone=public --add-port=8000/tcp --permanent
```

## 방화벽을 리로드합니다.
```shell
$ sudo firewall-cmd --reload
```

## 원격서버 포트 접근 확인
```shell
$ telnet 34.73.59.253 8080
```

## linux iptables
```shell
$ sudo iptables -L
$ sudo iptables -Ln
$ sudo iptables -t nat -L
$ sudo iptables -t nat -L --line-numbers
$ sudo iptables -t nat -D PREROUTING 1

$ sudo sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080
```

## linux 리소스
```shell
// 디스크 용량
$ df
$ df -h
// 특정 디렉토리 용량
$ du -sh *
// 리소스 확인
$ top
```
