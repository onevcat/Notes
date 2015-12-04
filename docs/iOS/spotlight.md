# Spotlight 搜索

相关 API

- NSUserActivity
- Core Spotlight
- Web markup

### NSUserActivity

在 iOS 8 中 Handoff 被引入。iOS 9 中添加了一些额外的 property 来启用搜索。

为 model 建立 `NSUserActivity`，然后设置 `eligibleForSearch`：

```swift
// In model
public var userActivityUserInfo: [NSObject: AnyObject] {
  return ["id": objectId]
}

public var userActivity: NSUserActivity {
    let activity = NSUserActivity(activityType: "reverse dns domain name")
    activity.title = title
    activity.userInfo = userActivityUserInfo
    activity.keywords = [{localized key words}]
    return activity
}

// In vc for a model
let activity = xxx.userActivity
activity.eligibleForSearch = true

// 完成配置后可以将这个 activity 设置为 view controller 的 activity：
self.userActivity = activity
```

最后，override `updateUserActivityState`。系统将不时调用这个方法，需要保持 userInfo 最新。

```swift
override func updateUserActivityState(activity: NSUserActivity) {
  activity.addUserInfoEntriesFromDictionary(xxx.userActivityUserInfo)
}
```

### Core Spotlight

建立搜索索引，可以看做用来查询搜索信息的数据库。

通过设置合适的 `CSSearchableItemAttributeSet` 可以在 Spotlight 的 搜索结果中显示详细信息。

#### 在 CSSearchableItemAttributeSet 中设置唯一 id

通过设置 `relatedUniqueIdentifier` 可以把当前 activity 和其他的 (不一定是你的 app 中的) 进行区分，比如同一个地点 keyword 在不用 app 中都出现。CoreSpotlight 的 indexing 系统将通过这个 id 区分不同 activity 并进行优先级排序。

```swift
attributeSet.relatedUniqueIdentifier = {某个唯一 id}
```

在设置 `relatedUniqueIdentifier` 后，需要创建对应的 `CSSearchableItem` 提供给 CS 框架。

```swift
// In model
var searchableItem: CSSearchableItem {
    let item = CSSearchableItem(uniqueIdentifier: objectId, domainIdentifier: Employee.domainIdentifier, attributeSet: attributeSet)
    return item
}

let items = allItems()
let searchableItems = employees.map {
  $0.searchableItem
}
CSSearchableIndex.defaultSearchableIndex().indexSearchableItems(searchableItems) {
  error in
  if let error = error {
    print("Error indexing: \(error)")
  } else {
    print("Indexed")
  }
}
```

注意，普通的 `eligibleForSearch` 和 `index` 的结果在 app delegate 中对应的 activity 的类型是不同的。

search item -> 设置的 identifier，userInfo 中设置的 key 
indexed item -> CSSearchableItemActionType, CSSearchableItemActivityIdentifier

### Web markup

将网页内容映射到 app 侧。
