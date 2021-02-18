---
title: "[Linux] CentOS(RHEL 7) Firewall : firewalld"
excerpt: "CentOS(RHEL 7) Firewall : firewalld"
description: "CentOS(RHEL 7) Firewall : firewalld"
modified: 2021-02-08
categories: "ETC"
tags: [Linux, CentOS, Firewall, firewalld, RHEL7]
---

## 1. Install firewalld
### Install
```shell
$ yum install firewalld
```

### Enable & Start
```shell
$ systemctl enable firewalld
$ systemctl start firewalld
```

## 2. Setting Zone

### Zone list
```shell
$ firewall-cmd --get-zones
$ firewall-cmd --list-all-zones
$ firewall-cmd --get-default-zone
$ firewall-cmd --get-active-zone
$ firewall-cmd --permanent --list-all --zone=newzone
```

### Add Zone
```shell
$ firewall-cmd --permanent --new-zone=newzone
```
> When `firewall-cmd --reload` is executed after firewall setup, it is initialized. So you have to set `--permanent`

### Remove Zone
```shell
$ firewall-cmd --permanent --delete-zone=newzone
```

### Set Default Zone
```shell
$ firewall-cmd --set-default-zone=newzone  
```

## 3. Setting Service
### Service list
```shell
$ firewall-cmd --get-services
$ firewall-cmd --list-services --zone=public
$ firewall-cmd --permanent --list-all --zone=public
```

### Add Service
```shell
$ firewall-cmd --permanent --zone=newzone --add-service=http
$ firewall-cmd --permanent --zone=newzone --add-service=https
```

### Remove Service
```shell
$ firewall-cmd --permanent --zone=newzone --remove-service=http
```

## 4. Setting Port

### Add Port
```shell
$ firewall-cmd --permanent --zone=newzone --add-port=4000/tcp
$ firewall-cmd --permanent --zone=newzone --add-port=8000-9000/tcp
```

### Remove Port
```shell
$ firewall-cmd --permanent --zone=newzone --remove-port=4000/tcp
```

## 5. Reload firewall
```shell
$ firewall-cmd --reload
```

## 6. Restart firewalld
```shell
$ systemctl restart firewalld
```

## # Allow IP
```shell
$ firewall-cmd --permanent --zone=public --add-source=111.111.1.0/24 --add-port=22/tcp
```

## # White list by adding a rich-rule
```shell
$ sudo firewall-cmd --zone=public --permanent --add-rich-rule="rule family='ipv4' source address='10.x.x.x' accept"
```

## # Ban IP by adding a rich-rule
```shell
$ sudo firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.x.x.0/24' reject"
$ sudo firewall-cmd --permanent --zone=dmz --add-rich-rule="rule family='ipv4' source address='10.x.x.0/24' drop"
```


## Reference
- [https://firewalld.org](https://firewalld.org){:target="_blank"}
- [https://www.lesstif.com/system-admin/rhel-centos-firewall-22053128.html](https://www.lesstif.com/system-admin/rhel-centos-firewall-22053128.html){:target="_blank"}
- [https://stackoverflow.com/questions/24729024/open-firewall-port-on-centos-7](https://stackoverflow.com/questions/24729024/open-firewall-port-on-centos-7){:target="_blank"}
