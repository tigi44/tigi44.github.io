---
title: "[WWDC21] What`s new in SwiftUI (SwiftUI 3.0)"
excerpt: "[WWDC21] What`s new in SwiftUI"
description: "[WWDC21] What`s new in SwiftUI"
modified: 2021-06-25
categories: "WWDC21"
tags: [WWDC21, SwiftUI, SwiftUI 3.0]

header:
  teaser: /assets/images/teaser/wwdc21-teaser.jpg
---

# Better Lists
## asyncImage view
- async, concurrency

```swift
// AsyncImage
AsyncImage(url: photo.url)
  .frame(width: 400, height:266)
  .mask(RoundedRectangle(cornerRadius: 16))


// placeholder
AsyncImage(url: photo.url) { image in
  image
    .resizable()
    .aspectRatio(contentMode: .fill)
} placeholder: {
  randomPlaceholderColor()
    .opacity(0.2)
}
.frame(width: 400, height:266)
.mask(RoundedRectangle(cornerRadius: 16))


// custom animations and error handling
AsyncImage(url: photo.url, transaction: .init(animation: .spring())) { phase in
  switch phase {
    case .empty:
      randomPlaceholderColor()
        .opacity(0.2)
        .transition(.opacity.combined(with: .scale))
    case .success(let image):
      image
        .resizable()
        .aspectRatio(contentMode: .fill)
        .transition(.opacity.combined(with: .scale))
    case .failure(let error):
      ErrorView(error)
    @unknown default:
      ErrorView()
  }
}
.frame(width: 400, height:266)
.mask(RoundedRectangle(cornerRadius: 16))
```

## refreshable modifier
- async, concurrency
- pull-to-refresh

```swift
NavigationView {
  List { ... }
  .refreshable {
    await photoStore.update()
  }
}
```

## task modifier
- async, concurrency
- first load
- automatically cancel when the view is removed

```swift
NavigationView {
  List { ... }
  .refreshable {
    await photoStore.update()
  }
  .tast {
    await photoStore.update()
  }
}


// iterating over an AsyncSequence
NavigationView {
  List { ... }
  .refreshable {
    await photoStore.update()
  }
  .tast {
    for await photo in photoStore.newestPhotos {
      photoStore.push(photo)
    }
  }
}
```

## interactive collections
- in List, more useful for a binding
- we can use the same technique in a ForEach view within our list instead

```swift
@State var directions: [Direction] = [ ... ]

...

NavigationView {
  List($directions) { $directions in
    Label {
      TextField("Instructions", text: $direction.text)
    } icon: {
      DirectionsIcon(direction)
    }
  }
}


// ForEach
NavigationView {
  List {
    ForEach($directions) { $directions in
      Label {
        TextField("Instructions", text: $direction.text)
      } icon: {
        DirectionsIcon(direction)
      }
    }
  }
}
```

## listRowSeparatorTint modifier
```swift
NavigationView {
  List {
    ForEach($directions) { $directions in
      Label {
        TextField("Instructions", text: $direction.text)
      } icon: {
        DirectionsIcon(direction)
      }
      .listRowSeparatorTint(Color.blue)
    }
  }
}
```

## listRowSeparator modifier
```swift
NavigationView {
  List {
    ForEach($directions) { $directions in
      Label {
        TextField("Instructions", text: $direction.text)
      } icon: {
        DirectionsIcon(direction)
      }
      .listRowSeparator(.hidden)
    }
  }
}
```

## swipeActions modifier
- custom swipe actions
- set swipe edge

```swift
NavigationView {
  List {
    ForEach($characters) { $character in
      CharacterProfile(character)
        .swipeActions(edge: .leading) {
          Button {
            togglePinned(for: $charecter)
          } label: {
            if charater.isPinned {
              Label("Unpin", systemImage: "pin.slash")
            } else {
              Label("Pin", systemImage: "pin")
            }
          }
          .tint(.yellow)
        }
        .swipeActions(edge: .trailing) { ... }
    }
  }
}
```

## macOS List : listStyle
```swift
@State private var selection = Set<StoryCharacter.ID>()

...

List(selection: $selection) { ... }
  .listStyle(.inset)

List(selection: $selection) { ... }
  .listStyle(.inset(alternatesRowBackgrounds: true))
```

# Beyond Lists
## Table
- on macOS

![table](/assets/images/post/wwdc21/swiftui/table.png)

```swift
Table(character) {
  TableColumn("<>") { charaterIcon($0) }
    .width(20)
  TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
    .width(40)
  TableColumn("Name", value: \.name)
  TableColumn("Powers", value: \.powers)
}


// selection, sorting
@State private var singleSelection: StoryCharater.ID?
@State private var sortOrder: [KeyPathComparator(\StoryCharacter.name)]
@State private var sorted: [StoryCharater]?

...

Table(character, selection: $singleSelection, sortOrder: $sortOrder) {
  TableColumn("<>") { charaterIcon($0) }
    .width(20)
  TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
    .width(40)
  TableColumn("Name", value: \.name)
  TableColumn("Powers", value: \.powers)
}
.onChange(of: characters) { sorted = $0.sorted(using: sortOrder) }
.onChange(of: sortOrder) { sorted = characters.sorted(using: $0) }
```

## FetchRequest
```swift
// CoreData tables
@FetchRequest(sortDescriptors: [sortDescriptor(\.name)])
private var characters: FetchRequest<StoryCharacter>
@State private var selection = Set<StoryCharacter.ID>()

Table(character, selection: $singleSelection, sortOrder: $charaters.sortDescriptors) {
  TableColumn("<>") { charaterIcon($0) }
    .width(20)
  TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
    .width(40)
  TableColumn("Name", value: \.name)
  TableColumn("Powers", value: \.powers)
}
```

## SectionedFetchRequest
```swift
@SelectionFetchRequest(
  sectionIdentifier: \.isPinned,
  sortDecsriptors: [
    SortDescriptor(\.isPinned, order: .reverse),
    SortDescriptor(\.lastModified)
  ],
  animation: .default)
private var characters: SectionedFetchRequest<...>

List {
  ForEach(characters) { section in
    Section(section.id ? "Pinned" : "Heros % Villains") {
      ForEach(section) { character in
        CharacterRowView(character)
      }
    }
  }
}
```

## Searchable modifier
```swift
NavigationView {
  List { ... }
  .listStyle(.sidebar)
  .searchable(text: $characters.filterText)
  .navigationTitle("characters")
}
```

## Drag previews
```swift
CharacterIcon(character)
  .onDrag {
    character.itemProvider
  } preview: {
    Label {
      Text(character.name)
    } icon: {
      CharacterIcon(character)
    }
  }
```

## ImportsItemProviders modifier
```swift
CharacterIcon(character)
  .onDrag { ... } preview: { ... }
  .importsItemProviders(StoryCharacter.imageAttachmebtTypes) { itemProviders in
    guard let first = itemProviders.first else { return false }
    async {
      character.headerImage = await StoryCharacter.loadHeaderImage(from: first)
    }
    return true
  }


// ImportFromDevicesCommand
WidowGroup {
  ContentView()
}
.commands {
  ImportFromDevicesCommand()
}
```

## ExportsItemProviders modifier
```swift
CharacterIcon(character)
  .onDrag { ... } preview: { ... }
  .importsItemProviders(StoryCharacter.imageAttachmebtTypes) { itemProviders in
    guard let first = itemProviders.first else { return false }
    async {
      character.headerImage = await StoryCharacter.loadHeaderImage(from: first)
    }
    return true
  }
  .exportsItemProviders(StoryCharacter.contetnTypes) { [character.itemProvider] }
```

# Advanced Graphics
## SF Symbols
- Monochrome, Multicolor, Hierarchical, Palett

![symbol](/assets/images/post/wwdc21/swiftui/symbol.png)

```swift
Image(systemName: "phone.down.circle")
  .symbolRenderingMode(.monochrome)

Image(systemName: "phone.down.circle")
  .symbolRenderingMode(.palette)
  .foregroundStyle(Color.cyan, Color.purple)


// Symbol variant
Image(systemName: "heart.circle.fill")

Image(systemName: "heart")
  .symbolVariant(.circle)
  .symbolVariant(.fill)
```

## Canvas View
- Canvas supports immediate-mode drawing similar to drawRect from UIKit or AppKit

```swift
let symbols = Array(repeating: Symbol("swift"), count 3166)

...

Canvas { context, size in
  let metrics = SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
  for (index, symbol) in symbols.enumerated() {
    let rect = mertics[index]
    let image = context.resolve(symbol.image)
    context.draw(image, in: rect.fit(image.size))
  }
}


// with gesture
@GestureState private var focalPoint: CGPoint? = nil

...

Canvas { context, size in
  let metrics = SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
  for (index, symbol) in symbols.enumerated() {
    let rect = mertics[index]
    let (sRect, opacity) = rect.fishEyeTransform(around: focalPoint)

    context.opacity = opacity
    let image = context.resolve(symbol.image)
    context.draw(image, in: sRect.fit(image.size))
  }
}
.gesture(DragGesture(minimumDistance: 0).updating($focalPoint) { value, focalPoint, _ in
  focalPoint = value.location
})


// with accessibility children
Canvas { context, size in
  let metrics = SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
  for (index, symbol) in symbols.enumerated() {
    let rect = mertics[index]
    let (sRect, opacity) = rect.fishEyeTransform(around: focalPoint)

    context.opacity = opacity
    let image = context.resolve(symbol.image)
    context.draw(image, in: sRect.fit(image.size))
  }
}
.gesture(DragGesture(minimumDistance: 0).updating($focalPoint) { value, focalPoint, _ in
  focalPoint = value.location
})
.accessibilityLabel("Symbol Browser")
.accessibilityChildren {
  List(symbols) {
    Text($0.name)
  }
}
```

## TimelineView
- created with a schedule

```swift
// Canvas with TimelineView
let symbols = Array(repeating: Symbol("swift"), count 3166)

...

TimelineView(.animation) {
  let time = $0.date.timeIntervalSince1970
  Canvas { context, size in
    let metrics = SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
    let focalPoint = focalPoint(at: time, in: size)
    for (index, symbol) in symbols.enumerated() {
      let rect = metrics[index]
      let (sRect, opacity) = rect.fishEyeTransform(around: focalPoint, at: time)

      context.opacity = opacity
      let image = context.resolve(symbol.image)
      context.draw(image, in: sRect.fit(image.size))
    }
  }
}

```

### TimelineSchedule
- like watchOS, TimelineView can preload using TimelineSchedule

## privacySensitive
- a watch enters the Always On state (watchOS 8)
- a widget in the lock screen

```swift
Image(systemName: favoriteSymbol)
  .privacySensitive(true)
```

## Background material
```swift
VStack { ... }
  .background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16.0))
```

## SafeAreaInset
```swift
ScrollView { ... }
  .safeAreaInset(edge: .bottom, spacing: 0) {
    VStack(spacing: 0) {
      Divider()
      VStack { ... }
    }
  }
```

## SwiftUI preview orientation
```swift
struct ColorList_Previews: PreviewProvider {
  static var previews: som View {
    ColorList()
      .previewInterfaceOrientation(.portrait)
    ColorList()
      .previewInterfaceOrientation(.landscapeLeft)
  }
}
```

# Text and Keyboard

## Text
### Markdown
- support `Markdown`
```swift
Text("**Hello**, world!")
Text("**Hello**, world! [WWDC](https://developer.apple.com/wwdc21/)")
```
- AttributedString
```swift
var formattedDate: AttributedString {
  var formattedDate: AttributedString = ...
  ...
  return formattedDate
}
```

- LocalizedString
![LocalizedString](/assets/images/post/wwdc21/swiftui/LocalizedString.png)

### Dynamic Type
- not support on macOS
- by system text size

```swift
Header("Today's Activities")
  .dynamicTypeSize(.large ... .extraExtraLarge)
```

### Text Selection
```swift
Text("info")
  .textSelection(.enabled)


VStack {
  ActivityHeader(...)
  Divider()
  Text("info")
}
.textSelection(.enabled)
```

### Text Formatting: List
```swift
people.map(\.nameComponents).formatted(.list(memberStyle: .name(style: .short), type: .and))
```

### TextField Formatting
```swift
TextField("New Person", value: $newAttendee, format: .name(style: .medium))
```

### TextField prompts and labels
```swift
TextField("Name:", text: $activity.name, promt: Text("New Activity"))
TextField("Location:", text: $activity.location)
```

### TextField submission
```swift
TextField("New Person", value: $newAttendee, format: .name(style: .medium))
  .onSubmit {
    activity.append(Person(newAttendee))
    newAttendd = PersonNameComponents()
  }
  .submitLabel(.done)
```

## Keyboard
### Toolbar

![Toolbar](/assets/images/post/wwdc21/swiftui/Toolbar.png)

```swift
Form { ... }
.toolbar {
  ToolbarItemGroup(placement: .keyboard) {
    Button(action: selectionPreviousField) {
      Label("Previous", systemImage: "chevron.up")
    }
    .disabled(!hasPreviousField)

    Button(action: selectionNextField) {
      Label("Previous", systemImage: "chevron.down")
    }
    .disabled(!hasNextField)
  }
}
```

## FocusState
```swift
@FocusState private var addAttendeeIdFocused: Bool

...

TextField("New Person", value: $newAttendee)
  .focused($addAttendeeIdFocused)
```

# More Buttons

![Buttons](/assets/images/post/wwdc21/swiftui/Buttons.png)

## Bordered Button
- on iOS

```swift
Button("Add") { ... }
.buttonStyle(.bordered)
.tint(.greed)

// view hierarchy
ScrollView {
  LazyVStack {
    ForEach(0..<10) { _ in
      Button("Add") { ... }
    }
  }
}
.buttonStyle(.bordered)
```

## Control Size and Prominence
```swift
HStack {
  ForEach(entry.tags) { tag in
    Button(tag.name) { ... }
  }
}
.buttonStyle(.bordered)
.controlSize(.small)
.controlProminence(.increased)


// Large buttons
HStack {
  ForEach(entry.tags) { tag in
    Button(tag.name) { ... }
  }
}
.buttonStyle(.bordered)
.controlSize(.large)


// Destructive buttons
ButtonEntryCell(entry)
  .contextMenu {
    Section {
      Button("Open") { ... }
      Button("Delete...", role: .destructive) { ... }
    }
  }
```

## Keyboard short cut
- return key on keyboard

```swift
Button("Add") { ... }
.keyboardShortcut(.defaultAction)
```

## Confirmation dialog
- iOS shows as an action sheet
- iPadOS as a popover
- macOS as an alert

```swift
ButtonEntryCell(entry)
  .contextMenu { ... }
  .confirmationDialog("Are you sure you want to delete \(pendingDeletion)?"), isPresented: $showConfirmation) {
    Button(Delete, role: .destructive) { // delete the entry }
  } message : {
    Text("Deleting \(pendingDeletion) will remove it from all of your jars")
  }
```

## Menu buttons
- on macOS

```swift
Menu("Add") {
  ForEach(jarStore.allJars) { jar in
    Button("Add to \(jar.name)") {
      jarStore.add(buttonEntry, to: jar)
    }
  }
} primaryAction: {
  jarStore.addToDefaultJar(buttonEntry)
}
.menuStyle(BorderedButtonMenuStyle())
.menuIndicator(.hidden)
```

## Toggle buttons
- on iOS

```swift
Toggle(isOn: $showOnlyNew) {
  Label("Show New Buttons", systemImage: "sparkles")
}
.toggleStyle(.button)
```

## Control group
```swift
ControlGroup {
  Button(action: archive) {
    Label("Archive", systemImage: "archiveBox")
  }
  button(action: delete) {
    Label("Delete", systemImage: "trash")
  }
}


ControlGroup {
  Menu {
    ForEach(history) { ... }
  } label: {
    Label("Back", systemImage: "chevron.backward")
  } primaryAction: {
    goBack(to: history[0])
  }
  .disabled(history.isEmpty)

  Menu {
    ForEach(forwardHistory) { ... }
  } label: {
    Label("Forward", systemImage: "chevron.forward")
  } primaryAction: {
    goForward(to: forwardHistory[0])
  }
  .disabled(forwardHistory.isEmpty)
}
```

# Reference
- [https://developer.apple.com/wwdc21/10018](https://developer.apple.com/wwdc21/10018){:target="_blank"}
