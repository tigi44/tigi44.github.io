---
title: "[iOS, SwiftUI] Privacy View in SwiftUI (with PrivacySensitive and Redacted)"
excerpt: "Privacy View in SwiftUI (with PrivacySensitive and Redacted)"
description: "Privacy View in SwiftUI (with PrivacySensitive and Redacted)"
modified: 2022-01-26
categories: "iOS"
tags: [iOS, Swift, SwiftUI, PrivacySensitive, Redacted]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# PrivacySensitive
- [https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/privacysensitive(\_:)](https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/privacysensitive(_:)){:target="_blank"}

```swift
struct BankAccountView: View {
    var body: some View {
        VStack {
            Text("Account #")

            Text(accountNumber)
                .font(.headline)
                .privacySensitive() // Hide only the account number.
        }
    }
}
```

# Redacted
- [https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/redacted(reason:)](https://developer.apple.com/documentation/swiftui/outlinesubgroupchildren/redacted(reason:)){:target="_blank"}
- .redacted(reason: .placeholder)
- .redacted(reason: .privacy)
- .unredacted()

|unredacted|redacted|
|-|-|
|![unredacted](/assets/images/post/ios/redacted/unredacted.png)|![redacted](/assets/images/post/ios/redacted/redacted.png)|

```swift
struct BankAccountView: View {
    var body: some View {
        VStack {
            Image(systemName: "banknote")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 100)

            Text("Account #")
                .unredacted() // show account text

            Text("1234567890")
                .font(.headline)
        }
        .redacted(reason: .placeholder)
    }
}
```


# Privacy Redaction
- [https://developer.apple.com/documentation/swiftui/redactionreasons/privacy](https://developer.apple.com/documentation/swiftui/redactionreasons/privacy){:target="_blank"}

```swift
struct BankingContentView: View {
   @Environment(.redactionReasons) var redactionReasons

   var body: some View {
       if redactionReasons.contains(.privacy) {
           FullAppCover()
       } else {
           AppContent()
       }
   }
}
```
