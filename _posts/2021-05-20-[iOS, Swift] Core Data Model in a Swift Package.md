---
title: "[iOS, Swift] Core Data Model Programmatically in a Swift Package"
excerpt: "Core Data Model Programmatically in a Swift Package"
description: "Core Data Model Programmatically in a Swift Package"
modified: 2021-05-20
categories: "iOS"
tags: [iOS, CoreData, Swift]

header:
  teaser: /assets/images/swift-teaser.png
---

# Data Model File in a XCode Project (.xcdatamodeld)

## Data Model
![datamodel](/assets/images/post/ios/coredata/datamodel.png)
![datamodel2](/assets/images/post/ios/coredata/datamodel2.png)

## XML
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<model type="com.apple.IDECoreDataModeler.DataModel" documentVersion="1.0" lastSavedToolsVersion="18154" systemVersion="20E241" minimumToolsVersion="Automatic" sourceLanguage="Swift" userDefinedModelVersionIdentifier="">
    <entity name="ExampleEntity" representedClassName="ExampleEntity" syncable="YES" codeGenerationType="class">
        <attribute name="name" optional="YES" attributeType="String"/>
        <attribute name="date" optional="YES" attributeType="Date" usesScalarValueType="NO"/>
    </entity>
</model>
```

# Create Core Data Model Programmatically

## 1. NSManagedObject

```swift
public final class ExampleEntity: NSManagedObject {
    @NSManaged var name: String
    @NSManaged var date: Date
}
```

## 2. NSEntityDescription

```swift
func entityDescription() -> NSEntityDescription {

    let entity = NSEntityDescription()
    entity.name = "ExampleEntity"
    entity.managedObjectClassName = NSStringFromClass(ExampleEntity.self)

    // Attributes
    let nameAttr = NSAttributeDescription()
    nameAttr.name = "name"
    nameAttr.attributeType = .stringAttributeType
    nameAttr.isOptional = true

    let dateAttr = NSAttributeDescription()
    dateAttr.name = "date"
    dateAttr.attributeType = .dateAttributeType

    entity.properties = [nameAttr,
                         dateAttr]

    return entity
}
```


## 3. NSManagedObjectModel

```swift
func managedObjectModel() -> NSManagedObjectModel {

    let model = NSManagedObjectModel()

    model.entities = [entityDescription()]

    return model
}
```

## 4. NSPersistentContainer

```swift
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentCloudKitContainer(name: "ExampleModel", managedObjectModel: managedObjectModel())

    container.loadPersistentStores(completionHandler: { (storeDescription, error) in
        if let error = error as NSError? {
            fatalError("Unresolved error \(error), \(error.userInfo)")
        }
    })
    return container
}()
```

# Reference
- [https://dmytro-anokhin.medium.com/core-data-and-swift-package-manager-6ed9ff70921a](https://dmytro-anokhin.medium.com/core-data-and-swift-package-manager-6ed9ff70921a){:target="_blank"}
