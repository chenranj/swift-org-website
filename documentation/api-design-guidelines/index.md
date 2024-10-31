---
layout: page
title: API 设计指南
official_url: https://swift.org/documentation/api-design-guidelines/
redirect_from: /documentation/api-design-guidelines.html
---
<!-- {% comment %}
页面上 <pre> 元素的宽度经过仔细调整，所以我们
可以去掉滚动条框。
{% endcomment %} -->
<style>
article pre {
    overflow: visible;
}
</style>

<!-- {% comment %}
定义一些变量，帮助我们构建可展开的详细部分
而不需要太多样板代码。我们使用复选框而不是
<details>...</details> 因为它允许我们：

  * 编写 CSS 确保打印时详细信息不会被隐藏。
  * 添加一个按钮，可以一次性展开或折叠所有部分。
{% endcomment %} -->
{% capture expand %}{::nomarkdown}
<input type="checkbox" class="detail">
{:/nomarkdown}{% endcapture %}
{% assign detail = '<div class="more" markdown="1">' %}
{% assign enddetail = '</div>' %}

<div class="info screenonly" markdown="1">
为了便于作为快速参考使用，许多指南的详细信息
可以单独展开。打印此页面时，详细信息永远不会被隐藏。
<input type="button" id="toggle" value="立即展开所有详细信息" onClick="show_or_hide_all()" />
</div>

## 目录
{:.no_toc}

* TOC
{:toc}

## 简介

在编写 Swift 代码时提供清晰、一致的开发者体验很大程度上取决于 API 中出现的名称和习语。
这些设计指南解释了如何确保你的代码感觉像是更大的 Swift 生态系统的一部分。

## 基础原则

* **使用点的清晰性是你最重要的目标。**
  诸如方法和属性之类的实体只声明一次但
  会被*多次使用*。设计 API 时要使这些用法清晰简洁。
  在评估设计时，仅仅阅读声明通常是不够的；
  始终检查用例以确保它在上下文中看起来清晰。
  {:#clarity-at-the-point-of-use}

* **清晰比简短更重要。**虽然 Swift
  代码可以紧凑，但使代码用最少的字符达到最小可能的长度并*不是*目标。
  Swift 代码中的简短性（如果出现）是强大类型系统和
  自然减少样板代码的特性的副作用。
  {:#clarity-over-brevity}

* **为每个声明写文档注释。**
  编写文档时获得的见解可能对你的设计产生深远的影响，
  所以不要推迟这项工作。
  {:#write-doc-comment}

  <div class="warning" markdown="1">
  如果你在用简单的术语描述你的 API 的功能时遇到困难，
  **你可能设计了错误的 API。**
  </div>

  {{expand}}
  {{detail}}
  {% assign ref = 'https://developer.apple.com/library/prerelease/mac/documentation/Xcode/Reference/xcode_markup_formatting_ref/' %}
  {% capture SymbolDoc %}{{ref}}SymbolDocumentation.html#//apple_ref/doc/uid/TP40016497-CH51-{% endcapture %}

  * **使用 Swift 的 [Markdown 方言]({{ref}})。**

  * **以描述所声明实体的摘要开始。**
    通常，一个 API 可以从其声明和摘要中完全理解。

    ~~~ swift
    /// **返回包含相同元素但顺序相反的 `self` 的"视图"。**
    func reversed() -> ReverseCollection<Self>
    ~~~

    {{expand}}
    {{detail}}

    * **专注于摘要**；这是最重要的部分。许多
      优秀的文档注释仅由一个出色的摘要组成。

    * **如果可能，使用单个句子片段**，以句点结尾。
      不要使用完整的句子。

    * **描述函数或方法*做什么*以及它*返回什么***，
      省略空效果和 `Void` 返回：

      ~~~ swift
      /// **在 `self` 的开头插入** `newHead`。
      mutating func prepend(_ newHead: Int)

      /// **返回**包含 `head` 后跟 `self` 元素的 `List`。
      func prepending(_ head: Element) -> List

      /// 如果非空，则**移除并返回** `self` 的第一个元素；
      /// 否则返回 `nil`。
      mutating func popFirst() -> Element?
      ~~~

      注意：在像 `popFirst` 这样的罕见情况，摘要由
      多个用分号分隔的句子片段组成。

    * **描述下标*访问*什么**：

      ~~~ swift
      /// **访问**第 `index` 个元素。
      subscript(index: Int) -> Element { get set }
      ~~~

    * **描述初始化器*创建*什么**：

      ~~~ swift
      /// **创建**包含 `n` 个 `x` 重复的实例。
      init(count n: Int, repeatedElement x: Element)
      ~~~

    * 对于所有其他声明，**描述声明的实体*是*什么**。

      ~~~ swift
      /// **一个集合，支持**在任何位置进行同样高效的插入/删除。
      struct List {

        /// `self` 开头的元素，如果 self 为空则为 `nil`。
        var first: Element?
        ...
      ~~~

    {{enddetail}}

  * **可选地，继续**添加一个或多个段落和项目符号。
    段落之间用空行分隔，使用完整的句子。

    ~~~ swift
    /// 将 `items` 中每个元素的文本表示    <span class="graphic">←</span><span class="commentary">摘要</span>
    /// 写入标准输出。
    ///                                              <span class="graphic">←</span><span class="commentary">空行</span>
    /// 每个项目 `x` 的文本表示 <span class="graphic">←</span><span class="commentary">附加讨论</span>
    /// 由表达式 `String(x)` 生成。
    ///
    /// - **Parameter separator**: 在项目之间    <span class="graphic">⎫</span>
    ///   打印的文本。                             <span class="graphic">⎟</span>
    /// - **Parameter terminator**: 在末尾   <span class="graphic">⎬</span><span class="commentary"><a href="{{SymbolDoc}}SW14">参数部分</a></span>
    ///   打印的文本。                                <span class="graphic">⎟</span>
    ///                                              <span class="graphic">⎭</span>
    /// - **Note**: 要在不添加尾随          <span class="graphic">⎫</span>
    ///   换行符的情况下打印，传递 `terminator: ""`             <span class="graphic">⎟</span>
    ///                                              <span class="graphic">⎬</span><span class="commentary"><a href="{{SymbolDoc}}SW13">符号命令</a></span>
    /// - **SeeAlso**: `CustomDebugStringConvertible`,   <span class="graphic">⎟</span>
    ///   `CustomStringConvertible`, `debugPrint`.   <span class="graphic">⎭</span>
    public func print<Target: OutputStreamType>(
      _ items: Any..., separator: String = " ", terminator: String = "\n")
    ~~~

    {{expand}}
    {{detail}}

    * **使用已认可的
      [符号文档标记]({{SymbolDoc}}SW1)
      元素**在适当时添加超出摘要的信息。

    * **了解并使用带有
      [符号命令语法]({{SymbolDoc}}SW13)的已认可项目符号。** 像 Xcode 这样的流行开发
      工具对以下关键字开头的项目符号给予特殊处理：

      | [Attention]({{ref}}Attention.html) | [Author]({{ref}}Author.html) | [Authors]({{ref}}Authors.html) | [Bug]({{ref}}Bug.html) |
      | [Complexity]({{ref}}Complexity.html) | [Copyright]({{ref}}Copyright.html) | [Date]({{ref}}Date.html) | [Experiment]({{ref}}Experiment.html) |
      | [Important]({{ref}}Important.html) | [Invariant]({{ref}}Invariant.html) | [Note]({{ref}}Note.html) | [Parameter]({{ref}}Parameter.html) |
      | [Parameters]({{ref}}Parameters.html) | [Postcondition]({{ref}}Postcondition.html) | [Precondition]({{ref}}Precondition.html) | [Remark]({{ref}}Remark.html) |
      | [Requires]({{ref}}Requires.html) | [Returns]({{ref}}Returns.html) | [SeeAlso]({{ref}}SeeAlso.html) | [Since]({{ref}}Since.html) |
      | [Throws]({{ref}}Throws.html) | [ToDo]({{ref}}Todo.html) | [Version]({{ref}}Version.html) | [Warning]({{ref}}Warning.html) |

    {{enddetail}}

  {{enddetail}}

## 命名

### 促进�����使用

* **包所要的词以避免使用点处的歧义**，让阅读代码的人
  能够理解名称的含义。
  {:#include-words-to-avoid-ambiguity}

  {{expand}}
  {{detail}}
  例如，考虑一个在集合中移除给定位置元素的方。

  ~~~ swift
  extension List {
    public mutating func remove(at position: Index) -> Element
  }
  employees.remove(at: x)
  ~~~
  {:.good}

  如果我们从方法签名中省略 `at` 这个词，可能会让读者
  误以为该方法会搜索并移除一个等于 `x` 的元素，而不是使用 `x` 来
  指示要移除的元素的位置。

  ~~~ swift
  employees.remove(x) // 不清楚：我们是在移除 x 吗？
  ~~~
  {:.bad}

  {{enddetail}}

* **省略不必要的词。**名称中的每个词都应该在使用点传达
  重要信息。
  {:#omit-needless-words}

  {{expand}}
  {{detail}}
  可能需要更多的词来澄清意图或消除歧义，但那些
  重复读者已经拥有的信息的词应该被省略。特别是，省略那些
  *仅仅重复*类型信息的词。

  ~~~ swift
  public mutating func removeElement(_ member: Element) -> Element?

  allViews.removeElement(cancelButton)
  ~~~
  {:.bad}

  在这种情况下，`Element` 这个词在调用点没有添加任何重要信息。
  这个 API 会更好：

  ~~~ swift
  public mutating func remove(_ member: Element) -> Element?

  allViews.remove(cancelButton) // 更清晰
  ~~~
  {:.good}

  偶尔，重复类型信息是必要的，以避免歧义，但通常
  最好使用描述参数*角色*而不是其类型的词。参见下一项
  了解详情。
  {{enddetail}}

* **根据它们的角色而不是类型约束来命名变量、参数和关联类型。**
  {:#name-according-to-roles}

  {{expand}}
  {{detail}}
  ~~~ swift
  var **string** = "Hello"
  protocol ViewController {
    associatedtype **View**Type : View
  }
  class ProductionLine {
    func restock(from **widgetFactory**: WidgetFactory)
  }
  ~~~
  {:.bad}

  以这种方式重用类型名称未能优化清晰性和
  表达性。相反，努力选择一个表达实体*角色*的名称。

  ~~~ swift
  var **greeting** = "Hello"
  protocol ViewController {
    associatedtype **ContentView** : View
  }
  class ProductionLine {
    func restock(from **supplier**: WidgetFactory)
  }
  ~~~
  {:.good}

  如果一个关联类型与其协议约束如此紧密绑定，以至于
  协议名称*就是*角色，通过在协议名称后附加
  `Protocol` 来避免冲突：

  ~~~ swift
  protocol Sequence {
    associatedtype Iterator : Iterator**Protocol**
  }
  protocol Iterator**Protocol** { ... }
  ~~~
  {{enddetail}}

* **补偿弱类型信息**以澄清参数的角色。
  {:#weak-type-information}

  {{expand}}
  {{detail}}
  特别是当参数类型是 `NSObject`、`Any`、`AnyObject`
  或基本类型如 `Int` 或 `String` 时，类型信息和
  使用点的上下文可能无法完全传达意图。在这个
  例子中，声明可能是清晰的，但使用点是模糊的。

  ~~~ swift
  func add(_ observer: NSObject, for keyPath: String)

  grid.add(self, for: graphics) // 模糊
  ~~~
  {:.bad}

  要恢复清晰性，**在每个弱类型的参数前加上一个描述其
  角色的名词**：

  ~~~ swift
  func add**Observer**(_ observer: NSObject, for**KeyPath** path: String)
  grid.addObserver(self, forKeyPath: graphics) // 清晰
  ~~~
  {:.good}
  {{enddetail}}

### 努力实现流畅的使用

* **优先选择使用点形成语法通顺的英语短语的方法和函数名称。**
  {:#methods-and-functions-read-as-phrases}

  {{expand}}
  {{detail}}
  ~~~swift
  x.insert(y, at: z)          <span class="commentary">"x，在 z 处插入 y"</span>
  x.subviews(havingColor: y)  <span class="commentary">"x 的具有颜色 y 的子视图"</span>
  x.capitalizingNouns()       <span class="commentary">"x，将名词大写"</span>
  ~~~
  {:.good}

  ~~~swift
  x.insert(y, position: z)
  x.subviews(color: y)
  x.nounCapitalize()
  ~~~
  {:.bad}

  当这些参数不是调用含义的���心时，在第一个或第二个参数之后，
  流畅性可以适当降低：

  ~~~swift
  AudioUnit.instantiate(
    with: description,
    **options: [.inProcess], completionHandler: stopProgressBar**)
  ~~~
  {{enddetail}}

* **工厂方法的名称应以 "`make`" 开头，**
  例如 `x.makeIterator()`。
  {:#begin-factory-name-with-make}

* **初始化器和[工厂方法](https://en.wikipedia.org/wiki/Factory_method_pattern)调用**的第一个参数
  不应该与基本名称形成以基本名称开头的短语，
  例如 `x.makeWidget(cogCount: 47)`
  {:#init-factory-phrase-ends-with-basename}

  {{expand}}
  {{detail}}
  例如，这些调用中的第一个参数不作为基本名称所在短语的一部分来阅读：

  ~~~swift
  let foreground = **Color**(red: 32, green: 64, blue: 128)
  let newPart = **factory.makeWidget**(gears: 42, spindles: 14)
  let ref = **Link**(target: destination)
  ~~~
  {:.good}

  在下面的例子中，API 作者试图在第一个参数中创造语法连续性。

  ~~~swift
  let foreground = **Color(havingRGBValuesRed: 32, green: 64, andBlue: 128)**
  let newPart = **factory.makeWidget(havingGearCount: 42, andSpindleCount: 14)**
  let ref = **Link(to: destination)**
  ~~~
  {:.bad}

  在实践中，这条准则与[参数标签](#argument-labels)的准则一起意味着
  除非调用执行[值保留类型转换](#type-conversion)，否则第一个参数将有标签。

  ~~~swift
  let rgbForeground = RGBColor(cmykForeground)
  ~~~
  {{enddetail}}

* **根据它们的副作用来命名函数和方法**
  {:#name-according-to-side-effects}

  * 没有副作用的应该读作名词短语，
    例如 `x.distance(to: y)`、`i.successor()`。

  * 有副作用的应该读作祈使动词短语，
    例如 `print(x)`、`x.sort()`、`x.append(y)`。

  * **命名可变/不可变方法对**要保持一致。
    可变方法通常会有一个不可变的变体，具有相似的语义，
    但返回新值而不是就地更新实例。

    * 当操作**自然地用动词描述**时，使用动词的祈使语气
      来命名可变方法，并应用"ed"或"ing"后缀来命名其不可变对应方法。

      |可变|不可变|
      |-
      |`x.sort()`|`z = x.sorted()`|
      |`x.append(y)`|`z = x.appending(y)`|

      {{expand}}
      {{detail}}

      * 优先使用动词的过去[分词](https://en.wikipedia.org/wiki/Participle)
        （通常添加"ed"）来命名不可变变体：

        ~~~ swift
        /// 原地反转 `self`。
        mutating func reverse()

        /// 返回 `self` 的反转副本。
        func revers**ed**() -> Self
        ...
        x.reverse()
        let y = x.reversed()
        ~~~

      * 当添加"ed"在语法上不合适时（因为动词有直接宾语），
        使用动词的现在[分词](https://en.wikipedia.org/wiki/Participle)，
        通过添加"ing"来命名不可变变体。

        ~~~ swift
        /// 从 `self` 中删除所有换行符
        mutating func stripNewlines()

        /// 返回删除了所有换行符的 `self` 的副本。
        func strip**ping**Newlines() -> String
        ...
        s.stripNewlines()
        let oneLine = t.strippingNewlines()
        ~~~

      {{enddetail}}

    * 当操作**自然地用名词描述**时，使用名词来命名不可变方法，
      并应用"form"前缀来命名其可变对应方法。

      |不可变|可变|
      |-
      |`x = y.union(z)`|`y.formUnion(z)`|
      |`j = c.successor(i)`|`c.formSuccessor(&i)`|

* **布尔方法和属性的使用应该在非可变时读作关于接收者的断言**，例如 `x.isEmpty`、
  `line1.intersects(line2)`。
  {:#boolean-assertions}

* **描述*某物是什么*的协议应该读作名词**（例如 `Collection`）。
  {:#protocols-describing-what-is-should-read-as-nouns}

* **描述*能力*的协议应该使用后缀 `able`、`ible` 或 `ing`**
  （例如 `Equatable`、`ProgressReporting`）。
  {:#protocols-describing-capability-should-use-suffixes}

* 其他**类型、属性、���量和常量的名称应该读作名词**。
  {:#name-of-others-should-read-as-nouns}

### 术语使用得当

**专业术语**
: *名词* - 在特定领域或专业中具有精确、专门含义的词或短语。

* **避免晦涩的术语**，如果一个更常见的词能同样好地传达含义。
  如果"skin"能满足你的目的，就不要说"epidermis"。
  专业术语是一个重要的交流工具，但只应该在捕获否则会丢失的关键含义时使用。
  {:#avoid-obscure-terms}

* **如果你确实使用专业术语，要坚持其既定含义**。
  {:#stick-to-established-meaning}

  {{expand}}
  {{detail}}
  使用技术术语而不是更常见词语的唯一理由是它能*精确地*
  表达否则会模糊或不清楚的内容。因此，API 应该
  严格按照其公认的含义使用该术语。

  * **不要让专家感到惊讶**：任何已经熟悉该术语的人
    如果我们似乎为它发明了新的含义，都会感到惊讶和愤怒。
    {:#do-not-surprise-an-expert}

  * **不要让初学者感到困惑**：任何试图学习该术语的人
    很可能会在网上搜索并找到其传统含义。
    {:#do-not-confuse-a-beginner}
  {{enddetail}}

* **避免缩写。**缩写，尤其是非标准的缩写，
  实际上是专业术语，因为理解取决于
  正确将它们转换为非缩写形式。
  {:#avoid-abbreviations}

  > 使用的任何缩写的预期含义应该
  > 通过网络搜索容易找到。

* **拥抱先例。**不要以牺牲与现有文化的一致性为代价
  来为完全的初学者优化术语。
  {:#embrace-precedent}

  {{expand}}
  {{detail}}
  使用 `Array` 来命名一个连续的数据结构比使用
  像 `List` 这样的简化术语更好，即使初学者
  可能更容易理解 `List` 的含义。数组在现代计算中
  是基础的，所以每个程序员都知道——或者
  很快就会学到——什么是数组。使用大多数程序员
  熟悉的术语，他们的网络搜索和问题就会得到回报。

  在特定的编程*领域*内，比如数学，
  一个广泛有先例的术语如 `sin(x)` 比解释性短语
  如 `verticalPositionOnUnitCircleAtOriginOfEndOfRadiusWithAngle(x)`
  更可取。注意在这种情况下，先例优先于避免缩写的指导原则：
  虽然完整的词是 `sine`，但"sin(*x*)"在程序员中已经
  使用了几十年，在数学家中使用了几个世纪。
  {{enddetail}}

## 约定

### 一般约定

* **记录任何不是 O(1) 的计算属性的复杂度。**人们经常假设属性访问不涉及
  重要的计算，因为他们以存储属性作为
  心智模型。当这个假设可能被违反时，请务必提醒他们。
  {:#document-computed-property-complexity}

* **优先使用方法和属性而不是自由函数。**自由函数
  仅在特殊情况下使用：
  {:#prefer-method-and-properties-to-functions}

  {{expand}}
  {{detail}}

  1. 当没有明显的 `self`：

     ~~~
     min(x, y, z)
     ~~~

  2. 当函数是无约束的泛型：

     ~~~
     print(x)
     ~~~

  3. 当函数语法是既定领域表示法的一部分：

     ~~~
     sin(x)
     ~~~

  {{enddetail}}

* **遵循大小写约定。**类型和协议的名称是
  `UpperCamelCase`。其他所有内容都是 `lowerCamelCase`。
  {:#follow-case-conventions}

  {{expand}}
  {{detail}}

  在美式英语中通常全部大写出现的[首字母缩写词和缩略语](https://en.wikipedia.org/wiki/Acronym)
  应该根据大小写约定统一使用大写或小写：

  ~~~swift
  var **utf8**Bytes: [**UTF8**.CodeUnit]
  var isRepresentableAs**ASCII** = true
  var user**SMTP**Server: Secure**SMTP**Server
  ~~~

  其他缩写词应该被视为普通单词：

  ~~~swift
  var **radar**Detector: **Radar**Scanner
  var enjoys**Scuba**Diving = true
  ~~~
  {{enddetail}}


{% comment %}
* **注意语法歧义**。许多词既可以作为
   名词也可以作为动词，例如"insert"、"record"、"contract"和
   "drink"。考虑这些双重角色可能如何影响你的 API 的清晰性��
  {:#be-conscious-of-grammatical-ambiguity}
{% endcomment %}

* **方法可以共享基本名称**，当它们共享相同的基本
  含义或当它们在不同的领域中操作时。
  {:#similar-methods-can-share-a-base-name}

  {{expand}}
  {{detail}}
  例如，以下做法是鼓励的，因为这些方法本质上做
  相同的事情：

  ~~~ swift
  extension Shape {
    /// 如果 `other` 在 `self` 的区域内，则返回 `true`；
    /// 否则返回 `false`。
    func **contains**(_ other: **Point**) -> Bool { ... }

    /// 如果 `other` 完全在 `self` 的区域内，则返回 `true`；
    /// 否则返回 `false`。
    func **contains**(_ other: **Shape**) -> Bool { ... }

    /// 如果 `other` 在 `self` 的区域内，则返回 `true`；
    /// 否则返回 `false`。
    func **contains**(_ other: **LineSegment**) -> Bool { ... }
  }
  ~~~
  {:.good}

  而且由于几何类型和集合是不同的领域，
  在同一个程序中这样做也是可以的：

  ~~~ swift
  extension Collection where Element : Equatable {
    /// 如果 `self` 包含一个等于
    /// `sought` 的元素，则返回 `true`；否则返回 `false`。
    func **contains**(_ sought: Element) -> Bool { ... }
  }
  ~~~
  {:.good}

  然而，这些 `index` 方法有不同的语义，应该
  被命名为不同的名称：

  ~~~ swift
  extension Database {
    /// 重建数据库的搜索索引
    func **index**() { ... }

    /// 返回给定表中的第 `n` 行。
    func **index**(_ n: Int, inTable: TableID) -> TableRow { ... }
  }
  ~~~
  {:.bad}

  最后，避免"基于返回类型重载"，因为它在存在
  类型推断时会导致歧义。

  ~~~ swift
  extension Box {
    /// 如果有的话，返回存储在 `self` 中的 `Int`，
    /// 否则返回 `nil`。
    func **value**() -> Int? { ... }

    /// 如果有的话，返回存储在 `self` 中的 `String`，
    /// 否则返回 `nil`。
    func **value**() -> String? { ... }
  }
  ~~~
  {:.bad}

  {{enddetail}}

### 参数
{:#parameter-names}

~~~swift
func move(from **start**: Point, to **end**: Point)
~~~

* **选择参数名称以服务于文档**。即使参数名称不会出现在
  函数或方法的使用点，它们也扮演着重要的解释角色。
  {:#choose-parameter-names-to-serve-doc}

  {{expand}}
  {{detail}}
  选择这些名称使文档易于阅读。例如，
  这些名称使文档读起来很自然：

  ~~~swift
  /// 返回包含 `self` 中满足 `**predicate**` 的元素的
  /// `Array`。
  func filter(_ **predicate**: (Element) -> Bool) -> [Generator.Element]

  /// 用 `**newElements**` 替换给定的 `**subRange**` 中的元素。
  mutating func replaceRange(_ **subRange**: Range<Index>, with **newElements**: [E])
  ~~~
  {:.good}

  而这些则使文档变得笨拙和不合语法：

  ~~~swift
  /// 返回包含 `self` 中满足 `**includedInResult**` 的元素的
  /// `Array`。
  func filter(_ **includedInResult**: (Element) -> Bool) -> [Generator.Element]

  /// 用 `**with**` 的内容替换 **`r` 指示的元素范围**。
  mutating func replaceRange(_ **r**: Range<Index>, **with**: [E])
  ~~~
  {:.bad}

  {{enddetail}}

* **当它简化常见用法时，利用默认参数**。任何具有单个常用值的
  参数都是默认值的候选。
  {:#take-advantage-of-defaulted-parameters}

  {{expand}}
  {{detail}}
  默认参数通过隐藏不相关的信息来改进可读性。例如：

  ~~~ swift
  let order = lastName.compare(
    royalFamilyName**, options: [], range: nil, locale: nil**)
  ~~~
  {:.bad}

  可以变成更简单的：

  ~~~ swift
  let order = lastName.**compare(royalFamilyName)**
  ~~~
  {:.good}

  默认参数通常比使用方族更可取，因为它们对任何
  试图理解 API 的人来说都带来更低的认知负担。

  ~~~ swift
  extension String {
    /// *...描述...*
    public func compare(
       _ other: String, options: CompareOptions **= []**,
       range: Range<Index>? **= nil**, locale: Locale? **= nil**
    ) -> Ordering
  }
  ~~~
  {:.good}

  上面的可能不简单，但它比下面的要简单得多：

  ~~~ swift
  extension String {
    /// *...描述 1...*
    public func **compare**(_ other: String) -> Ordering
    /// *...描述 2...*
    public func **compare**(_ other: String, options: CompareOptions) -> Ordering
    /// *...描述 3...*
    public func **compare**(
       _ other: String, options: CompareOptions, range: Range<Index>) -> Ordering
    /// *...描述 4...*
    public func **compare**(
       _ other: String, options: StringCompareOptions,
       range: Range<Index>, locale: Locale) -> Ordering
  }
  ~~~
  {:.bad}

  方法族的每个成员都需要单独记录
  并被用户理解。要在它们之间做出选择，用户需要
  理解所有这些，而且偶尔会出现令人惊讶的关系——例如，
  `foo(bar: nil)` 和 `foo()` 并不总是同义词——这
  使得这成为一个寻找微小差异的繁琐过程，这些差异主要存在于
  几乎相同的文档中。使用带有默认值的单个方法提供了
  一个极大优越的程序员体验。
  {{enddetail}}

* **优先将带有默认值的参数放在参数列表的末尾**。没有默认值的
  参数通常对方法的语义更重要，并提供了一个稳定的
  初始使用模式。
  {:#parameter-with-defaults-towards-the-end}

* **如果你的 API 将在生产环境中运行，优先使用 `#fileID`** 而不是替代方案。
  `#fileID` 节省空间并保护开发者的隐私。在永远不会被最终用户运行的 API 中
  （如测试助手和脚本）使用 `#filePath`，如果完整路径将简化开发工作流程或
  用于文件 I/O。使用 `#file` 以保持与 Swift 5.2 或更早版本的源代码兼容性。

### 参数标签

~~~swift
func move(from **start**: Point, to **end**: Point)
x.move(**from:** x, **to:** y)
~~~

* **当参数不能有效区分时省略所有标签**，
  例如 `min(number1, number2)`、`zip(sequence1, sequence2)`。
  {:#no-labels-for-indistinguishable-arguments}

* **在执行值保留类型转换的初始化器中，省略第一个参数标签**，
  例如 `Int64(someUInt32)`
  {:#type-conversion}

  {{expand}}
  {{detail}}
  第一个参数应该始终是转换的源。

  ~~~
  extension String {
    // 将 `x` 转换为给定基数的文本表示
    init(**_** x: BigInt, radix: Int = 10)   <span class="commentary">← 注意开头的下划线</span>
  }

  text = "The value is: "
  text += **String(veryLargeNumber)**
  text += " and in hexadecimal, it's"
  text += **String(veryLargeNumber, radix: 16)**
  ~~~

  然而，在"缩小"类型转换中，建议使用描述缩小的标签。

  ~~~ swift
  extension UInt32 {
    /// 创建具有指定 `value` 的实例。
    init(**_** value: Int16)            <span class="commentary">← 扩展，所以没有标签</span>
    /// 创建具有 `source` 最低 32 位的实例。
    init(**truncating** source: UInt64)
    /// 创建具有 `valueToApproximate` 最接近的
    /// 可表示近似值的实例。
    init(**saturating** valueToApproximate: UInt64)
  }
  ~~~

  > 值保留类型转换是一个
  > [单态映射](https://en.wikipedia.org/wiki/Monomorphism)，即
  > 源值中的每个差异都会导致
  > 结果值中的差异。
  > 例如，从 `Int8` 到 `Int64` 的转换是值保留的，
  > 因为每个不同的 `Int8` 值都被转换为不同的 `Int64` 值。
  > 相反方向的转换，然而，不可能是值保留的：`Int64` 有更多可能的值
  > 超过了 `Int8` 能表示的范围。
  >
  > 注意：能否检索原始值与转换是否值保留无关。

  {{enddetail}}

* **当第一个参数构成[介词短语](https://en.wikipedia.org/wiki/Adpositional_phrase#Prepositional_phrases)的一部分��，
  给它一个参数标签**。参数标签通常应该从[介词](https://en.wikipedia.org/wiki/Preposition)开始，
  例如 `x.removeBoxes(havingLength: 12)`。
  {:#give-prepositional-phrase-argument-label}

  {{expand}}
  {{detail}}
  当前两个参数代表单个抽象的部分时，会出现一个例外。

  ~~~swift
  a.move(**toX:** b, **y:** c)
  a.fade(**fromRed:** b, **green:** c, **blue:** d)
  ~~~
  {:.bad}

  在这种情况下，在介词*之后*开始参数标签，以保持抽象清晰。

  ~~~swift
  a.moveTo(**x:** b, **y:** c)
  a.fadeFrom(**red:** b, **green:** c, **blue:** d)
  ~~~
  {:.good}
  {{enddetail}}

* **否则，如果第一个参数构成语法短语的一部分，则省略其标签**，
  将任何前置词附加到基本名称上，例如 `x.addSubview(y)`
  {:#omit-first-argument-if-partial-phrase}

  {{expand}}
  {{detail}}
  这条准则意味着如果第一个参数*不*构成语法短语的一部分，它应该有一个标签。

  ~~~swift
  view.dismiss(**animated:** false)
  let text = words.split(**maxSplits:** 12)
  let studentsByName = students.sorted(**isOrderedBefore:** Student.namePrecedes)
  ~~~
  {:.good}

  注意，重要的是短语要传达正确的含义。
  以下内容在语法上是正确的，但会表达错误的意思。

  ~~~swift
  view.dismiss(false)   <span class="commentary">不要取消？取消一个布尔值？</span>
  words.split(12)       <span class="commentary">分割数字 12？</span>
  ~~~
  {:.bad}

  还要注意，具有默认值的参数可以被省略，
  在这种情况下不构成语法短语的一部分，所以它们
  应该始终有标签。
  {{enddetail}}

* **标记所有其他参数。**

## 特殊说明

* **标记元组成员并命名闭包参数**，当它们出现在你的 API 中时。
  {:#label-closure-parameters}

  {{expand}}
  {{detail}}
  这些名称具有
  解释性作用，可以在文档注释中引用，
  并提供对元组成员的表达性访问。

  ~~~ swift
  /// 确保我们至少为 `requestedCapacity` 个元素
  /// 持有唯一引用的存储。
  ///
  /// 如果需要更多存储，调用 `allocate` 时
  /// **`byteCount`** 等于要分配的最大对齐
  /// 字节数。
  ///
  /// - Returns:
  ///   - **reallocated**: 如果分配了新的内存块则为 `true`；
  ///     否则为 `false`。
  ///   - **capacityChanged**: 如果更新了 `capacity` 则为 `true`；
  ///     否则为 `false`。
  mutating func ensureUniqueStorage(
    minimumCapacity requestedCapacity: Int,
    allocate: (_ **byteCount**: Int) -> UnsafePointer&lt;Void&gt;
  ) -> (**reallocated:** Bool, **capacityChanged:** Bool)
  ~~~

  闭包参数的名称应该像顶层函数的[参数名称](#parameter-names)一样选择。
  不支持在调用点出现的闭包参数的标签。
  {{enddetail}}

* **对无约束多态性要特别小心**（例如 `Any`、
  `AnyObject` 和无约束的泛型参数），以避免
  重载集中的歧义。
  {:#unconstrained-polymorphism}

  {{expand}}
  {{detail}}
  例如，考虑这个重载集：

  ~~~ swift
  struct Array<Element> {
    /// 在 `self.endIndex` 处插入 `newElement`。
    public mutating func append(_ newElement: Element)

    /// 按顺序在 `self.endIndex` 处插入 `newElements` 的内容。
    public mutating func append<S: SequenceType>(_ newElements: S)
      where S.Generator.Element == Element
  }
  ~~~
  {:.bad}

  这些方法形成一个语义族，参数类型
  乍看起来是截然不同的。然而，当 `Element`
  是 `Any` 时，单个元素可能与元素序列具有相同的类型。

  ~~~ swift
  var values: [Any] = [1, "a"]
  values.append([2, 3, 4]) // [1, "a", [2, 3, 4]] 还是 [1, "a", 2, 3, 4]？
  ~~~
  {:.bad}

  要消除歧义，更明确地命名第二个重载。

  ~~~ swift
  struct Array {
    /// 在 `self.endIndex` 处插入 `newElement`。
    public mutating func append(_ newElement: Element)

    /// 按顺序在 `self.endIndex` 处插入 `newElements` 的内容。
    public mutating func append<S: SequenceType>(**contentsOf** newElements: S)
      where S.Generator.Element == Element
  }
  ~~~
  {:.good}

  注意新名称如何更好地匹配文档注释。
  在这种情况下，编写文档注释实际上
  引起了 API 作者对这个问题的注意。
  {{enddetail}}


<script>
var elements = document.querySelectorAll("pre code");
for (i in elements) {
    var element = elements[i];
    if (element.textContent) {
        element.innerHTML = element.textContent
            .replace(/\*\*([^\*]+)\*\*/g, "<strong>$1</strong>")
            .replace(/\*([^\*]+)\*/g, "<em>$1</em>");
    }
}
function show_or_hide_all(){
    var checkboxes = document.getElementsByClassName('detail');
    var button = document.getElementById('toggle');

    if(button.value == '立即展开所有详细信息'){
        for (var i in checkboxes){
            checkboxes[i].checked = 'FALSE';
        }
        button.value = '立即折叠所有详细信息'
    }else{
        for (var i in checkboxes){
            checkboxes[i].checked = '';
        }
        button.value = '立即展开所有详细信息';
    }
}
if (location.search.match(/[?&]expand=true\b/)) {
    show_or_hide_all();
}
</script>
