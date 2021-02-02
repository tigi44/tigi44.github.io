---
title: "[Jekyll] Install and Serve Jekyll"
excerpt: "Install and Serve Jekyll"
description: "Install and Serve Jekyll"
modified: 2021-02-02
categories: "ETC"
tags: [Jekyll, CentOS, rvm, ruby, HomeBrew]
---

## Install Ruby

### rvm (CentOS)
```shell
$ yum install libyaml-devel glibc-headers autoconf gcc-c++ glibc-devel patch readline-devel zlib-devel libffi-devel openssl-devel automake libtool bison sqlite-devel

$ curl -sSL https://get.rvm.io | bash -s stable --ruby

// if fail with commands, execute commands
$ curl -sSL https://rvm.io/mpapis.asc | gpg --import -
$ curl -sSL https://get.rvm.io | bash -s stable --ruby

$ rvm version
$ ruby -v
$ gem -v
```


### yum (CentOS, Fedora, RHEL)
- CentOS, Fedora, RHEL은 yum 패키지 관리 시스템을 사용합니다.

```shell
$ sudo yum install ruby
$ ruby -v
```

### Homebrew (macOS)
- Homebrew는 macOS에서 일반적으로 사용되는 패키지 관리자입니다.

```shell
$ brew install ruby
$ ruby -v
```

## Install Jekyll
```shell
$ gem install jekyll bundler
$ jekyll -v
```

## Serve Jekyll
```shell
$ jekyll serve
or
$ bundle exec jekyll serve
or
$ bundle exec jekyll serve --host 0.0.0.0 --port 80
```

## Issue
- Jekyll serve fails on Ruby 3.0
   - [https://github.com/jekyll/jekyll/issues/8523](https://github.com/jekyll/jekyll/issues/8523){:target="_blank"}
