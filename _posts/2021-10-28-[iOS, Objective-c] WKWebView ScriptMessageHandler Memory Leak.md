---
title: "[iOS, Objective-c] WKWebView ScriptMessageHandler Memory Leak"
excerpt: "WKWebView ScriptMessageHandler Memory Leak"
description: "WKWebView ScriptMessageHandler Memory Leak"
modified: 2021-10-28
categories: "iOS"
tags: [iOS, Objective-c, WKWebView, WKScriptMessageHandler, MemoryLeak]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# addScriptMessageHandler
```obj-c

@interface WebViewController() <WKScriptMessageHandler>

...

@end

@implementation WebViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    WKWebViewConfiguration *sConfiguration      = [[WKWebViewConfiguration alloc] init];
    WKUserContentController *sContentController = [[WKUserContentController alloc] init];

    [sContentController addScriptMessageHandler:self name:@"message"];
    [sConfiguration     setUserContentController:sContentController];

    self.webview = [[WKWebView alloc] initWithFrame:CGRectZero configuration:sConfiguration];

    ...
}

...

- (void)userContentController:(nonnull WKUserContentController *)userContentController didReceiveScriptMessage:(nonnull WKScriptMessage *)message
{
  ...
}

@end
```

# Memory leak

```obj-c
WKUserContentController *sContentController = [[WKUserContentController alloc] init];
[sContentController addScriptMessageHandler:self name:@"message"];
```

- the `WKUserContentController` holds a strong reference to the message handler `self`
- so it causes a `retain cycle`

# Solution #1

- remove the message handler

```obj-c
self.webview.configuration.userContentController.removeScriptMessageHandlerForName("message")
```

# Solution #2

- create a object to solve the retain cycle

- objective-c

```obj-c
@interface WeakScriptMessageDelegate : NSObject<WKScriptMessageHandler>

@property (nonatomic, weak) id<WKScriptMessageHandler> scriptDelegate;

- (instancetype)initWithDelegate:(id<WKScriptMessageHandler>)scriptDelegate;

@end

@implementation WeakScriptMessageDelegate

- (instancetype)initWithDelegate:(id<WKScriptMessageHandler>)scriptDelegate
{
    self = [super init];
    if (self) {
        _scriptDelegate = scriptDelegate;
    }
    return self;
}

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    [self.scriptDelegate userContentController:userContentController didReceiveScriptMessage:message];
}

@end

// add message hanlder
WKUserContentController *sContentController = [[WKUserContentController alloc] init];
[sContentController addScriptMessageHandler:[[WeakScriptMessageDelegate alloc] initWithDelegate:self] name:@"message"];
```

- swift

```swift
class LeakAvoider : NSObject, WKScriptMessageHandler {

    weak var delegate : WKScriptMessageHandler?

    init(delegate:WKScriptMessageHandler) {
        self.delegate = delegate
        super.init()
    }

    func userContentController(userContentController: WKUserContentController, didReceiveScriptMessage message: WKScriptMessage) {
            self.delegate?.userContentController(userContentController, didReceiveScriptMessage: message)
    }
}

// add message hanlder
self.webview.configuration.userContentController.addScriptMessageHandler(LeakAvoider(delegate:self), name: "message")
```

# Reference
- [https://stackoverflow.com/questions/26383031/wkwebview-causes-my-view-controller-to-leak](https://stackoverflow.com/questions/26383031/wkwebview-causes-my-view-controller-to-leak){:target="_blank"}
