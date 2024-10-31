---
layout: page
title: 社区概述
---

Swift.org 社区有一个独特的目标：打造世界上最好的通用编程语言。我们将在开放环境中共同开发这门语言，欢迎任何希望参与的人贡献。本指南文档描述了 Swift 社区的组织方式，以便我们能够共同为 Swift 添加令人惊叹的新功能，并使其可用于更多平台的更多开发者。

## 交流

Swift 语言在开放环境中开发，所有关于语言或社区流程的技术或管理主题都应该在 Swift 公共论坛中讨论。我们鼓励公开对话，Swift 语言的活跃开发者应该关注相关的论坛分类。

* 论坛分类目录和电子邮件说明在[论坛部分](#forums)。
* 所有 Swift 项目的源代码可以在 GitHub 上的 [github.com/apple][github] 找到。
* Swift 语言的错误追踪系统维护在 [github.com/swiftlang/swift/issues][bugtracker]。

项目空间内的所有交流都应遵守 Swift 项目的[行为准则](/code-of-conduct)。

## 社区结构

以连贯、清晰的视角推进 Swift 编程语言的发展需要强有力的领导。领导层来自社区，并与更广泛的贡献者和用户群体密切合作。社区内的角色包括：

* __[项目负责人](#project-lead)__ 从社区任命技术领导者。Apple Inc. 是项目负责人，通过其代表与社区互动。
* __[核心团队](#core-team)__ 是负责 Swift 项目战略方向和监督的小组。
* __[代码所有者](#code-owners)__ 是负责 Swift 代码库特定领域的个人。
* __[提交者](/contributing/#commit-access)__ 是任何拥有 Swift 代码库提交权限的人。
* __[贡献者](/contributing/#contributing-code)__ 是任何贡献补丁或帮助代码审查的人。
* __指导小组__
   * __[语言](#language-steering-group)__ 是一个专家小组，以连贯的方向推动 Swift 语言向前发展。
   * __[平台](/platform-steering-group)__ 是一个专家小组，使 Swift 语言及其工具能够在新环境中使用。
* __工作组__
   * __[C++ 互操作性](/cxx-interop-workgroup)__ 是一个致力于添加 Swift 和 C++ 之间双向互操作性支持的团队。
   * __[贡献者体验](/contributor-experience-workgroup)__ 是一个支持 Swift 项目贡献者的团队，包括在 Swift 论坛上的贡献。
   * __[文档](/documentation-workgroup)__ 是一个帮助指导 Swift 文档体验的团队。
   * __[服务器端 Swift](/sswg)__ 是一个促进使用 Swift 开发和部署服务器应用程序的团队。
   * __[网站](/website-workgroup/)__ 是一个帮助指导 Swift.org 网站发展的团队。

最重要的是，每个使用 Swift 的人都是我们扩展社区中的重要成员。

#### 项目负责人

[通过论坛联系](https://forums.swift.org/new-message?username=tkremenek)

Apple Inc. 是项目负责人，作为项目的仲裁者。项目负责人任命领导职位的高级人员，这些领导者来自全球 Swift 贡献者社区。社区领导者和代码贡献者共同努力不断改进 Swift，该语言将通过所有参与者的良好工作而进步。

[Ted Kremenek](mailto:kremenek@apple.com) 是 Apple 指定的代表，作为项目负责人的发言人。

#### 核心团队

[通过论坛联系](https://forums.swift.org/new-message?groupname=core-team)

核心团队为 Swift 社区的各个工作组和倡议提供凝聚力，提供支持和战略调整。项目负责人任命核心团队成员，以带来经验、专业知识和领导力的混合，使该团队能够共同作为 Swift 项目及其社区的有效管理者。核心团队成员预计会随时间变化。

当前的核心团队成员是：

{% assign people = site.data.core_team | sort: "name" %}
{% for person in people %}* {{ person.name }}
{% endfor %}

我们感谢以下荣誉退休核心团队成员的服务：

{% assign people = site.data.core_team_emeriti | sort: "name" %}
{% for person in people %}* {{ person.name }}
{% endfor %}

#### 语言指导小组

[通过论坛联系](https://forums.swift.org/new-message?groupname=language-workgroup)

语言指导小组由 Swift 项目负责人和核心团队确定的专家组成，这些专家拥有平衡的观点和专业知识，可以审查、指导并战略性地调整语言的变化。语言指导小组审查并帮助迭代来自社区的[语言演进提案](/contributing/#evolution-process)，作为这些提案的批准者。工作组成员帮助连贯地推动 Swift 语言向前发展，以创建最好的通用编程语言。语言指导小组成员预计会随时间变化。

当前的语言指导小组成员是：

{% assign people = site.data.language_wg | sort: "name" %}
{% for person in people %}* {{ person.name }}
{% endfor %}

#### 代码所有者

[通过论坛联系](https://forums.swift.org/new-message?groupname=code-owners)

代码所有者是被分配到 Swift 项目特定领域的个人，代码质量是他们的主要责任。Swift 总项目由众多子项目组成，包括 Swift 标准库、LLDB 调试器的扩展和 Swift 包管理器等。每个子项目都将被分配一个代码所有者。然后代码所有者负责获取所有贡献的审查、收集社区反馈，并将批准的补丁整合到产品中。

任何人都可以审查代码，我们欢迎所有感兴趣的人进行代码审查。代码审查程序不是由中央全局政策规定的。相反，这个过程由每个代码所有者定义。

任何活跃并表现出价值的社区成员都可以通过在论坛上发帖提出成为代码所有者，或由另一个成员提名。如果其他贡献者同意，项目负责人将进行任命并将新所有者的名字添加到代码所有者文件中。这个职位完全是自愿的，可以随时辞职。

当前代码所有者的列表可以在 Swift 源代码树根目录的 `CODE_OWNERS.txt` 文件中找到。我们还维护一个邮件组，这样你可以[发送电子邮件][email-owners]给所有代码所有者。

对 Swift 的成功来说，可能没有什么比强大、投入的代码所有者更重要了。我们都欠他们尊重、感激之情，以及我们能提供的任何帮助。

每个贡献者负责将其姓名添加到项目根目录的 `CONTRIBUTORS.txt` 文件中并维护联系信息。如果你是在公司的支持下做出贡献，请添加你的公司信息，而不要将自己也列为额外的版权持有人。

{% include_relative _forums.md %}

[homepage]: ./index.html "Swift.org 主页"
[community]: ./community.html  "Swift.org 社区概述"
[contributing_code]: /contributing/#contributing-code  "贡献代码"
[test_guide]: ./test_guide.html "编写优秀 Swift 测试的详细指南"
[blog]: ./blog_home.html  "Swift.org 工程博客"
[faq]: ./faq.html  "Swift.org 常见问题解答"
[downloads]: ./downloads.html  "下载 Swift 工具的最新构建"
[forums]:  ./forums.html
[contributors]: ./CONTRIBUTORS.txt "查看所有 Swift 项目作者"
[owners]: ./CODE_OWNERS.txt "查看所有 Swift 项目代码所有者"
[license]: ./LICENSE.txt "查看 Swift 许可证"

[email-conduct]: mailto:conduct@swift.org  "向行为准则工作组发送电子邮件"
[email-owners]: mailto:code-owners@forums.swift.org  "向代码所有者发送电子邮件"
[email-users]: mailto:swift-users@swift.org  "向其他 Swift 用户发送电子邮件"
[email-devs]: mailto:swift-dev@swift.org  "向开发者讨论列表发送电子邮件"
[email-lead]: mailto:project-lead@swift.org "向负责 Swift.org 的 Apple 领导发送电子邮件"

[github]: https://github.com/apple  "Apple 在 GitHub 上的主页"
[repo]: git+ssh://github.com/apple "托管在 GitHub 上的仓库链接"
[bugtracker]:  http://github.com/swiftlang/swift/issues

[swift-apple]: https://developer.apple.com/swift  "Apple 开发者 Swift 主页"
