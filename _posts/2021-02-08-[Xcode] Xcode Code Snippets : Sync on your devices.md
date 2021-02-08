---
title: "[Xcode] Xcode Code Snippets : Sync on your devices"
excerpt: "Xcode Code Snippets : Sync on your devices"
description: "Xcode Code Snippets : Sync on your devices"
modified: 2021-02-08
categories: "iOS"
tags: [Xcode, Snippet, Sync]
---

## Create a Snippet on Xcode

- Create

![xcode-code-snippet](/assets/images/post/snippet/create-snippet.png)

- Edit (Command + Shift + L)

![xcode-code-snippet](/assets/images/post/snippet/edit-snippet.png)

## Sync the code snippets using the iCloud Drive

- Code snippets path

```shell
/Users/{USERNAME}/Library/Developer/Xcode/UserData/CodeSnippets/
```

- Copy the code snippets to the iCloud Drive

```shell
$ mkdir ~/Library/Mobile\ Documents/com~apple~CloudDocs/XcodeUserData

$ mv -v /Users/{USERNAME}/Library/Developer/Xcode/UserData/CodeSnippets ~/Library/Mobile\ Documents/com~apple~CloudDocs/XcodeUserData
```

- Link the code snippets with your devices

```shell
$ rm /Users/{USERNAME}/Library/Developer/Xcode/UserData/CodeSnippets

$ ln -s ~/Library/Mobile\ Documents/com~apple~CloudDocs/XcodeUserData/CodeSnippets /Users/{USERNAME}/Library/Developer/Xcode/UserData/
```


## Reference
- [https://medium.com/@nictheawesome/sync-your-xcode-code-snippets-across-devices-because-you-can-733816be1d89](https://medium.com/@nictheawesome/sync-your-xcode-code-snippets-across-devices-because-you-can-733816be1d89){:target="_blank"}
- [https://sarunw.com/posts/how-to-create-code-snippets-in-xcode/](https://sarunw.com/posts/how-to-create-code-snippets-in-xcode/){:target="_blank"}
