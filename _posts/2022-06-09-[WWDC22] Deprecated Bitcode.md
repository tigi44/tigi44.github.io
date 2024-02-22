---
title: "[WWDC22] Deprecated Bitcode"
excerpt: "what`s new in Xcode14?"
description: "what`s new in Xcode14?"
modified: 2022-06-09
categories: "iOS"
tags: [iOS, WWDC22, bitcode]

header:
  teaser: /assets/images/teaser/wwdc21-teaser.png
---

# Bitcode

## Release Notes - Deprecated
- https://developer.apple.com/documentation/xcode-release-notes/xcode-14-release-notes

### General
```
Deprecations

Starting with Xcode 14, bitcode is no longer required for watchOS and tvOS applications, and the App Store no longer accepts bitcode submissions from Xcode 14.
Xcode no longer builds bitcode by default and generates a warning message if a project explicitly enables bitcode: “Building with bitcode is deprecated. Please update your project and/or target settings to disable bitcode.” The capability to build with bitcode will be removed in a future Xcode release. IPAs that contain bitcode will have the bitcode stripped before being submitted to the App Store. Debug symbols for past bitcode submissions remain available for download. (86118779)
```

### Build System
```
Deprecations

Because bitcode is now deprecated, builds for iOS, tvOS, and watchOS no longer include bitcode by default. (87590506)

The legacy build system has been removed. (90801041)

Building iOS projects with deployment targets for the armv7, armv7s, and i386 architectures is no longer supported. (92831716)
```
