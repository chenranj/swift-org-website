## 报告错误

报告错误是任何人帮助改进 Swift 的好方法。开源 Swift 项目使用 GitHub Issues 来追踪错误。

<div class="info" markdown="1">
如果一个错误只能在 Xcode 项目或 playground 中重现，
或者如果这个错误与 Apple NDA 有关，
请改为向 Apple 的[错误报告系统][apple-bugtracker]提交报告。
</div>

在开启问题时，请包含以下内容：

- **对问题的简明描述。**
  如果问题是崩溃，请包含堆栈跟踪。否则，描述你期望看到的行为，以及你实际观察到的行为。

- **一个可重现的测试用例。**
  仔细检查你的测试用例是否能重现该问题。一个相对较小的样本（大约在 50 行代码以内）最好直接粘贴到描述中；较大的样本可以作为附件上传。考虑将样本减少到尽可能少的代码——更小的测试用例更容易推理，对贡献者来说也更有吸引力。

- **重现问题的环境描述。**
  包括 Swift 编译器的版本、部署目标（如果明确设置）和你的平台信息。

因为 Swift 正在非常活跃的开发中，我们收到了很多错误报告。在开启新问题之前，请花点时间浏览我们的[现有问题](https://github.com/swiftlang/swift/issues)以减少报告重复的机会。

<div class="warning" markdown="1">
在提交请求新语言功能的问题之前，请参阅[Swift 演进过程部分](#participating-in-the-swift-evolution-process)。
</div>

[bugtracker]: http://github.com/swiftlang/swift/issues
[apple-bugtracker]: https://bugreport.apple.com
[evolution-repo]: https://github.com/swiftlang/swift-evolution "链接到 GitHub 上的 Swift 演进仓库"
