---
categories: "NginX"
title: "[NginX] Install NginX on CentOS"
description: "Install NginX"
modified: 2020-06-29
tags: [NginX, CentOS, proxy, Web, Server, HTTP, HTTPS]
---

# 1. Install NginX Using yum
### add NginX to a repo
```shell
$ sudo vi /etc/yum.repos.d/nginx.repo
```

```shell
# add to `/etc/yum.repos.d/nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```
### install
```shell
$ sudo yum install -y nginx
```

# 2. Firewall
### open 8080 port for NginX
```shell
firewall-cmd --permanent --zone=public --add-port=8080/tcp
firewall-cmd --reload
firewall-cmd --list-ports
```

# 3. Configure NginX Server
```shell
$ sudo vi /etc/nginx/conf.d/default.conf
```
```shell
server {
  listen       8080;
  server_name  localhost;

  location / {
    proxy_pass http://localhost:3000; #proxy server port
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    proxy_buffering off;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```
### Test Setting
```shell
$ sudo nginx -t
```
### Reload Setting
```shell
$ sudo nginx -s reload
```


# 4. Execute NginX demon
```shell
$ systemctl start nginx
$ systemctl enable nginx #for booting the server
```

# # Configure for SSL (HTTP)
```shell
$ sudo vi /etc/nginx/conf.d/default.conf
```
```shell
server {
  listen       80;
  listen       443 ssl;
  server_name  localhost;

  ssl_certificate  /etc/nginx/ssl/server.crt;
  ssl_certificate_key /etc/nginx/ssl/server.key;

  location / {
    proxy_pass http://localhost:3000;
    ...
  }
}
```
### Redirect All Requests to HTTPS
```shell
server {
    listen 80 default_server;

    server_name _;

    return 301 https://$host$request_uri;
}
```
```shell
server {
  listen 80;

  server_name foo.com;
  return 301 https://foo.com$request_uri;
}
```

# # Issues NginX
### 502 Permission (Bad Gateway)
```shell
# Setting for httpd of SELinux
$ sudo setsebool -P httpd_can_network_connect on
```
### Register a port to SELinux
```shell
$ sudo semanage port -a -t http_port_t -p tcp 8089
```

# Reference
- https://cofs.tistory.com/412
- https://www.sitepoint.com/configuring-nginx-ssl-node-js/
- https://velog.io/@jakeseo_me/Node에서-NGINX를-리버스-프록시로-사용하기-번역
- https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/
- https://serversforhackers.com/c/redirect-http-to-https-nginx
- https://cofs.tistory.com/411
