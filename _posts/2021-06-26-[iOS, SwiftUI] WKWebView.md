---
title: "[iOS, SwiftUI] WKWebView in SwiftUI"
excerpt: "WKWebView in SwiftUI"
description: "WKWebView in SwiftUI"
modified: 2021-06-26
categories: "iOS"
tags: [iOS, SwiftUI, WKWebView, WebView]

header:
  teaser: /assets/images/teaser/swiftui-teaser.png
---

# UIViewRepresentable
- for using UIKit

```swift
import SwiftUI
import WebKit

struct WebUIView: UIViewRepresentable {
    var url: String

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    func makeUIView(context: Context) -> WKWebView {
        let preferences = WKPreferences()
        preferences.javaScriptCanOpenWindowsAutomatically = false

        let configuration = WKWebViewConfiguration()
        configuration.preferences = preferences

        let webView = WKWebView(frame: CGRect.zero, configuration: configuration)
        webView.uiDelegate = context.coordinator
        webView.navigationDelegate = context.coordinator
        webView.allowsBackForwardNavigationGestures = true
        webView.scrollView.isScrollEnabled = true

        if let url = URL(string: url) {
            webView.load(URLRequest(url: url))
        }

        return webView
    }

    func updateUIView(_ webView: WKWebView, context: Context) {

    }
}

// Coordinator
class Coordinator : NSObject {

    var parent: WebUIView

    init(_ parent: WebUIView) {
        self.parent = parent
    }
}

// WKNavigationDelegate
extension Coordinator: WKNavigationDelegate {

    func webView(_ webView: WKWebView,
                   decidePolicyFor navigationAction: WKNavigationAction,
                   decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {

        decisionHandler(.allow)
    }
}

// WKUIDelegate
extension Coordinator: WKUIDelegate {

}
```

# WebView
- use in SwiftUI


```swift
struct WebView: View {
    let url: String

    var body: some View {
        WebUIView(url: url)
    }
}

struct WebView_Previews: PreviewProvider {
    static var previews: some View {
        WebView(url: "https://www.apple.com")
    }
}
```
