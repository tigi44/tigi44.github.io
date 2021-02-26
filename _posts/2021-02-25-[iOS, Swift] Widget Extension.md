---
title: "[iOS, Swift] Widget Extension"
excerpt: "Make a iOS Widget"
description: "Make a iOS Widget"
modified: 2021-02-25
categories: "iOS"
tags: [iOS, Widget, Swift]

header:
  teaser: /assets/images/swift-teaser.png
---

# Widget Extension
- iOS 14 Widget

![widgets](/assets/images/post/ios/widget/widgets.png)
![editablewidget](/assets/images/post/ios/widget/editablewidget.png)

# Example SourceCode
[https://github.com/tigi44/WidgetKitExample](https://github.com/tigi44/WidgetKitExample){:target="_blank"}

# About Widget HIG
- [https://developer.apple.com/design/human-interface-guidelines/ios/system-capabilities/widgets/](https://developer.apple.com/design/human-interface-guidelines/ios/system-capabilities/widgets/){:target="_blank"}

위젯은 단순히 앱을 여는 버튼이 아닌, 간략한 정보를 일정 주기로 업데이트하면서 보여주기 위한 용도

단순히 앱을 여는 단축키기능의 위젯은 사용자가 메인 홈 화면에 추가 하지 않는다....

위젯 사이즈는 스몰, 미디움, 라지 로 구분 (위젯 사이즈에 따른 가치를 두고 개발 지향, 사이즈에 맞는 정보 노출)

# Widget
- Languge: SwiftUI + WidgetKit
- Data Share: Userdefaults, Keychain....

![widget](/assets/images/post/ios/widget/widget.png)

## Widget Main

```swift
@main
struct WeatherWidget: Widget {
    let kind: String = "WeatherWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: WeatherWidgetProvider()) { entry in
            WeatherWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("Weather Widget")
        .description("This is an weather widget")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```
- kind
- configurationDisplayName
- description
- supportedFamilies

> WidgetBundle :
A container used to expose multiple widgets from a single widget extension.
```swift
@main
struct WeatherWidgetBundle: WidgetBundle {
    @WidgetBundleBuilder
    var body: some Widget {
        WeatherWidget()
        AirWidget()
    }
}
```

## Provider
### TimelineEntry
```swift
struct WeatherWidgetEntry: TimelineEntry {
    let date: Date
    let icon: String
    let location: String
    let temperature: Double
}
```
### TimelineProvider
```swift
struct WeatherWidgetProvider: TimelineProvider {
    func placeholder(in context: Context) -> WeatherWidgetEntry {
        WeatherWidgetEntry(date: Date(), icon: "", location: "", temperature: 0)
    }

    func getSnapshot(in context: Context, completion: @escaping (WeatherWidgetEntry) -> ()) {
        let entry = WeatherWidgetEntry(date: Date(), icon: "01d", location: "location", temperature: 0)
        completion(entry)
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [WeatherWidgetEntry] = []

        let currentDate = Date()
        for secondOffset in 0 ..< 4 {
            let entryDate = Calendar.current.date(byAdding: .second, value: secondOffset * 10, to: currentDate)!
            let weatherData = WeatherLocalData[secondOffset]
            let entry = WeatherWidgetEntry(date: entryDate, icon: weatherData["icon"] as! String, location: weatherData["location"]  as! String, temperature: weatherData["temp"]  as! Double)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}
```

> context ([https://zeddios.tistory.com/1091](https://zeddios.tistory.com/1091){:target="_blank"}) : isPreview, family, displaySize, environmentVariantes

- placeholder
  - 위젯 데이터 로딩전의 디폴트값으로 노출 가능
- getSnapshot
  - 위젯 설정 프리뷰 화면에서 호출 (context.isPreview)
- getTimeline
  - TimelineEntry를 통해 위젯 데이터 셋팅
  - TimelineEntry에 설정된 date 시간에 위젯에 데이터 노출
  - Timeline 상에 설정된 TimelineReloadPolicy 값에 따라 reload 실행

### Timeline

```swift
var entries: [TimelineEntry] = []
let timeline = Timeline(entries: entries, policy: .atEnd)
```

- TimelineEntry 배열과 TimelineReloadPolicy값을 갖고 있는 객체
- TimelineReloadPolicy
  - atEnd : TimelineEntry 배열을 모두 실행하고 난 후 reload
  ![TimelineReloadPolicyAtEnd](/assets/images/post/ios/widget/TimelineReloadPolicyAtEnd.png)

  - after(date) : TimelineEntry 배열 실행여부와 상관 없이 특정 시간 후 reload
  ![TimelineReloadPolicyAfter](/assets/images/post/ios/widget/TimelineReloadPolicyAfter.png)

  - never : TimelineEntry 배열만 실행


## View

![weatherwidget_small](/assets/images/post/ios/widget/weatherwidget_small.png)
![weatherwidget_medium](/assets/images/post/ios/widget/weatherwidget_medium.png)

```swift
struct WeatherWidgetEntryView : View {

    var entry: WeatherWidgetProvider.Entry
    @Environment(\.widgetFamily) var family

    ...

    var body: some View {
        switch family {
        case .systemSmall:
            WeahterSmallView(entry: entry)
        default:
            WeahterMediumView(entry: entry)
        }
    }
}
```

>Environment: widgetFamily (위젯 크기별 구분 가능)
>- .systemSmall
>- .systemMedium
>- .systemLarge

>Widget view size
>![widgetviewsize](/assets/images/post/ios/widget/widgetviewsize.png)

# Configuration Widget (Editable Widget)
![editablewidget](/assets/images/post/ios/widget/editablewidget.png)

## Create Intent Definition
- create a custom intent definition in a widget
![intentdefinition](/assets/images/post/ios/widget/intentdefinition.png)

- create the intent configuration and setting parameters
![intent](/assets/images/post/ios/widget/intent.png)

## Intent Widget Main
- Widget에서 StaticConfiguration 대신, IntentConfiguration 사용
- intent는 위에서 만든 커스텀 intent type 사용

```swift
@main
struct EditableWidget: Widget {
    let kind: String = "EditableWidget"

    var body: some WidgetConfiguration {
        IntentConfiguration(kind: kind, intent: ConfigurationIntent.self, provider: Provider()) { entry in
            EditableWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("Editable Widget")
        .description("This is an editable widget.")
    }
}
```

## Intent Provider
- TimelineProvider 대신, IntentTimelineProvider 사용
- configuration에 커스텀 intent type 사용

```swift
struct Provider: IntentTimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date(), configuration: ConfigurationIntent())
    }

    func getSnapshot(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date(), configuration: configuration)
        completion(entry)
    }

    func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []

        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate, configuration: configuration)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}
```

## Intents Extension (Dynamic parameter options)
![parameterlist](/assets/images/post/ios/widget/parameterlist.png)

- 커스텀하게 만든 Intent에서 parameter값을 동적 리스트로 변경하여 사용하고 싶다면, `Intents Extension` target을 추가하여 사용
- create a target : intents extension
![intentsextensiontarget](/assets/images/post/ios/widget/intentsextensiontarget.png)
![intentsextension2](/assets/images/post/ios/widget/intentsextension2.png)
- add the custom intent to supported intents
- setting target membership of the intentdefinition of the custom intent
![IntentDefinitionTargetMembership](/assets/images/post/ios/widget/IntentDefinitionTargetMembership.png)

- IntentHandler
![intentsextension](/assets/images/post/ios/widget/intentsextension.png)

- IntentHandler에 커스텀 intent type의 IntentHandling 구현, IntentHandling 안에서 동적 파라미터 리스트 입력 가능

```swift
class IntentHandler: INExtension {

    override func handler(for intent: INIntent) -> Any {
        // This is the default implementation.  If you want different objects to handle different intents,
        // you can override this and return the handler you want for that particular intent.

        return self
    }

}

extension IntentHandler: ConfigurationIntentHandling {
    func provideParameterOptionsCollection(for intent: ConfigurationIntent, with completion: @escaping (INObjectCollection<NSString>?, Error?) -> Void) {
        var items: [NSString] = []

        items.append("first string")
        items.append("second string")
        items.append("third string")
        items.append("forth string")

        completion(INObjectCollection(items: items), nil)
    }
}
```


# Widget Depp Link
- small 사이즈의 위젯경우, 위젯 전체에 링크가 걸리는 형태만 지원
- medium, large 사이즈 위젯에서는 `Link()`구현을 통해 각 리스트별 링크 동작 지원
- 위젯전체 딥링크 : .widgetURL()
- 리스트 일부 딥링크 : Link(){}

# Widget Center
- 앱내에서 위젯들의 동작을 컨트롤 가능

```swift
Button(action: {
            // WidgetCenter.shared.reloadTimelines(ofKind: "WeatherWidget")
            WidgetCenter.shared.reloadAllTimelines()
        }, label: {
            Text("reload all widget")
        })
```

# Issue
- 위젯뷰내 이미지 URL로 비동기 로드시 이슈
  - [https://stackoverflow.com/questions/63086029/ios-widgetkit-remote-images-fails-to-load](https://stackoverflow.com/questions/63086029/ios-widgetkit-remote-images-fails-to-load){:target="_blank"}
  - [https://developer.apple.com/forums/thread/652581](https://developer.apple.com/forums/thread/652581){:target="_blank"}
- configurable 위젯 reset 안되는 이슈
  - [https://stackoverflow.com/questions/63764083/how-to-reset-intent-extension-configurations-in-widgetkit](https://stackoverflow.com/questions/63764083/how-to-reset-intent-extension-configurations-in-widgetkit){:target="_blank"}

# Reference
- [https://medium.com/swlh/build-your-first-ios-widget-part-3-36ba53033e33](https://medium.com/swlh/build-your-first-ios-widget-part-3-36ba53033e33){:target="_blank"}
- [https://zeddios.tistory.com/1091](https://zeddios.tistory.com/1091){:target="_blank"}

- [https://developer.apple.com/design/human-interface-guidelines/ios/system-capabilities/widgets/](https://developer.apple.com/design/human-interface-guidelines/ios/system-capabilities/widgets/){:target="_blank"}
- [https://developer.apple.com/documentation/widgetkit](https://developer.apple.com/documentation/widgetkit){:target="_blank"}
- [https://developer.apple.com/documentation/swiftui/widgetbundle](https://developer.apple.com/documentation/swiftui/widgetbundle){:target="_blank"}
- [https://developer.apple.com/documentation/swiftui/widget](https://developer.apple.com/documentation/swiftui/widget){:target="_blank"}
- [https://developer.apple.com/documentation/widgetkit/timeline](https://developer.apple.com/documentation/widgetkit/timeline){:target="_blank"}
- [https://developer.apple.com/documentation/widgetkit/timelineentry](https://developer.apple.com/documentation/widgetkit/timelineentry){:target="_blank"}
- [https://developer.apple.com/documentation/widgetkit/timelineprovider](https://developer.apple.com/documentation/widgetkit/timelineprovider){:target="_blank"}
- [https://developer.apple.com/documentation/widgetkit/making-a-configurable-widget](https://developer.apple.com/documentation/widgetkit/making-a-configurable-widget){:target="_blank"}
- [https://developer.apple.com/documentation/widgetkit/widgetcenter](https://developer.apple.com/documentation/widgetkit/widgetcenter){:target="_blank"}
