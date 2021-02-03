---
title: "[zsh, ohmyzsh] Setting ZSH with Oh-My-Zsh on my terminal"
excerpt: "Installing zsh with oh-my-zsh"
description: "Installing zsh with oh-my-zsh"
modified: 2021-02-03
categories: "ETC"
tags: [zsh, oh-my-zsh, terminal, ohmyzsh]
---

## 0. What is ZSH
- [ZSH](https://www.zsh.org){:target="_blank"}
- [What is ZSH](https://www.howtogeek.com/362409/what-is-zsh-and-why-should-you-use-it-instead-of-bash/){:target="_blank"}

## 0. What is oh-my-zsh
- [oh-my-zsh](https://ohmyz.sh){:target="_blank"}
- [https://github.com/ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh){:target="_blank"}

## 1. Installing ZSH
- [Installing ZSH](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH){:target="_blank"}

### Default
```shell
// install zsh
$ sudo apt install zsh

// verify installation zsh
$ zsh --version

// confirm your authorized shells list
$ chsh -l
or
$ cat /etc/shells

// Make it your default shell
$ chsh -s $(which zsh)
or
$ chsh username -s $(which zsh)

// verify default shells
$ echo $SHELL
$ $SHELL --version
```
- [change shell in linux](https://zetawiki.com/wiki/리눅스_쉘_변경_chsh){:target="_blank"}

### MacOS
```shell
$ brew install zsh
```
> Since macOS 10.15 Catalina, the default shell has been changed to zsh

### Centos/RHEL
```shell
$ sudo yum update && sudo yum -y install zsh
```

## 2. Installing oh-my-zsh
```shell
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
![ohmyzsh](/assets/images/post/zsh/ohmyzsh.png)

- you can change themes, plugins and the other options in .zshrc

```shell
$ vi ~/.zshrc
```

## 3. Themes

```shell
$ vi ~/.zshrc
```

- Robby's theme is the default one.

```
ZSH_THEME="robbyrussell"
```

- To use a different theme, simply change the value to match the name of your desired theme

```
ZSH_THEME="agnoster"
```
![ohmyzsh-agnoster](/assets/images/post/zsh/ohmyzsh-agnoster.png)

> Note: many themes require installing [the Powerline Fonts](https://github.com/powerline/fonts){:target="_blank"} or [Naver D2Coding Font](https://github.com/naver/d2codingfont){:target="_blank"} in order to render properly.

- [External Themes](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes){:target="_blank"}

## 4. Plugins

```shell
$ vi ~/.zshrc
```

```
plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
)
```

### Installing zsh-syntax-highlighting

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting){:target="_blank"}

```shell
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### Installing zsh-autosuggestions

- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions){:target="_blank"}

```shell
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
