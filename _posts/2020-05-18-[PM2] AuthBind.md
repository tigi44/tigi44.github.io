---
title: "[PM2] AuthBind"
excerpt: "PM2 AuthBind"
description: "PM2 AuthBind"
last_modified_at: 2020-05-18T14:00:00+09:00
categories: "ETC"
tags: [Development, PM2, AuthBind]

toc: true

header:
  teaser: https://pm2.keymetrics.io/assets/logo.png
---

## PM2 AuthBind
- [PM2](https://pm2.keymetrics.io){: target="_blank"}
- PM2를 이용하여, 80이나 443과 같은 root권한이 필요한 프로세스를 올릴 경우 `AuthBind` 필요

## install
```shell
$ sudo apt-get install authbind
or
$ sudo yum install authbind
$ wget https://s3.amazonaws.com/aaronsilber/public/authbind-2.1.1-0.1.x86_64.rpm
$ sudo rpm -Uvh https://s3.amazonaws.com/aaronsilber/public/authbind-2.1.1-0.1.x86_64.rpm
```

## execute
```shell
$ sudo touch /etc/authbind/byport/%port%
$ sudo chown %user% /etc/authbind/byport/%port%
$ sudo chown centos /etc/authbind/byport/%port%
$ sudo chmod 755 /etc/authbind/byport/%port%
$ authbind --deep pm2 update
$ authbind --deep pm2 start ecosystem.config.js
$ authbind --deep pm2 start ecosystem.config.js --env production
```

## confirm port
```shell
$ netstat -tnlp
```

## Reference
- [https://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/#listening-on-port-80-wo-root](https://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/#listening-on-port-80-wo-root){: target="_blank"}
