---
layout: page
title: 创建一个命令行工具
---

> 本指南的源代码可在 [GitHub](https://github.com/apple/swift-getting-started-cli) 上找到。

  
{% include getting-started/_installing.md %}

 引导
---

  
让我们用新的 Swift 开发环境编写一个小应用程序。首先，我们将使用 SwiftPM 为我们创建一个新项目。在您选择的终端中，运行：

    $ mkdir MyCLI
    $ cd MyCLI
    $ swift package init --name MyCLI --type executable
    

  
这将生成一个名为 MyCLI 的新目录，其中包含以下文件：

    .
    ├── Package.swift
    └── Sources
        └── main.swift
    

  
`Package.swift` 是 Swift 的清单文件。您可以在此处保存项目的元数据以及依赖项。

  
`Sources/main.swift` 是应用程序的入口点，我们将在其中编写应用程序代码。

  
事实上，SwiftPM 为我们生成了一个“Hello， world！”项目！

  
我们可以通过在终端中运行 `swift run` 来运行该程序。

    $ swift run MyCLI
    Building for debugging...
    [3/3] Linking MyCLI
    Build complete! (0.68s)
    Hello, world!
    

 添加依赖项
------

  
基于 Swift 的应用程序通常由提供有用功能的库组成。

  
在这个项目中，我们将使用一个名为 [example-package-figlet](https://github.com/apple/example-package-figlet) 的包，它将帮助我们制作 ASCII 艺术。

  
你可以在 [Swift Package Index](https://swiftpackageindex.com) 上找到更多有趣的库 -- 这是 Swift 的非官方包索引。

  
为此，我们使用以下信息扩展了 `Package.swift` 文件：

    // swift-tools-version: 5.8
    // The swift-tools-version declares the minimum version of Swift required to build this package.
    
    import PackageDescription
    
    let package = Package(
        name: "MyCLI",
        dependencies: [
          .package(url: "https://github.com/apple/example-package-figlet", branch: "main"),
        ],
        targets: [
            // Targets are the basic building blocks of a package, defining a module or a test suite.
            // Targets can depend on other targets in this package and products from dependencies.
            .executableTarget(
                name: "MyCLI",
                dependencies: [
                    .product(name: "Figlet", package: "example-package-figlet"),
                ],
                path: "Sources"),
        ]
    )
    

  
运行 `swift build` 将指示 SwiftPM 下载新的依赖项，然后继续构建代码。

  
运行此命令还为我们创建了一个新文件 `Package.resolved`。此文件是我们在本地使用的依赖项的确切版本的快照。

 一个小应用程序
--------

  
首先删除 `main.swift`。我们将用名为 `MyCLI.swift` 的新文件替换它。将以下代码添加到其中：

    import Figlet
    
    @main
    struct FigletTool {
      static func main() {
        Figlet.say("Hello, Swift!")
      }
    }
    

  
这为应用提供了一个新的入口点，如果需要，该入口点可以是异步的。您可以有一个 `main.swift` 文件或一个 `@main`入口点，但不能两者兼而有之。

  
通过`导入 Figlet`，我们现在可以使用 `example-package-figlet` 包导出的 `Figlet` 模块。

  
一旦我们保存了它，我们就可以使用 `swift run` 运行我们的应用程序 假设一切顺利，您应该看到您的应用程序将其打印到屏幕上：

      _   _          _   _                 ____               _    __   _     _ 
     | | | |   ___  | | | |   ___         / ___|  __      __ (_)  / _| | |_  | |
     | |_| |  / _ \ | | | |  / _ \        \___ \  \ \ /\ / / | | | |_  | __| | |
     |  _  | |  __/ | | | | | (_) |  _     ___) |  \ V  V /  | | |  _| | |_  |_|
     |_| |_|  \___| |_| |_|  \___/  ( )   |____/    \_/\_/   |_| |_|    \__| (_)
                                    |/                                          
    

 参数解析
-----

  
大多数命令行工具都需要能够解析命令行参数。

  
为了将此功能添加到我们的应用程序中，我们添加了对 [swift-argument-parser](https://github.com/apple/swift-argument-parser) 的依赖项。

  
为此，我们使用以下信息扩展了 `Package.swift` 文件：

    // swift-tools-version: 5.8
    // The swift-tools-version declares the minimum version of Swift required to build this package.
    
    import PackageDescription
    
    let package = Package(
        name: "MyCLI",
        dependencies: [
          .package(url: "https://github.com/apple/example-package-figlet", branch: "main"),
          .package(url: "https://github.com/apple/swift-argument-parser", from: "1.0.0"),
        ],
        targets: [
            // Targets are the basic building blocks of a package, defining a module or a test suite.
            // Targets can depend on other targets in this package and products from dependencies.
            .executableTarget(
                name: "MyCLI",
                dependencies: [
                    .product(name: "Figlet", package: "example-package-figlet"),
                    .product(name: "ArgumentParser", package: "swift-argument-parser"),
                ],
                path: "Sources"),
        ]
    )
    

  
现在，我们可以导入 `swift-argument-parser` 提供的参数解析模块，并在我们的应用程序中使用它：

    import Figlet
    import ArgumentParser
    
    @main
    struct FigletTool: ParsableCommand {
      @Option(help: "Specify the input")
      public var input: String
    
      public func run() throws {
        Figlet.say(self.input)
      }
    }
    

  
有关 [swift-argument-parser](https://github.com/apple/swift-argument-parser) 如何解析命令行选项的更多信息，请参阅 [swift-argument-parser 文档](https://github.com/apple/swift-argument-parser)文档。

  
一旦我们保存了它，我们就可以使用 `swift run MyCLI --input 'Hello， world！'`

  
请注意，在这种情况下，我们需要指定可执行文件，以便我们可以将`输入`参数传递给它。

  
假设一切顺利，您应该看到您的应用程序将此打印到屏幕上：

      _   _          _   _                                           _       _   _ 
     | | | |   ___  | | | |   ___         __      __   ___    _ __  | |   __| | | |
     | |_| |  / _ \ | | | |  / _ \        \ \ /\ / /  / _ \  | '__| | |  / _` | | |
     |  _  | |  __/ | | | | | (_) |  _     \ V  V /  | (_) | | |    | | | (_| | |_|
     |_| |_|  \___| |_| |_|  \___/  ( )     \_/\_/    \___/  |_|    |_|  \__,_| (_)
                                    |/                                             
    

* * *

>   
> 本指南的源代码可以在 [GitHub 上](https://github.com/apple/swift-getting-started-cli)找到