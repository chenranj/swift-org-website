---
layout: page
date: 2024-06-04 15:13:07
title: 为 Swift 开发配置 Neovim
author: [etcwilde]
---

[Neovim](https://neovim.io) 是一个流行的终端文本编辑器 _Vim_ 的现代重新实现。
Neovim 除了 _Vim_ 对原始 _Vi_ 编辑器的改进外，还添加了异步操作和强大的 Lua 绑定等新特性，
提供了流畅的编辑体验。

本文将指导你为 Swift 开发配置 Neovim，为各种插件提供配置，以构建一个可用的 Swift 编辑环境。
配置文件将逐步构建，文章末尾包含这些文件的完整组装版本。
这不是一个如何使用 Neovim 的教程，假设你已经熟悉像 _Neovim_、_Vim_ 或 _Vi_ 这样的模态文本编辑器。
我们还假设你已经在计算机上安装了 Swift 工具链。如果没有，请参阅
[Swift 安装说明](https://www.swift.org/install)。

虽然本文引用了 Ubuntu 22.04，但配置本身适用于任何安装了较新版本 Neovim 和 Swift 工具链的
操作系统。

基本设置和配置包括：

1. 安装 Neovim
2. 安装 `lazy.nvim` 来管理插件
3. 配置 SourceKit-LSP 服务器
4. 使用 _nvim-cmp_ 设置基于语言服务器的代码补全
5. 使用 _LuaSnip_ 设置代码片段

以下部分将帮助指导你完成设置：

- [前提条件](#prerequisites)
- [包管理](#packaging-with-lazy)
- [语言服务器支持](#language-server-support)
    - [文件更新](#file-updating)
- [代码补全](#code-completion)
- [代码片段](#Snippets)
- [完整配置文件](#files)

> 提示：如果你已经安装了 Neovim、Swift 和包管理器，可以直接跳到[语言服务器支持](#language-server-support)部分。

> 注意：如果你跳过了[前提条件](#prerequisites)部分，请确保你的
Neovim 版本是 v0.9.4 或更高，否则可能在使用一些语言服务器协议(LSP) Lua API 时遇到问题。

## 前提条件

首先，你需要安装 Neovim。Neovim 暴露的 Lua API 正在快速发展。我们想要利用语言服务器协议(LSP)
集成支持的最新改进，所以我们需要一个相当新的 Neovim 版本。

我使用的是 x86_64 架构的 Ubuntu 22.04。不幸的是，Ubuntu 22.04 的 `apt` 仓库中的
Neovim 版本太旧，不支持我们将要使用的许多 API。

对于这次安装，我使用 `snap` 来安装 Neovim v0.9.4。
Ubuntu 24.04 有足够新的 Neovim 版本，所以普通的
`apt install neovim` 命令就可以工作。
关于在其他操作系统和 Linux 发行版上安装 Neovim，
请参阅 [Neovim 安装页面](https://github.com/neovim/neovim/blob/master/INSTALL.md)。

```console
 $  sudo snap install nvim --classic
 $  nvim --version
NVIM v0.9.4
Build type: RelWithDebInfo
LuaJIT 2.1.1692716794
Compilation: /usr/bin/cc -O2 -g -Og -g -Wall -Wextra -pedantic -Wno-unused-pa...

   system vimrc file: "$VIM/sysinit.vim"
  fall-back for $VIM: "/usr/share/nvim"

Run :checkhealth for more info
```

## 入门

我们的路径中已经有了可用的 Neovim 和 Swift。虽然我们可以从 `vimrc` 文件开始，但 Neovim 正在
从使用 vimscript 过渡到 Lua。Lua 更容易找到文档因为它是一个真正的编程语言，
运行速度更快，并且将配置从主运行循环中抽离出来，使编辑器保持流畅。
你仍然可以使用带有 vimscript 的 `vimrc`，但我们将使用 Lua。

主要的 Neovim 配置文件放在 `~/.config/nvim` 中。其他 Lua 文件
放在 `~/.config/nvim/lua` 中。现在创建一个 `init.lua`：

```console
 $  mkdir -p ~/.config/nvim/lua && cd ~/.config/nvim
 $  nvim init.lua
```

> 注意：下面的示例包含插件的 GitHub 链接，以帮助你快速访问文档。你也可以直接探索插件本身。

## 使用 _lazy.nvim_ 进行包管理

虽然可以手动设置所有内容，但使用包管理器可以帮助
保持包的更新，并确保当你将配置复制到新计算机时所有内容都正确安装。Neovim 也有一个
内置插件管理器，但我发 [_lazy.nvim_](https://github.com/folke/lazy.nvim) 工作得很好。

我们将从一个小的引导脚本开始，如果 _lazy.nvim_ 还没有安装，就安装它，
将它添加到运行时路径中，最后配置我们的包。

在你的 `init.lua` 顶部写入：
```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
    vim.fn.system({
        "git",
        "clone",
        "--filter=blob:none",
        "https://github.com/folke/lazy.nvim.git",
        "--branch=stable",
        lazypath
    })
end
vim.opt.rtp:prepend(lazypath)
```

这段代码如果 _lazy.nvim_ 不存在就克隆它，然后将它添加到运行时路径中。现在我们初始化 _lazy.nvim_ 并告诉它在哪里查找插件
规格。

```lua
require("lazy").setup("plugins")
```

这配置 _lazy.nvim_ 在我们的 `lua/` 目录下的 `plugins/` 目录中查找每个插件。我们还需要一个地方来放置我们自己的非插件
相关配置，所以我们将它放在 `config/` 中。现在创建这些目录。

```console
 $  mkdir lua/plugins lua/config
```

有关配置 _lazy.nvim_ 的详细信息，请参阅 [lazy.nvim 配置](https://github.com/folke/lazy.nvim?tab=readme-ov-file#%EF%B8%8F-configuration)。

![_lazy.nvim_ package manger](/assets/images/zero-to-swift-nvim/Lazy.png)

请注意，您的配置不会与此完全相同。
我们只安装了_lazy.nvim_，因此它是目前唯一在配置中列出的插件。
配置中列出的唯一插件。
这看起来并不令人兴奋，所以我添加了一些额外的插件，使其看起来更吸引人。
让它看起来更吸引人。

检查是否正常工作
- 启动 Neovim。

   你首先会看到一个错误，提示没有为模块插件的规格。这只是表示没有任何插件。

 - 按 Enter 键并输入 `:Lazy`。

   _lazy.nvim_会列出已安装的插件。现在应该只有一个：
   “lazy.nvim"。这是_lazy.nvim_在跟踪和更新自己。

 - 我们可以通过_lazy.nvim_菜单管理我们的插件。
    - 按 `I` 键将安装新插件。
    - 按下 `U` 会更新已安装的插件。
    - 按下 `X` 会删除_lazy.nvim_安装的任何插件，但在配置中不再被跟踪。
      在配置中不再被跟踪的插件。

## 语言服务器支持

语言服务器会响应编辑器的请求，提供特定语言的支持。
Neovim 内置了对语言服务器协议（LSP）的支持，因此你不需要为 LSP
但手动为每个 LSP 服务器添加配置是一项繁重的工作。
但手动为每个 LSP 服务器添加配置是一项繁重的工作。Neovim 有一个配置 LSP 服务器的软件包、
[nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)。

请在 `lua/plugins/lsp.lua` 下创建一个新文件。在该文件中，我们将首先
添加以下代码段。

```lua
return {
    {
        "neovim/nvim-lspconfig",
        config = function()
            local lspconfig = require('lspconfig')
            lspconfig.sourcekit.setup {}
        end,
    }
}
```

While this gives us LSP support through SourceKit-LSP, there are no keybindings,
so it's not very practical. Let's hook those up now.

We'll set up an auto command that fires when an LSP server attaches in the `config`
function under where we set up the `sourcekit` server. The keybindings are
applied to all LSP servers so you end up with a consistent experience across
languages.

```lua
config = function()
    local lspconfig = require('lspconfig')
    lspconfig.sourcekit.setup {}

    vim.api.nvim_create_autocmd('LspAttach', {
        desc = 'LSP Actions',
        callback = function(args)
            vim.keymap.set('n', 'K', vim.lsp.buf.hover, {noremap = true, silent = true})
            vim.keymap.set('n', 'gd', vim.lsp.buf.definition, {noremap = true, silent = true})
        end,
    })
end,
```

![LSP 驱动的实时错误息](/assets/images/zero-to-swift-nvim/LSP-Error.png)

我创建了一个小的 Swift 包示例，用于异步计算[斐波那契数列](https://oeis.org/A000045)。
在 `fibonacci` 函数的任何引用上按 `shift` + `k` 
会显示该函数的文档，以及函数签名。
LSP 集成还显示我们的代码中有一个错误。

### 文件更新

SourceKit-LSP 越来越依赖编辑器通知服务器某些文件发生变化。这种需求通过 _动态注册_ 来传达。
你不需要理解这是什么意思，但 Neovim 不实现动态注册。当你更新包清单时，
或者向 `compile_commands.json` 文件添加新文件时，你会注意到这一点，LSP 在不重启 Neovim 的情况下无法工作。

相反，我们知道 SourceKit-LSP 需要这个功能，所以我们将静态启用它。我们将更新我们的 `sourcekit` 设置配置，
手动设置 `didChangeWatchedFiles` 功能。

```lua
lspconfig.sourcekit.setup {
    capabilities = {
        workspace = {
            didChangeWatchedFiles = {
                dynamicRegistration = true,
            },
        },
    },
}
```

如果你有兴趣了解更多关于这个问题的信息，以下问题中的讨论更详细地描述了这个问题：
 - [LSP: 实现动态注册](https://github.com/neovim/neovim/issues/13634)
 - [向服务器功能响应添加 documentFormattingProvider](https://github.com/microsoft/vscode-eslint/pull/1307)

## 代码补全

![LSP 驱动的自动补全完成 Foundation 模块](/assets/images/zero-to-swift-nvim/LSP-Autocomplete.png)

我们将使用 [_nvim-cmp_](https://github.com/hrsh7th/nvim-cmp) 作为代码补全机制。
我们将首先告诉 _lazy.nvim_ 下载该包，并在我们进入插入模式时懒加载它，因为如果你不编辑文件就不需要代码补全。

```lua
-- lua/plugins/codecompletion.lua
return {
    {
        "hrsh7th/nvim-cmp",
        version = false,
        event = "InsertEnter",
    },
}
```

接下来，我们将配置一些补全源以提供代码补全结果。
_nvim-cmp_ 不附带补全源，这些是额外的插件。
对于此配置，我希望基于 LSP、文件路径补全和
当前缓冲区中的文本提供结果。更多信息请参阅 _nvim-cmp_ Wiki 上的[源列表](https://github.com/hrsh7th/nvim-cmp/wiki/List-of-sources)。

首先，我们将告诉 _lazy.nvim_ 关于新插件，并且 _nvim-cmp_ 依赖于它们。
这确保了 _lazy.nvim_ 在加载 _nvim-cmp_ 时会初始化它们。

```lua
-- lua/plugins/codecompletion.lua
return {
    {
        "hrsh7th/nvim-cmp",
        version = false,
        event = "InsertEnter",
        dependencies = {
            "hrsh7th/cmp-nvim-lsp",
            "hrsh7th/cmp-path",
            "hrsh7th/cmp-buffer",
        },
    },
    { "hrsh7th/cmp-nvim-lsp", lazy = true },
    { "hrsh7th/cmp-path", lazy = true },
    { "hrsh7th/cmp-buffer", lazy = true },
}
```

现在我们需要配置 _nvim-cmp_ 以利用代码补全源。
与许多其他插件不同，_nvim-cmp_ 隐藏了许多内部工作，因此
配置它与其他插件略有不同。特别是，你会注意到设置键绑定的不同之处。我们首先在其自身的配置函数中要求
模块，并将显式调用设置函数。

```lua
{
    "hrsh7th/nvim-cmp",
    version = false,
    event = "InsertEnter",
    dependencies = {
        "hrsh7th/cmp-nvim-lsp",
        "hrsh7th/cmp-path",
        "hrsh7th/cmp-buffer",
    },
    config = function()
        local cmp = require('cmp')
        local opts = {
            -- 从哪里获取补全结果
            sources = cmp.config.sources {
                { name = "nvim_lsp" },
                { name = "buffer"},
                { name = "path" },
            },
            -- 使 'enter' 键选择补全
            mapping = cmp.mapping.preset.insert({
                ["<CR>"] = cmp.mapping.confirm({ select = true })
            }),
        }
        cmp.setup(opts)
    end,
},
```

使用 `tab` 键选择补全是一个相当流行的选项，所以我们现在就设置它。

```lua
mapping = cmp.mapping.preset.insert({
    ["<CR>"] = cmp.mapping.confirm({ select = true }),
    ["<tab>"] = cmp.mapping(function(original)
        if cmp.visible() then
            cmp.select_next_item() -- 如果正在补全则运行补全选择
        else
            original()      -- 如果没有补全则运行原始行为
        end
    end, {"i", "s"}),
    ["<S-tab>"] = cmp.mapping(function(original)
        if cmp.visible() then
            cmp.select_prev_item()
        else
            original()
        end
    end, {"i", "s"}),
}),
```

按下 `tab` 键时，如果补全菜单可见，将选择下一个补全项，而 `shift` + `tab` 将选择上一个项。tab 行为
如果菜单不可见，则回退到任何预定义的行为。

## 代码片段

代码片段是通过将短文本片段扩展为任何你喜欢的内容来提高工作流程的好方法。现在让我们连接这些。我们将使用 [_LuaSnip_](https://github.com/L3MON4D3/LuaSnip) 作为我们的
代码片段插件。

在你的插件目录中创建一个新文件，用于配置代码片段插件。

```lua
-- lua/plugins/snippets.lua
return {
    {
        'L3MON4D3/LuaSnip',
        config = function(opts)
            require('luasnip').setup(opts)
            require('luasnip.loaders.from_snipmate').load({ paths = "./snippets" })
        end,
    },
}
```

现在我们将代码片段扩展连接到 _nvim-cmp_。首先，我们将
_LuaSnip_ 作为 _nvim-cmp_ 的依赖项添加，以确保它在
_nvim-cmp_ 之前加载。然后我们将其连接到 tab 键扩展行为。

```lua
{
    "hrsh7th/nvim-cmp",
    version = false,
    event = "InsertEnter",
    dependencies = {
        "hrsh7th/cmp-nvim-lsp",
        "hrsh7th/cmp-path",
        "hrsh7th/cmp-buffer",
        "L3MON4D3/LuaSnip",
    },
    config = function()
        local cmp = require('cmp')
        local luasnip = require('luasnip')
        local opts = {
            -- 从哪里获取补全结果
            sources = cmp.config.sources {
                { name = "nvim_lsp" },
                { name = "buffer"},
                { name = "path" },
            },
            mapping = cmp.mapping.preset.insert({
                -- 使 'enter' 键选择补全
                ["<CR>"] = cmp.mapping.confirm({ select = true }),
                -- 超级 tab 行为
                ["<tab>"] = cmp.mapping(function(original)
                    if cmp.visible() then
                        cmp.select_next_item() -- 如果正在补全则运行补全选择
                    elseif luasnip.expand_or_jumpable() then
                        luasnip.expand_or_jump() -- 扩展代码片段
                    else
                        original()      -- 如果没有补全则运行原始行为
                    end
                end, {"i", "s"}),
                ["<S-tab>"] = cmp.mapping(function(original)
                    if cmp.visible() then
                        cmp.select_prev_item()
                    elseif luasnip.expand_or_jumpable() then
                        luasnip.jump(-1)
                    else
                        original()
                    end
                end, {"i", "s"}),
            }),
            snippets = {
                expand = function(args)
                    luasnip.lsp_expand(args)
                end,
            },
        }
        cmp.setup(opts)
    end,
},
```

现在我们的 tab 键在超级 tab 方式中被彻底重载。
 - 如果补全窗口打开，按下 tab 将选择列表中的下一个项目。
 - 如果你在代码片段上按下 tab，代码片段将扩展，并继续按下 tab 将光标移动到下一个选择点。
 - 如果你既没有代码补全也没有扩展代码片段，它将表现得像一个普通的 `tab` 键。

现在我们需要编写一些代码片段。_LuaSnip_ 支持多种代码片段格式，
包括流行的 [TextMate](https://macromates.com/textmate/manual/snippets) 的子集，
[Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets) 代码片段格式，
以及其自己的 [基于 Lua 的](https://github.com/L3MON4D3/LuaSnip/blob/master/Examples/snippets.lua) API。

以下是一些我发现有用的代码片段：

```snipmate
snippet pub "public access control"
  public $0

snippet priv "private access control"
  private $0

snippet if "if statement"
  if $1 {
    $2
  }$0

snippet ifl "if let"
  if let $1 = ${2:$1} {
    $3
  }$0

snippet ifcl "if case let"
  if case let $1 = ${2:$1} {
    $3
  }$0

snippet func "function declaration"
  func $1($2) $3{
    $0
  }

snippet funca "async function declaration"
  func $1($2) async $3{
    $0
  }

snippet guard
  guard $1 else {
    $2
  }$0

snippet guardl
  guard let $1 else {
    $2
  }$0

snippet main
  @main public struct ${1:App} {
    public static func main() {
      $2
    }
  }$0
```

另一个值得一提的流行代码片段插件是
[UltiSnips](https://github.com/SirVer/ultisnips)，它允许你在定义代码片段时使用内联
Python，从而可以编写一些非常强大的代码片段。

# 结论

一旦一切配置正确，使用 Neovim 进行 Swift 开发将是一个很好的体验。这里有成千上万的插件供你探索，本文为你在 Neovim 中构建 Swift 开发体验提供了一个坚实的基础。

# 文件

以下是这些配置文件的最终形式。

```lua
-- init.lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
    vim.fn.system({
        "git",
        "clone",
        "--filter=blob:none",
        "https://github.com/folke/lazy.nvim.git",
        "--branch=stable",
        lazypath
    })
end
vim.opt.rtp:prepend(lazypath)
require("lazy").setup("plugins", {
  ui = {
    icons = {
      cmd = "",
      config = "",
      event = "",
      ft = "",
      init = "",
      keys = "",
      plugin = "",
      runtime = "",
      require = "",
      source = "",
      start = "",
      task = "",
      lazy = "",
    },
  },
})

vim.opt.wildmenu = true
vim.opt.wildmode = "list:longest,list:full" -- don't insert, show options

-- line numbers
vim.opt.nu = true
vim.opt.rnu = true

-- textwrap at 80 cols
vim.opt.tw = 80

-- set solarized colorscheme.
-- NOTE: Uncomment this if you have installed solarized, otherwise you'll see
--       errors.
-- vim.cmd.background = "dark"
-- vim.cmd.colorscheme("solarized")
-- vim.api.nvim_set_hl(0, "NormalFloat", { bg = "none" })
```

```lua
-- lua/plugins/codecompletion.lua
return {
  {
    "hrsh7th/nvim-cmp",
    version = false,
    event = "InsertEnter",
    dependencies = {
      "hrsh7th/cmp-nvim-lsp",
      "hrsh7th/cmp-path",
      "hrsh7th/cmp-buffer",
    },
    config = function()
      local cmp = require('cmp')
      local luasnip = require('luasnip')
      local opts = {
        sources = cmp.config.sources {
          { name = "nvim_lsp", },
          { name = "path", },
          { name = "buffer", },
        },
        mapping = cmp.mapping.preset.insert({
          ["<CR>"] = cmp.mapping.confirm({ select = true }),
          ["<tab>"] = cmp.mapping(function(original)
            print("tab pressed")
            if cmp.visible() then
              print("cmp expand")
              cmp.select_next_item()
            elseif luasnip.expand_or_jumpable() then
              print("snippet expand")
              luasnip.expand_or_jump()
            else
              print("fallback")
              original()
            end
          end, {"i", "s"}),
          ["<S-tab>"] = cmp.mapping(function(original)
            if cmp.visible() then
              cmp.select_prev_item()
            elseif luasnip.expand_or_jumpable() then
              luasnip.jump(-1)
            else
              original()
            end
          end, {"i", "s"}),

        })
      }
      cmp.setup(opts)
    end,
  },
  { "hrsh7th/cmp-nvim-lsp", lazy = true },
  { "hrsh7th/cmp-path", lazy = true },
  { "hrsh7th/cmp-buffer", lazy = true },
}
```

```lua
-- lua/plugins/lsp.lua
return {
  {
    "neovim/nvim-lspconfig",
    config = function()
      local lspconfig = require('lspconfig')
    lspconfig.sourcekit.setup {
      capabilities = {
          workspace = {
            didChangeWatchedFiles = {
              dynamicRegistration = true,
            },
          },
        },
      }

      vim.api.nvim_create_autocmd('LspAttach', {
        desc = "LSP Actions",
        callback = function(args)
          vim.keymap.set("n", "K", vim.lsp.buf.hover, {noremap = true, silent = true})
          vim.keymap.set("n", "gd", vim.lsp.buf.definition, {noremap = true, silent = true})
        end,
      })
    end,
  },
}
```

```lua
-- lua/plugins/snippets.lua
return {
  {
    'L3MON4D3/LuaSnip',
    lazy = false,
    config = function(opts)
      local luasnip = require('luasnip')
      luasnip.setup(opts)
      require('luasnip.loaders.from_snipmate').load({ paths = "./snippets"})
    end,
  }
}
```

```snipmate
# snippets/swift.snippets

snippet pub "public access control"
  public $0

snippet priv "private access control"
  private $0

snippet if "if statement"
  if $1 {
    $2
  }$0

snippet ifl "if let"
  if let $1 = ${2:$1} {
    $3
  }$0

snippet ifcl "if case let"
  if case let $1 = ${2:$1} {
    $3
  }$0

snippet func "function declaration"
  func $1($2) $3{
    $0
  }

snippet funca "async function declaration"
  func $1($2) async $3{
    $0
  }

snippet guard
  guard $1 else {
    $2
  }$0

snippet guardl
  guard let $1 else {
    $2
  }$0

snippet main
  @main public struct ${1:App} {
    public static func main() {
      $2
    }
  }$0
  ```