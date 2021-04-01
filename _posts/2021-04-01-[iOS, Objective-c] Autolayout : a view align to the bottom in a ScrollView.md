---
title: "[iOS, Objective-c] Autolayout : a View align to the bottom in a ScrollView"
excerpt: "Autolayout : a View align to the bottom in a ScrollView"
description: "Autolayout : a View align to the bottom in a ScrollView"
modified: 2021-04-01
categories: "iOS"
tags: [iOS, Autolayout, Objective-c]

header:
  teaser: /assets/images/ios-teaser.png
---

# a View align to the bottom in a ScrollView

![thirdHeight_50](/assets/images/post/ios/autolayout/thirdHeight_50.png)

![thirdHeight_450](/assets/images/post/ios/autolayout/thirdHeight_450.png)

![thirdHeight_600](/assets/images/post/ios/autolayout/thirdHeight_600.png)

## ViewController Code
<script src="https://gist.github.com/tigi44/510fd2c23d643e57cd5eebfe73e6c5bf.js"></script>

## Setup ScrollView (GreenColor View)
```obj-c
- (void)setupScrollView
{
    _scrollView = [UIScrollView new];
    _scrollView.translatesAutoresizingMaskIntoConstraints = false;
    _scrollView.backgroundColor = UIColor.greenColor;
    _scrollView.showsHorizontalScrollIndicator = false;
    _scrollView.showsVerticalScrollIndicator = false;
    [self.view addSubview:_scrollView];
    [NSLayoutConstraint activateConstraints:@[
        [_scrollView.topAnchor constraintEqualToAnchor:self.view.topAnchor constant:UIApplication.sharedApplication.windows.firstObject.safeAreaInsets.top],
        [_scrollView.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor],
        [_scrollView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor constant:-UIApplication.sharedApplication.windows.firstObject.safeAreaInsets.bottom],
        [_scrollView.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor],
    ]];
}
```

## Setup ContentView (YellowColor View)

- Set the height constraint of contentView to `Priority: UILayoutPriorityDefaultLow`

```obj-c
- (void)setupContentView
{
    _contentView = [UIView new];
    _contentView.translatesAutoresizingMaskIntoConstraints = false;
    _contentView.backgroundColor = UIColor.yellowColor;
    [self.scrollView addSubview:_contentView];
    [NSLayoutConstraint activateConstraints:@[
        [_contentView.topAnchor constraintEqualToAnchor:self.scrollView.topAnchor],
        [_contentView.leadingAnchor constraintEqualToAnchor:self.scrollView.leadingAnchor],
        [_contentView.bottomAnchor constraintEqualToAnchor:self.scrollView.bottomAnchor],
        [_contentView.trailingAnchor constraintEqualToAnchor:self.scrollView.trailingAnchor],
        [_contentView.widthAnchor constraintEqualToAnchor:self.scrollView.widthAnchor]
    ]];

    NSLayoutConstraint *sHeightOfContentView = [_contentView.heightAnchor constraintEqualToAnchor:self.scrollView.heightAnchor];
    sHeightOfContentView.priority = UILayoutPriorityDefaultLow;
    sHeightOfContentView.active = true;
}
```

## Setup Labels

- The important thing is a `greater-than-or-equal` constraint between thirdLabel and bottomLabel
- `The location of bottmLabel` changes depending on `the height of thirdLabel`.

```obj-c
- (void)setupLabels
{
    _firstLabel = [UILabel new];
    _firstLabel.translatesAutoresizingMaskIntoConstraints = false;
    _firstLabel.backgroundColor = UIColor.grayColor;
    _firstLabel.text = @"firstLabel";
    [self.contentView addSubview:_firstLabel];
    [NSLayoutConstraint activateConstraints:@[
        [_firstLabel.topAnchor constraintEqualToAnchor:self.contentView.topAnchor constant:kMargin.top],
        [_firstLabel.leadingAnchor constraintEqualToAnchor:self.contentView.leadingAnchor constant:kMargin.left],
        [_firstLabel.trailingAnchor constraintEqualToAnchor:self.contentView.trailingAnchor constant:-kMargin.right],
        [_firstLabel.heightAnchor constraintEqualToConstant:50.f]
    ]];

    _secondLabel = [UILabel new];
    _secondLabel.translatesAutoresizingMaskIntoConstraints = false;
    _secondLabel.backgroundColor = UIColor.grayColor;
    _secondLabel.text = @"secondLabel";
    [self.contentView addSubview:_secondLabel];
    [NSLayoutConstraint activateConstraints:@[
        [_secondLabel.topAnchor constraintEqualToAnchor:self.firstLabel.bottomAnchor constant:40],
        [_secondLabel.leadingAnchor constraintEqualToAnchor:self.contentView.leadingAnchor constant:kMargin.left],
        [_secondLabel.trailingAnchor constraintEqualToAnchor:self.contentView.trailingAnchor constant:-kMargin.right],
        [_secondLabel.heightAnchor constraintEqualToConstant:50.f]
    ]];

    _thirdLabel = [UILabel new];
    _thirdLabel.translatesAutoresizingMaskIntoConstraints = false;
    _thirdLabel.backgroundColor = UIColor.grayColor;
    _thirdLabel.text = @"thirdLabel";
    [self.contentView addSubview:_thirdLabel];
    [NSLayoutConstraint activateConstraints:@[
        [_thirdLabel.topAnchor constraintEqualToAnchor:self.secondLabel.bottomAnchor constant:40],
        [_thirdLabel.leadingAnchor constraintEqualToAnchor:self.contentView.leadingAnchor constant:kMargin.left],
        [_thirdLabel.trailingAnchor constraintEqualToAnchor:self.contentView.trailingAnchor constant:-kMargin.right],
        [_thirdLabel.heightAnchor constraintEqualToConstant:50.f]
    ]];

    _bottomLabel = [UILabel new];
    _bottomLabel.translatesAutoresizingMaskIntoConstraints = false;
    _bottomLabel.backgroundColor = UIColor.redColor;
    _bottomLabel.text = @"bottomLabel";
    [self.contentView addSubview:_bottomLabel];
    [NSLayoutConstraint activateConstraints:@[
        [_bottomLabel.topAnchor constraintGreaterThanOrEqualToAnchor:self.thirdLabel.bottomAnchor constant:40],
        [_bottomLabel.leadingAnchor constraintEqualToAnchor:self.contentView.leadingAnchor constant:kMargin.left],
        [_bottomLabel.bottomAnchor constraintEqualToAnchor:self.contentView.bottomAnchor constant:-kMargin.bottom],
        [_bottomLabel.trailingAnchor constraintEqualToAnchor:self.contentView.trailingAnchor constant:-kMargin.right],
        [_bottomLabel.heightAnchor constraintEqualToConstant:50.f]
    ]];
}
```
