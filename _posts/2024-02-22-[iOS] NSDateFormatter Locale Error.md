---
title: "[iOS] NSDateFormatter Locale Error"
excerpt: "NSDateFormatter Locale Error"
description: "NSDateFormatter Locale Error"
modified: 2024-02-22
categories: "iOS"
tags: [iOS, NSDateFormatter, Locale]

toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# 이슈
- 현재 시간 포맷(yyyyMMddHHmmssSSS)에서 `20230515175713092 -> 20230515오후 55713092` 이렇게 오전/오후 스트링이 추가된 상태로 나오는 이슈 발생

```obj-c
+ (NSString *)dateString
{
    NSDateFormatter *sDateFormat    = [[NSDateFormatter alloc] init];
    sDateFormat.dateFormat          = @"yyyyMMddHHmmssSSS";

    ...

    NSString *sDate                 = [sDateFormat stringFromDate:[NSDate date]];

    ...
}
```

# 원인
>When working with fixed format dates, such as RFC 3339, you set the dateFormat property to specify a format string. For most fixed formats, you should also set the locale property to a POSIX locale ("en_US_POSIX"), and set the timeZone property to UTC.

- 날짜 및 시간를 표기하는 포맷방식은 지역에 따라 다르며, 표준 포맷으로는 `ISO 8601(국제 표준)`, `RFC 3339(인터넷 표준)` 등이 있음
    - 지역에 따라, 날짜 및 시간을 표기하는 포맷이 다를 수 있음
- iOS에서는 이렇게 지역에 따라 달라지는 포맷을 하나로 통일하기 위해, `Locale(identifier: "en_US_POSIX")`을 설정할 수 있도록 지원
    - en-US의 날짜 및 시간 표기법을 따르도록 설정

## 이슈 재현
- iOS 국가설정에서 영국, 프랑스, 일본등 시간 표기시 24시간 포맷을 기본으로 사용하는 국가로 선택
- iOS 시간제 설정에서 24시간제 지원 설정을 끄고 12시간 포맷을 사용하도록 설정
- 코드상에서 NSDateFormatter 사용시 별도의 locale 설정없이, `HH` 시간포맷을 사용하면 24시간제를 지원 할 수 없는 상태가 되면서 `오후 5` 로 노출됨
- 한국이나, 미국등의 국가로 설정할시에는 24시간제 설정에 상관없이 `HH`이 정상적으로 24시간제로 노출

# 해결 방안
- NSDateFormatter 설정시 locale에 `Locale(identifier: "en_US_POSIX")` 값을 설정하여 고정된 24시간제 노출이 되도록 설정

```obj-c
+ (NSString *)dateString
{
    NSDateFormatter *sDateFormat    = [[NSDateFormatter alloc] init];
    sDateFormat.locale              = [NSLocale localeWithLocaleIdentifier:@"en_US_POSIX"];
    sDateFormat.dateFormat          = @"yyyyMMddHHmmssSSS";

    ...

    NSString *sDate                 = [sDateFormat stringFromDate:[NSDate date]];

    ...
}
```

# Reference
- [https://stackoverflow.com/questions/29374181/nsdateformatter-hh-returning-am-pm-on-ios-8-device](https://stackoverflow.com/questions/29374181/nsdateformatter-hh-returning-am-pm-on-ios-8-device){: target="_blank"}
- [https://developer.apple.com/documentation/foundation/nsdateformatter](https://developer.apple.com/documentation/foundation/nsdateformatter){: target="_blank"}
- [https://developer.apple.com/library/archive/qa/qa1480/_index.html](https://developer.apple.com/library/archive/qa/qa1480/_index.html){: target="_blank"}
- [https://developer.apple.com/forums/thread/700869](https://developer.apple.com/forums/thread/700869){: target="_blank"}
