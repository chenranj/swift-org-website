---
layout: page
date: 2024-06-04 12:00:00
title: 开始使用静态 Linux SDK
author: [al45tair]
---

众所周知，Swift 可以用于构建 Apple 平台（如 macOS 或 iOS）的软件，但 Swift 也支持其他平台，包括 Linux 和 Windows。

在 Linux 上构建特别有趣，因为从历史上看，用 Swift 编写的 Linux 程序需要确保目标系统上安装了 Swift 运行时及其所有依赖项。此外，为特定发行版或特定发行版的特定主要版本构建的程序不一定能在任何其他发行版上运行，在某些情况下甚至不能在同一发行版的不同主要版本上运行。

Swift 静态 Linux SDK 通过允许你将程序构建为_完全静态链接_的可执行文件来解决这两个问题，没有任何外部依赖（甚至不依赖 C 库），这意味着它将在_任何_ Linux 发行版上运行，因为它只依赖 Linux 系统调用接口。

此外，静态 Linux SDK 可以从 Swift 编译器和包管理器支持的任何平台使用；这意味着你可以在 macOS 上开发和测试你的程序，然后再构建并部署到基于 Linux 的服务器，无论是在本地运行还是在云中的某个地方。

### 静态与动态链接

_链接_是将计算机程序的不同部分连接起来并连接它们之间的任何引用的过程。对于_静态_链接，通常这些部分是_目标文件_，或_静态库_（实际上只是目标文件的集合）。

对于_动态_链接，这些部分是_可执行文件_和_动态库_（又称 dylibs、共享对象或 DLL）。

静态和动态链接之间有两个关键区别：

* 链接发生的时间。静态链接发生在构建程序时；动态链接发生在运行时。

* 静态库（或_归档_）实际上是单个目标文件的集合，而动态库是单体的这一事实。

后者很重要，因为传统上，静态链接器会包含命令行上明确列出的每个对象，但它_只会_从静态库中包含一个对象，如果这样做可以让它解析未解决的符号引用。如果你静态链接了一个你实际上没有使用的库，传统的静态链接器会完全丢弃该库，不会在你的最终二进制文件中包含任何来自它的代码。

在实践中，情况可能更复杂——静态链接器实际上可能基于你的目标文件中的单个_节_或_原子_工作，所以它实际上可能能够丢弃单个函数或数据片段，而不仅仅是整个对象。

### 静态链接的优缺点

静态链接的优点：

* 没有运行时开销。

* 只包含实际需要的库代码。

* 不需要单独安装动态库。

* 运行时没有版本问题。

静态链接的缺点：

* 程序不能共享代码（总体内存使用更高）。

* 无法在不重新构建程序的情况下更新依赖项。

* 可执行文件更大（尽管这可以通过不必安装单独的动态库来抵消）。

特别是在 Linux 上，也可以使用静态链接来完全消除对发行版提供的系统库的依赖，从而生成在任何发行版上都能工作的可执行文件，并且可以通过简单复制来安装。

### 安装 SDK

在开始之前，重要的是要注意：

* 你需要[从 swift.org 安装开源工具链](/install/)。

* 你不能使用 Xcode 提供的工具链来使用 SDK 构建程序。

* 如果你使用 macOS，你还需要确保通过[按照这里的说明](/install/macos/package_installer/)使用这个工具链中的 Swift 编译器。

* 工具链必须与你安装的静态 Linux SDK 版本匹配。静态 Linux SDK 在其文件名中包含相应的 Swift 版本，以帮助识别正确版本的 SDK。

* 当从远程 URL 安装 Swift SDK 时，你必须传递一个 `--checksum` 选项，其中包含 SDK 作者提供的相应校验和。

一旦这些都准备好了，实际安装静态 Linux SDK 很简单；在提示符下输入

```console
$ swift sdk install <URL-或-文件名> [--checksum <归档-URL-的校验和>]
```

提供 SDK 可以找到的 URL（和相应的校验和）或文件名。

例如，假设你已经安装了 `swift-6.0-DEVELOPMENT-SNAPSHOT-2024-07-02-a` 工具链，你需要输入

```console
$ swift sdk install https://download.swift.org/swift-6.0-branch/static-sdk/swift-6.0-DEVELOPMENT-SNAPSHOT-2024-07-02-a/swift-6.0-DEVELOPMENT-SNAPSHOT-2024-07-02-a_static-linux-0.0.1.artifactbundle.tar.gz --checksum 42a361e1a240e97e4bb3a388f2f947409011dcd3d3f20b396c28999e9736df36
```

来安装相应的静态 Linux SDK。

Swift 将下载并在你的系统上安装 SDK。你可以使用

```console
$ swift sdk list
```

列出已安装的 SDK，也可以使用

```console
$ swift sdk remove <SDK-名称>
```

来移除它们。

### 你的第一个静态链接的 Linux 程序

首先，创建一个目录来存放你的代码：

```console
$ mkdir hello
$ cd hello
```

接下来，让 Swift 为你创建一个新的程序包：

```console
$ swift package init --type executable
```

你可以在本地构建和运行这个程序：

```console
$ swift build
Building for debugging...
[8/8] Applying hello
Build complete! (15.29s)
$ .build/debug/hello
Hello, world!
```

但有了静态 Linux SDK，你也可以为 x86-64 和 ARM64 机器构建 Linux 二进制文件：

```console
$ swift build --swift-sdk x86_64-swift-linux-musl
Building for debugging...
[8/8] Linking hello
Build complete! (2.04s)
$ file .build/x86_64-swift-linux-musl/debug/hello
.build/x86_64-swift-linux-musl/debug/hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, with debug_info, not stripped
```

```console
$ swift build --swift-sdk aarch64-swift-linux-musl
Building for debugging...
[8/8] Linking hello
Build complete! (2.00s)
$ file .build/aarch64-swift-linux-musl/debug/hello
.build/aarch64-swift-linux-musl/debug/hello: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, with debug_info, not stripped
```

这些可以复制到适当的基于 Linux 的系统并执行：

```console
$ scp .build/x86_64-swift-linux-musl/debug/hello linux:~/hello
$ ssh linux ~/hello
Hello, world!
```

### 包依赖项呢？

使用 Foundation 或 Swift NIO 的 Swift 包应该可以直接工作。但是，如果你尝试使用使用 C 库的包，你可能需要做一些工作。这样的包通常包含类似以下代码的文件：

```swift
#if os(macOS) || os(iOS)
import Darwin
#elseif os(Linux)
import Glibc
#elseif os(Windows)
import ucrt
#else
#error(Unknown platform)
#endif
```

静态 Linux SDK 不使用 Glibc；相反，它建立在一个名为 [Musl](https://musl-libc.org) 的替代 C 库之上。我们选择这种方法有两个原因：

1. Musl 对静态链接有出色的支持。

2. Musl 采用许可宽松的许可证，这使得分发静态链接它的可执行文件变得容易。

如果你使用这样的依赖项，你将需要调整它以导入 `Musl` 模块而不是 `Glibc` 模块：

```swift
#if os(macOS) || os(iOS)
import Darwin
#elseif canImport(Glibc)
import Glibc
#elseif canImport(Musl)
import Musl
#elseif os(Windows)
import ucrt
#else
#error(Unknown platform)
#endif
```

偶尔可能会出现 C 库类型在 Musl 和 Glibc 之间导入方式的差异；如果有人添加了空值性注释，或者当指针类型使用前向声明的 `struct` 但从未提供实际定义时，有时会发生这种情况。通常问题很明显——函数参数或结果在一种情况下是 `Optional` 而在另一种情况下不是，或者指针类型被导入为 `OpaquePointer` 而不是 `UnsafePointer<FOO>`。

如果你发现自己需要进行这些类型的调整，你可以通过执行

```console
$ swift package edit SomePackage
```

使包依赖项可编辑，然后编辑出现在你的程序源目录中的 `Packages` 目录中的文件。你可能希望考虑向上游提交带有任何修复的 PR。
