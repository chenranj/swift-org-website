## LLVM 和 Swift

Swift 建立在 [LLVM 编译器基础设施][LLVM]之上。Swift 使用 LLVM Core 进行代码生成和优化（以及其他事项），使用 [Clang][Clang] 实现与 C 和 Objective-C 的互操作性，使用 [LLDB][LLDB] 进行调试和 REPL。

Apple 在 GitHub 上维护着 [LLVM Core](https://github.com/llvm/llvm-project) 源代码仓库的一个分支，名为 [llvm-project][llvm-project]。
这个仓库跟踪上游 LLVM 的开发，并包含
Swift 的额外更改。上游 LLVM 仓库
经常被合并到 Swift 特定的仓库中。尽可能
将上游 LLVM 和 Apple 分支之间的差异最小化，
仅限于 Swift 特别需要的更改。

### LLVM 更改去向何处？

Swift 遵循在最上游可行的仓库中进行更改的政策。
涉及 Apple 版本 LLVM 项目的 Swift 贡献应该直接进入
上游 LLVM 仓库，除非它们是 Swift 特有的。例如，
对 LLDB 中 Swift 类型的数据格式化程序的改进
属于 Apple LLVM 项目仓库，而对 LLVM
优化器的错误修复——即使只在对 Swift 生成的 LLVM IR
操作时才观察到——属于上游 LLVM。

对上游 LLVM 仓库的提交会自动合并到
相应的上游分支中的相应 Swift
仓库（[llvm-project][llvm-project] 中的 `next`）。

### Swift 和 LLVM 开发者政策
对 Swift 的 LLVM 或 Clang 克隆的贡献受 [LLVM
开发者政策](http://llvm.org/docs/DeveloperPolicy.html)管理，
并应遵循适当的
[许可](http://llvm.org/docs/DeveloperPolicy.html#copyright-license-and-patents)
和[编码标准](http://llvm.org/docs/CodingStandards.html)。
LLVM 代码的问题使用 [LLVM 错误数据库][llvm-bugs]追踪。
对于 LLDB，对带有 llvm.org 注释头的文件的更改必须进入
[llvm.org 的上游 LLDB](https://github.com/llvm/llvm-project/tree/main/lldb)
并遵守 [LLVM 开发者政策](http://llvm.org/docs/DeveloperPolicy.html)
和 [LLDB 编码约定](https://llvm.org/docs/CodingStandards.html)。
对 LLDB 的 Swift 特定部分（即那些带有 Swift.org
注释头的部分）的贡献使用 [Swift 许可证](/community/#license)，但仍然遵循
LLDB 编码约定。

[llvm-project]: https://github.com/apple/llvm-project
[LLVM]: http://llvm.org
[llvm-bugs]: https://github.com/llvm/llvm-project/issues "LLVM 错误追踪器"
[Clang]: http://clang.llvm.org
[LLDB]: http://lldb.llvm.org
