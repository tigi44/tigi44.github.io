---
title: "[Xcode] Suppressing warning: incompatible pointer types"
excerpt: "Suppressing warning: incompatible pointer types"
description: "Suppressing warning: incompatible pointer types"
modified: 2021-02-17
categories: "iOS"
tags: [Xcode, iOS, Suppressing warning]
---

## Warning
![warning](/assets/images/post/xcode/warning.png)

## Suppressing warning
```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wincompatible-pointer-types"

// warning code

#pragma clang diagnostic pop
```

![Suppressing warning](/assets/images/post/xcode/suppressingwarning.png)
