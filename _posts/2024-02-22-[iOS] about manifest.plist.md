---
title: "[[iOS] about manifest.plist"
excerpt: "about manifest.plist"
description: "about manifest.plist"
modified: 2024-02-22
categories: "iOS"
tags: [iOS, manifest, plist]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# install enterprise app

## manifest.plist
- 엔터프라이즈용 앱설치를 위해, 앱설치시 필요한 정보?
- [https://developer.apple.com/documentation/devicemanagement/installenterpriseapplicationcommand/command/manifest/](https://developer.apple.com/documentation/devicemanagement/installenterpriseapplicationcommand/command/manifest/){: target="_blank"}

### assets
- 설치시 필요한 items 설정
    - item
        - kind
        - url
- item > kind :
    - asset-pack-manifest : on-demand resource 용으로 사용
        - [https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/IntrotoHostingODR.html#//apple_ref/doc/uid/TP40015083-CH17-SW1](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/IntrotoHostingODR.html#//apple_ref/doc/uid/TP40015083-CH17-SW1){: target="_blank"}
    - display-image : 앱 설치시 로딩? 이미지
        - a fully-qualified URL pointing to a 57×57-pixel (72x72 for iPad) PNG icon used during download and installation
    - full-size-image :
        - a fully-qualified URL pointing to a 512×512-pixel PNG image that represents the iTunes app
- [https://developer.apple.com/documentation/devicemanagement/manifesturl/itemsitem/assetsitem](https://developer.apple.com/documentation/devicemanagement/manifesturl/itemsitem/assetsitem){: target="_blank"}

### metadata
- 설치 앱 정보
    - bundle-identifier
    - kind
    - bundle-version
    - title
- [https://developer.apple.com/documentation/devicemanagement/manifesturl/itemsitem/metadata](https://developer.apple.com/documentation/devicemanagement/manifesturl/itemsitem/metadata){: target="_blank"}

## create manifest.plist
```sh
PlistFile=$WORKSPACE/archiveOutput/${BUILD_SCHEME}.xcarchive/Info.plist
ManifestPlist=$WORKSPACE/archiveOutput/${BUILD_SCHEME}.ipa/manifest.plist

AssetsKind=software-package
Title=APP_NAME
MetadataKind=software
Version=$(/usr/libexec/PlistBuddy -c "Print ApplicationProperties:CFBundleShortVersionString" $PlistFile)
BundleIdentifier=$(/usr/libexec/PlistBuddy -c "Print ApplicationProperties:CFBundleIdentifier" $PlistFile)

rm -f $ManifestPlist

/usr/libexec/PlistBuddy -c "Add :items array" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0 Dict" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0:assets array" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0:assets:0 Dict" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0:assets:0:kind string $AssetsKind" $ManifestPlist

/usr/libexec/PlistBuddy -c "Add :items:0:metadata:title string $Title" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:bundle-version string $Version" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:bundle-identifier string $BundleIdentifier" $ManifestPlist
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:kind string $MetadataKind" $ManifestPlist
```


# Reference
- [https://stackoverflow.com/questions/44022642/ios-app-enterprise-distribution-deployment-missing-app-plist](https://stackoverflow.com/questions/44022642/ios-app-enterprise-distribution-deployment-missing-app-plist){: target="_blank"}
