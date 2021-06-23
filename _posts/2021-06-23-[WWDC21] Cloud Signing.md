---
title: "[WWDC21] Distribute apps in Xcode with cloud signing (Xcoude build with cloud signing)"
excerpt: "[WWDC21] Distribute apps in Xcode with cloud signing"
description: "Xcoude build with cloud signing"
modified: 2021-06-23
categories: "WWDC21"
tags: [WWDC21, Xcode, Xcode13, Cloud Signing, Xcode Build]

header:
  teaser: /assets/images/teaser/wwdc21-teaser.jpg
---

# Xcode13

# Archiving
```shell
$ xcodebuild -project $WORKSPACE/sampleapp.xcodeproj \
  -scheme sampleapp-scheme \
  clean archive -archivePath ~/Documents/sampleapp/sampleapp.xcarchive
```

# Exporting
## Export iPA file

- sampleapp_appstore_upload.plist file
![export_plist](/assets/images/post/wwdc21/cloudsigning/export_plist.png)

- Export sampleapp.ipa
```shell
$ xcodebuild -exportArchive \
  -archivePath ~/Documents/sampleapp/sampleapp.xcarchive \
  -exportOptionsPlist ~/Documents/sampleapp/sampleapp_appstore_upload.plist \
  -exportPath ~/Documents/sampleapp/sampleapp.ipa
```

## Automating Distribution With cloud signing

- sampleapp_appstore_upload.plist file
![cloudsinging_plist](/assets/images/post/wwdc21/cloudsigning/cloudsinging_plist.png)

- Use App Store Connect Keys
```shell
$ xcodebuild -exportArchive \
  -archivePath ~/Documents/sampleapp/sampleapp.xcarchive \
  -exportOptionsPlist ~/Documents/sampleapp/sampleapp_appstore_upload.plist \
  -authenticationKeyIssuerID xxxxxxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -authenticationKeyID xxxxxxxxxx \
  -authenticationKeyPath ~/PrivateKeys/AuthKey_xxxxxxxxx.p8 \
  -allowProvisioningUpdates
```

- If you sign in With Xcode, the only required flag will be '-allowProvisioningUpdates'
```shell
$ xcodebuild -exportArchive \
  -archivePath ~/Documents/sampleapp/sampleapp.xcarchive \
  -exportOptionsPlist ~/Documents/sampleapp/sampleapp_appstore_upload.plist \
  -allowProvisioningUpdates
```

# Reference
- [https://developer.apple.com/wwdc21/10204](https://developer.apple.com/wwdc21/10204){:target="_blank"}
