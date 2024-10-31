## 好的第一个问题

好的第一个问题是针对新加入 Swift 项目的贡献者，甚至是新接触 Swift 编译器等子项目的模式和概念的贡献者的错误、想法和任务。
好的第一个问题带有相应的标签，最容易通过访问 `github.com/apple/<repository>/contribute` 找到，例如
[github.com/apple/swiftlang/contribute](https://github.com/swiftlang/swift/contribute)
用于主要的 Swift 仓库。
它们应该是低优先级的，范围适中，不需要过多的重构、研究或调试 — 相反，它们应该鼓励
新人在 Swift 的某个部分试水，了解更多，并
做出真正的贡献。

任何具有[提交访问权限](#commit-access)并对特定领域有见解的人
都欢迎并鼓励确定或思考好的第一个问题。

{% comment %}
    // TODO: 这是我想迁移到 Jira 中某种"starter"标签后面的内容。目前：

* Swift 中间语言（SIL）往返：确保 SIL 解析器可以解析 SIL 打印器打印的内容。这是一个很好的项目，可以了解 SIL 及其使用方式，使其可往返对任何在 Swift 编译器上工作的人都有巨大的好处。
* 警告控制：[Clang](http://clang.llvm.org) 有一个很好的方案，可以将警告放入特定的警告组中，允许人们（从命令行）控制编译器发出哪些警告或将哪些警告视为错误。Swift 需要这个！
{% endcomment %}
