---
layout: post
categories: "iOS"
title: "[iOS, Objective-c] UIActivityViewController : Share File"
description: "UIActivityViewController"
modified: 2020-05-20
tags: [iOS, Objective-c, UIActivityViewController]
---

## UIActivityViewController
```obj-c

NSURL *sFilePathURL = [NSURL fileURLWithPath:aFilePath];
UIActivityViewController *sActivityController = [[UIActivityViewController alloc] initWithActivityItems:@[sFilePathURL] applicationActivities:nil];
[self presentViewController:sActivityController animated:YES completion:nil];

```
## ScreenShot
<figure>
	<a href="{{ site.url }}/images/post/ios/UIActivityViewController.png"><img src="{{ site.url }}/images/post/ios/UIActivityViewController.png" alt=""></a>
	<figcaption>UIActivityViewController ScreenShot</figcaption>
</figure>
