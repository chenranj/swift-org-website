---
layout: page-wide
title: 欢迎
hideTitle: true
atom: true
---

<div class="callout" markdown="1">
  <h1 class="preamble">Swift 是一款<strong>通用</strong>编程语言。<br/>新手<strong>易学</strong>，专家<strong>称心</strong><span>Swift <strong>快速</strong>、<strong>现代</strong>、<strong>安全</strong>，<br/>写起代码<strong>乐无边</strong>。</span></h1>

{% for snippet in site.data.featured_snippets %}
```swift
{{ snippet -}}
```
{: class="featured-snippet {% if forloop.first %}visible{% endif %}" }
{% endfor %}
</div>

<div class="banner primary">
  <p>使用 <a href="https://www.swift.org/migration/documentation/migrationguide/">官方迁移指南</a> 为 Swift 6 语言做好准备</p>
</div>

<div class="link-grid">
  <ul>
    <li>
      <a href="/install">
        <div class="flex-container">
          <div class="latest-release-container">
          <span>
            {{ site.data.builds.swift_releases.last.name }}
          </span>
          </div>
          最新发布
        </div>
      </a>
    </li>

    <li>
      <a href="/getting-started">
        <img src="/assets/images/landing-page/signs.svg" />
        开始使用
      </a>
    </li>

    <li>
      <a href="/documentation">
        <img src="/assets/images/landing-page/book.svg" />
        阅读文档
      </a>
    </li>

    <li>
      <a href="/packages">
        <img src="/assets/images/landing-page/box.svg" />
        探索软件包
      </a>
    </li>
  </ul>
</div>

## 使用案例

<ul class="use-cases">
  <li>
    <h3>Apple 平台</h3>
    <p>
      Swift 是一种强大而直观的编程语言，在 iOS、macOS 和其他苹果平台上运行时经过优化。
      <br><br>
      苹果公司提供了各种各样的框架和应用程序接口，使为这些平台开发的应用程序变得独特而有趣。
    </p>
    <a href="https://developer.apple.com/swift/resources/" class="cta-secondary">了解更多</a>
  </li>
  <li>
    <h3>跨平台命令行</h3>
    <p>
      编写 Swift 既互动又有趣，语法简洁而富有表现力。
      Swift 代码设计安全，生成的软件运行速度快如闪电。
      <br><br>
      SwiftArgumentParser 和 Swift 日益壮大的软件包生态系统让开发跨平台命令行工具变得轻而易举。
    </p>

    <a href="/getting-started/cli-swiftpm" class="cta-secondary">了解更多</a>
  </li>
  <li>
    <h3>服务器和网络</h3>
    <p>
      Swift 占用内存小、启动时间快、性能稳定，是服务器和其他网络应用程序的最佳选择。
      <br><br>
      SwiftNIO 和 Swift 的动态服务器生态系统为开发网络应用程序带来了乐趣。
    </p>

    <a href="/documentation/server" class="cta-secondary">了解更多</a>
  </li>
</ul>

## 参与其中

欢迎大家为 Swift 做出贡献。贡献不仅仅意味着编写代码或提交拉取请求，您还可以通过多种不同的方式参与其中，包括在论坛上回答问题、报告或分流错误，以及参与 Swift 的演进过程。

无论您想以何种方式参与其中，我们都要求您首先阅读 [社区概述](/community/)，了解对项目参与者的期望。如果您要贡献代码，还应了解如何从版本库中构建和运行 Swift，详见 [源代码](/documentation/source-code/)。

<ul class="getting-involved">
  <li>
    <h3>设计</h3>
    <p>
      参与 <em>Swift 演进过程</em>，帮助塑造 Swift 的未来。
    </p>
    <a href="/contributing/#swift-evolution" class="cta-secondary">了解更多</a>
  </li>
  <li>
    <h3>代码</h3>
    <p>
      为 Swift 编译器、标准库和项目的其他核心组件做出贡献。
    </p>
    <a href="/contributing/#contributing-code" class="cta-secondary">了解更多</a>
  </li>
  <li>
    <h3>排错</h3>
    <p>
      通过报告和分流错误，帮助提高 Swift 的质量。
    </p>
    <a href="/contributing/#triaging-bugs" class="cta-secondary">了解更多</a>
  </li>
</ul>

## 紧跟动态

了解 Swfit 中文社区最新动态。

<div class="links links-list-nostyle" markdown="1">
  - [访问 Swift.org 博客](/blog/)
  - [访问 SwiftGG 中文论坛](https://forums.swift.org.gg)
  - [访问 Swift 论坛（英文）](https://forums.swift.org)
  - [在 Github 上关注 SwiftGG](https://github.com/swiftggteam){:target="_blank" class="link-external"}
  - [在 X 上关注 Swift（以前的推特）](https://x.com/swiftlang){:target="_blank" class="link-external"}
</div>

<script>
  var featuredSnippets = document.querySelectorAll('.featured-snippet');
  var visibleSnippet = document.querySelector('.featured-snippet.visible');
  var randomIndex = Math.floor(Math.random() * featuredSnippets.length);

  visibleSnippet?.classList.remove('visible');
  featuredSnippets[randomIndex]?.classList.add('visible');
</script>
