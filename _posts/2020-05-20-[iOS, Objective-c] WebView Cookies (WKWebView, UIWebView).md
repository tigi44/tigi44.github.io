---
layout: post
categories: "iOS"
title: "[iOS, Objective-c] WebView Cookies (WKWebView, UIWebView)"
description: "WebView Cookies (WKWebView, UIWebView)"
modified: 2020-05-20
tags: [iOS, Objective-c, WebView, cookies, WKWebView, UIWebView]
---

# WKWebView
## Setup WKWebview for sharing Cookies : common WKProcessPool

```obj-c

@implementation WKWebViewPoolHandler

+ (WKProcessPool *)commonPool {
    static WKProcessPool *sWKProcessPool;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sWKProcessPool = [[WKProcessPool alloc] init];
    });
    return sWKProcessPool;
}

@end


...


- (WKWebViewConfiguration *)configuration
{
    if (_configuration == nil)
    {
        _configuration = [[WKWebViewConfiguration alloc] init];
        _configuration.processPool = [WKWebViewPoolHandler commonPool];
    }

    return _configuration;
}


...


self.webView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:self.configuration];

```
## Get WKWebview Cookies (then set cookies to a request)
```obj-c

if (@available(iOS 11.0, *)) {

    [WKWebsiteDataStore.defaultDataStore.httpCookieStore getAllCookies:^(NSArray<NSHTTPCookie *> *aCookies) {

        NSMutableURLRequest *sRequest             = [NSMutableURLRequest requestWithURL:sURL];
        NSDictionary        *sCookieHeaderFields  = [NSHTTPCookie requestHeaderFieldsWithCookies:aCookies];
        [sRequest setAllHTTPHeaderFields:sCookieHeaderFields];
    }];
}

```

```obj-c

if (@available(iOS 11.0, *)) {

    [self.webView.configuration.websiteDataStore.httpCookieStore getAllCookies:^(NSArray<NSHTTPCookie *> *aCookies) {

        NSMutableURLRequest *sRequest             = [NSMutableURLRequest requestWithURL:sURL];
        NSDictionary        *sCookieHeaderFields  = [NSHTTPCookie requestHeaderFieldsWithCookies:aCookies];
        [sRequest setAllHTTPHeaderFields:sCookieHeaderFields];
    }];
}

```

## Get Cookies from a response
```obj-c

#pragma mark - WKNavigationDelegate

- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler
{
    NSDictionary *sHeaders = ((NSHTTPURLResponse *)navigationResponse.response).allHeaderFields;
    NSArray      *sCookies = [NSHTTPCookie cookiesWithResponseHeaderFields:sHeaders forURL:sURL];

    for (NSHTTPCookie *sCookie in sCookies) {
        [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:sCookie];
    }

    decisionHandler(WKNavigationResponsePolicyAllow);
}

```


# UIWebView (NSHTTPCookieStorage)
## Setup Cookies policy
```obj-c
[NSHTTPCookieStorage sharedHTTPCookieStorage].cookieAcceptPolicy = NSHTTPCookieAcceptPolicyAlways;
```

## Get Cookies from NSHTTPCookieStorage
```obj-c

NSArray<NSHTTPCookie *> *sCookies = [NSHTTPCookieStorage sharedHTTPCookieStorage].cookies;

```
