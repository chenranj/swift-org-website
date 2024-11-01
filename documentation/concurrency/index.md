---
layout: page
date: 2024-03-03 12:00:00
title: 启用完整并发检查
---

Swift 6 语言模式中的数据竞争安全性设计支持增量迁移。您可以逐个模块地解决项目中的数据竞争安全性问题，并且可以在 Swift 5 语言模式下将编译器的 actor 隔离和 `Sendable` 检查设置为警告，这样您可以在开启 Swift 6 语言模式之前评估消除数据竞争的进度。

可以使用 `-strict-concurrency` 编译器标志在 Swift 5 语言模式下将完整的数据竞争安全检查设置为警告。

## 使用 Swift 编译器

要在命令行直接运行 `swift` 或 `swiftc` 时启用完整并发检查，请传入 `-strict-concurrency=complete`：

```
~ swift -strict-concurrency=complete main.swift
```

## 使用 SwiftPM

### 在 SwiftPM 命令行调用中

可以使用 `-Xswiftc` 标志在 Swift 包管理器命令行调用中传入 `-strict-concurrency=complete`：

```
~ swift build -Xswiftc -strict-concurrency=complete
~ swift test -Xswiftc -strict-concurrency=complete
```

这对于在按照下一节所述在包清单中永久添加该标志之前，评估并发警告的数量很有帮助。

### 在 SwiftPM 包清单中

要在使用 Swift 5.9 或 Swift 5.10 工具的 Swift 包中为目标启用完整并发检查，请在给定目标的 Swift 设置中使用 [`SwiftSetting.enableExperimentalFeature`](https://developer.apple.com/documentation/packagedescription/swiftsetting/enableexperimentalfeature(_:_:))：

```swift
.target(
  name: "MyTarget",
  swiftSettings: [
    .enableExperimentalFeature("StrictConcurrency")
  ]
)
```

当使用 Swift 6.0 或更高版本的工具时，请在给定目标的 Swift 设置中使用 [`SwiftSetting.enableUpcomingFeature`](https://developer.apple.com/documentation/packagedescription/swiftsetting/enableupcomingfeature(_:_:))：

```swift
.target(
  name: "MyTarget",
  swiftSettings: [
    .enableUpcomingFeature("StrictConcurrency")
  ]
)
```

## 使用 Xcode

要在 Xcode 项目中启用完整并发检查，请在 Xcode 构建设置中将"Strict Concurrency Checking"设置为"Complete"。或者，您可以在 xcconfig 文件中将 `SWIFT_STRICT_CONCURRENCY` 设置为 `complete`：

```
// 在 Settings.xcconfig 中

SWIFT_STRICT_CONCURRENCY = complete;
```
