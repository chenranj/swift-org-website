---
redirect_from: "/continuous-integration/"
layout: page
title: Swift 持续集成
---

Swift 项目遵循[增量开发模型](../contributing/#contributing_code)，并在合并之前利用持续集成 (CI) 测试对拉取请求中的更改进行测试，作为维护项目稳定性的核心工具。该系统生成发布在 swift.org 上的快照构建，并针对活动分支运行测试。它也被用作审核过程的一部分，在提交拉取请求之前对其运行测试。

## 配置

我们的[持续集成系统](https://ci.swift.org)由 [Jenkins](https://jenkins.io) 提供支持，目前支持在 macOS、Ubuntu 18.04、Ubuntu 20.04、Ubuntu 22.04、CentOS 7 和 Amazon Linux 2 上进行构建和测试。还支持在 iOS、tvOS 和 watchOS 模拟器上进行测试。

### 任务组织

持续集成任务在 [CI 系统](https://ci.swift.org)中按以下类别组织：

* Development - 所有配置为使用主分支构建的任务
* Swift 5.6 - 所有配置为使用 release/5.6 构建的任务
* Packages - 为主分支和 release/5.6 分支创建工具链的任务
* Pull Request - 在从 GitHub 合并到主分支之前验证拉取请求的任务。

## 使用

您可以通过以下几种方式与 swift.org CI 系统交互：

* 集成任务状态 - 您可以在 [https://ci.swift.org](https://ci.swift.org) 查看所有集成任务的构建和测试状态。
* 拉取请求上的测试 - 通过拉取请求进行更改时，您的更改将在集成之前进行测试，并将结果发布回拉取请求中。

更多使用文档可以在[这里](https://github.com/swiftlang/swift/blob/main/docs/ContinuousIntegration.md)找到。

### 拉取请求测试

当在拉取请求上审核更改时，Swift 团队的成员将触发 CI 系统进行测试。可以触发在 macOS、Linux 或两个平台上运行测试。

![拉取请求 CI 触发器](../continuous-integration/images/ci_pull_command.png)

然后，测试状态将与拉取请求一起发布，显示测试正在进行中。您可以单击"详细信息"链接直接转到正在进行的测试的状态页面。

![CI 进度](../continuous-integration/images/ci_pending.png)

当测试完成时，该结果也会在拉取请求中更新

![CI 通过](../continuous-integration/images/ci_pass.png)

如果在测试过程中发现问题，您将获得指向失败详细信息的链接。

![CI 通过](../continuous-integration/images/ci_failure.png)

在将更改提交到开发分支之前，预期更改应符合 Swift 项目的[质量标准](../contributing/#quality)，您有责任修复更改发现的问题。如果您的更改破坏了开发或发布分支上的构建或测试，您将收到电子邮件通知。


## 社区参与

Swift 项目欢迎社区提出增加对其他配置的支持的建议。

### Swift 社区托管的持续集成

社区成员可以自愿在 [Swift 社区托管的持续集成](https://ci-external.swift.org) 上为其他平台托管节点，并负责维护主机系统。可以通过在以下位置创建拉取请求来启动新节点：[Swift 社区托管 CI 存储库](https://github.com/swiftlang/swift-community-hosted-continuous-integration)。有关该过程的更多信息记录在 [README.md](https://github.com/swiftlang/swift-community-hosted-continuous-integration/blob/main/README.md) 中。

Swift 社区托管的 CI 允许根据具体情况将不受支持的平台移至受支持的平台。根据提供的节点数量，@swift-ci 拉取请求测试也可以与社区托管的 CI 集成。
