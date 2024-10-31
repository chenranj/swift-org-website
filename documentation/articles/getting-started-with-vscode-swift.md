---
layout: page
date: 2024-05-28 12:00:00
title: 为 Swift 开发配置 VS Code
author: [matthewbastien, plemarquand]
---

[Visual Studio Code](https://code.visualstudio.com/) (VS Code) 是一个流行的通用编辑器，通过扩展性支持多种语言。Swift 扩展为编辑器带来了 Swift 语言特定的功能，为在所有平台上开发 Swift 应用程序提供了无缝的体验。

Swift 扩展包括：

- 语法高亮和代码补全
- 代码导航功能，如转到定义和查找所有引用
- 代码重构和快速修复
- 支持 Swift Package Manager 的包管理和任务
- 丰富的调试支持
- 使用 XCTest 或 Swift Testing 框架进行测试

Swift 扩展设计用于支持以下项目：

- Swift Package Manager 项目（例如使用 `Package.swift`）
- 可以生成 `compile_commands.json` 的项目（例如使用 CMake）

## 安装扩展

1. 首先，安装 Swift。如果您的系统上还没有安装 Swift，请参阅 [Swift.org 上的入门指南](/getting-started/)。
2. 下载并安装 [Visual Studio Code](https://code.visualstudio.com/Download)。
3. 从 [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=sswg.swift-lang) 安装 Swift 扩展，或直接在 VS Code 扩展面板中安装。

![从扩展面板安装 vscode-swift 扩展](/assets/images/getting-started-with-vscode-swift/installation.png)

## 创建新的 Swift 项目

要创建新的 Swift 项目，您可以使用 Swift 扩展中的 `Swift: Create New Project...` 命令来引导您完成这个过程。您可以通过打开命令面板并按照以下说明找到此命令。

- macOS：`CMD + Shift + P`
- 其他平台：`Ctrl + Shift + P`

![创建新项目命令显示可用的项目模板](/assets/images/getting-started-with-vscode-swift/create-new-project/select-project-template.png)

1. 从模板列表中选择您想要创建的项目类型。
2. 选择存储项目的目录。
3. 为您的项目命名。
4. 打开新创建的项目。系统会提示您在当前窗口中打开项目、在新窗口中打开项目，或将其添加到当前工作区。默认行为可以通过使用 `swift.openAfterCreateNewProject` 设置进行配置。

## 语言功能

Swift 扩展使用 [SourceKit-LSP](https://github.com/swiftlang/sourcekit-lsp) 来提供语言功能。SourceKit-LSP 在编辑器中提供以下功能。使用这些链接查看每个主题的 VS Code 文档：

- [代码补全](https://code.visualstudio.com/docs/editor/intellisense)
- [转到定义](https://code.visualstudio.com/docs/editor/editingevolved#_go-to-definition)
- [查找所有引用](https://code.visualstudio.com/Docs/editor/editingevolved#_peek)
- [重命名重构](https://code.visualstudio.com/docs/editor/refactoring#_rename-symbol)
- [诊断](https://code.visualstudio.com/docs/editor/editingevolved#_errors-warnings)
- [快速修复](https://code.visualstudio.com/docs/editor/editingevolved#_code-action)

SourceKit-LSP 还提供代码操作来自动化常见任务。VS Code 中的代码操作以灯泡的形式出现在编辑器边缘附近（参见下面的截图示例）。点击灯泡将显示可用的操作，其中可能包括：

- 向您的 `Package.swift` 添加目标
- 将 JSON 转换为协议
- 为您的函数添加文档

![Package swift 操作](/assets/images/getting-started-with-vscode-swift/language-features/package_actions.png)

<div class="warning" markdown="1">
在使用语言功能之前，您必须在项目上执行 `swift build` 命令，可以在命令行或使用 VS Code 中的任务来执行。这会填充 SourceKit-LSP 中的索引。
</div>

## Swift 任务

Visual Studio Code 提供任务作为运行外部工具的方式。请参阅 [通过任务与外部工具集成](https://code.visualstudio.com/docs/editor/tasks) 文档了解更多信息。

Swift 扩展提供了一些内置任务，您可以使用这些任务通过 Swift Package Manager 构建您的项目。您还可以通过在项目根文件夹中创建 `tasks.json` 文件来配置自定义任务。例如，这个 `tasks.json` 以发布模式构建您的 Swift 目标：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "swift",
      "label": "Swift Build All - Release",
      "detail": "swift build --build-tests",
      "args": ["build", "--build-tests", "-c", "release"],
      "env": {},
      "cwd": "${workspaceFolder}",
      "group": "build"
    }
  ]
}
```

上述任务被配置为属于 `build` 组。这意味着它将出现在 `run build tasks` 菜单中，该菜单可以在 macOS 上使用 `CMD + Shift + B` 或在其他平台上使用 `Ctrl + Shift + B` 打开：

![运行构建任务菜单](/assets/images/getting-started-with-vscode-swift/tasks/build-tasks.png)

在构建期间发生的任何错误都会作为诊断出现在编辑器中，与 SourceKit-LSP 提供的诊断一起显示。运行另一个构建任务会清除上一个构建任务的诊断。

## 调试

Visual Studio Code 提供丰富的调试体验。有关更多信息，请参阅 [调试](https://code.visualstudio.com/docs/editor/debugging) 文档。

Swift 扩展依赖 [Code-LLDB 扩展](https://github.com/vadimcn/vscode-lldb) 来启用调试支持。

<div class="warning" markdown="1">
首次启动 VS Code 时，Swift 扩展会提示您配置 LLDB 的设置。您需要将配置应用到全局（用户设置）或您的工作区（工作区设置）中，调试器才能正常工作。

![配置调试器](/assets/images/getting-started-with-vscode-swift/debugging/configure-lldb.png)

</div>

默认情况下，扩展会为 Swift 包中的每个可执行目标创建一个启动配置。您可以通过在项目根文件夹中添加 `launch.json` 文件来自行配置这些设置。例如，这个 `launch.json` 使用自定义参数启动 Swift 可执行文件：

```json
{
  "configurations": [
    {
      "type": "lldb",
      "name": "Debug swift-executable",
      "request": "launch",
      "sourceLanguages": ["swift"],
      "args": ["--hello", "world"],
      "cwd": "${workspaceFolder}",
      "program": "${workspaceFolder}/.build/debug/swift-executable",
      "preLaunchTask": "swift: Build Debug swift-executable"
    }
  ]
}
```

您可以通过 VS Code 中的调试视图启动调试会话。

1. 选择您想要调试的启动配置。
2. 点击绿色播放按钮启动调试会话。

可执行文件将被启动，您可以在 Swift 代码中设置断点，这些断点将在代码执行时被命中。

下面的截图显示了调试 Hello World 程序的示例。它在断点处暂停，您可以看到调试视图显示了作用域中变量的值。您还可以将鼠标悬停在编辑器中的标识符上以查看其变量值：

![调试](/assets/images/getting-started-with-vscode-swift/debugging/debugging.png)

## 测试资源管理器

Visual Studio Code 在左侧边栏提供了一个测试资源管理器视图，可用于：

- 导航到测试
- 运行测试
- 调试测试

Swift 扩展支持 [XCTest](https://developer.apple.com/documentation/xctest) 以及 [Swift Testing](https://swiftpackageindex.com/apple/swift-testing/main/documentation/testing)。当您编写测试时，它们会自动添加到测试资源管理器中。

![内联测试错误](/assets/images/getting-started-with-vscode-swift/testing/inline_assertion_failures.png)

要调试测试：

1. 设置断点
2. 使用 `Debug Test` 配置文件运行测试、测试套件或整个测试目标。

`Run Test with Coverage` 配置文件会检测被测试的代码，并在测试运行完成时打开代码覆盖率报告。当您浏览已覆盖的文件时，在测试期间执行的行号显示为绿色，而未执行的行号显示为红色。将鼠标悬停在行号上会显示已覆盖行被执行的次数。可以使用 `Test: Show Inline Coverage` 命令显示或隐藏行执行计数。

带有 [标签](https://swiftpackageindex.com/apple/swift-testing/main/documentation/testing/addingtags) 的 Swift Testing 测试可以在测试资源管理器中使用 `@TestTarget:tagName` 进行过滤。然后您可以运行或调试过滤后的测试列表。

<div class="warning" markdown="1">
Swift 扩展不支持在 Swift 5.10 或更早版本中运行 Swift Testing 测试。
</div>

## 高级工具链选择

Swift 扩展会自动检测您安装的 Swift 工具链。但是，它还提供了一个名为 `Swift: Select Toolchain...` 的命令，如果您安装了多个工具链，可以用它来在工具链之间进行选择。

<div class="warning" markdown="1">
这是一个高级功能，用于将 VS Code 配置为使用机器上默认工具链以外的工具链。建议在 macOS 上使用 `xcode-select` 或在 Linux 上使用 `swiftly` 来全局切换工具链。
</div>

系统可能会提示您选择在哪里配置这个新路径。您的选项是：

- 保存在用户设置中
- 保存在工作区设置中

请记住，工作区设置优先于用户设置：

![设置选择](/assets/images/getting-started-with-vscode-swift/toolchain-selection/configuration.png)

然后 Swift 扩展会提示您重新加载扩展以使用新的工具链。您必须这样做，否则扩展将无法正常工作：

![重新加载 VS Code 警告](/assets/images/getting-started-with-vscode-swift/toolchain-selection/reload.png)

## 贡献者

该扩展由 Swift 社区成员开发，并由 [Swift 服务器工作组](/sswg/) 维护。
