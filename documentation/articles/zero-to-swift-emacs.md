---
layout: page
date: 2024-06-05 12:00:00
title: 为 Swift 开发配置 Emacs
author: [al45tair]
---

Emacs 是一个高度可定制的文本编辑器，它起源于数字设备公司 TECO 编辑器的宏包。
多年来存在过许多类似 Emacs 的编辑器和衍生品，但本指南将专注于 [GNU
Emacs](https://www.gnu.org/software/emacs/)。

本指南将指导你为 Swift 开发设置一个新的 Emacs 安装；它不假定任何特定的操作系统，
但在相关时可能会给出系统特定的提示来帮助你入门。它也不是要教你使用 Emacs；如果你
想学习 Emacs，请尝试这个资源：[Emacs
Tour](https://www.gnu.org/software/emacs/tour/)。

我们假设在本指南中你已经安装了 Swift；如果没有，请在继续之前[按照 Swift 网站上适合你平台的安装说明](https://www.swift.org/install)。

## 安装 Emacs

在 GNU Emacs 网站上有通用的[安装说明](https://www.gnu.org/software/emacs/download.html)。下面为一些常见平台提供了具体说明。

### macOS

[Emacs for Mac OS X](https://emacsformacosx.com) 网站提供了标准 Mac 磁盘映像中的通用二进制文件。从该网站下载映像，打开它并将其拖到你的 `Applications` 文件夹是在 macOS 上安装原生 Emacs 的最简单方法。

### Microsoft Windows

你可以从[附近的 GNU 镜像](http://ftpmirror.gnu.org/emacs/windows)或[主 GNU FTP 服务器](http://ftp.gnu.org/gnu/emacs/windows/)下载适用于 Windows 的 GNU Emacs。最简单的方法可能是运行 `emacs-<version>-installer.exe` 可执行文件，它将安装 Emacs 并为你设置一些快捷方式。

### 基于 Debian 的 Linux（包括 Ubuntu）

在提示符下输入

```console
$ sudo apt-get install emacs
```

或者如果你在服务器系统或容器中，没有计划使用 GUI，则输入

```console
$ sudo apt-get install emacs-nox
```

### 基于 RedHat 的 Linux（RHEL、Centos、Fedora）

在提示符下输入

```console
$ sudo dnf install emacs
```

（在较旧的版本上你可能需要使用 `yum` 而不是 `dnf`，但如果有的话后者是更好的选择。）

## 配置 Emacs

Emacs 老手可能记得手动下载和安装 Lisp 包，但现在最好使用包管理器。
我们还将配置 [MELPA](https://melpa.org)，这是一个流行的 Emacs 包仓库。要做到这一点，打开 Emacs，输入 `C-x C-f ~/.emacs` 并按回车。然后将以下内容添加到你的 `.emacs` 文件中：

```lisp
;;; 添加 MELPA 作为包源
(require 'package)
(setq package-enable-at-startup nil)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/"))
(package-initialize)
```

我们还将设置 [`use-package`](https://github.com/jwiegley/use-package)，因为如果你想要能够只复制你的 `~/.emacs` 到一台新机器并让事情正常工作，这会简化包安装。为此，添加

```lisp
;;; 引导 `use-package'
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))

(eval-when-compile
  (require 'use-package))
```

过一会儿我们会想要能够定位 `sourcekit-lsp`；这取决于 Swift 的安装位置，所以让我们添加一个 Lisp 函数来做这个：

```lisp
;;; 定位 sourcekit-lsp
(defun find-sourcekit-lsp ()
  (or (executable-find "sourcekit-lsp")
      (and (eq system-type 'darwin)
           (string-trim (shell-command-to-string "xcrun -f sourcekit-lsp")))
      "/usr/local/swift/usr/bin/sourcekit-lsp"))
```

接下来，我们将安装一些有用的包：

```lisp
;;; 我们想要为 Swift 开发安装的包

;; .editorconfig 文件支持
(use-package editorconfig
    :ensure t
    :config (editorconfig-mode +1))

;; Swift 编辑支持
(use-package swift-mode
    :ensure t
    :mode "\\.swift\\'"
    :interpreter "swift")

;; Rainbow delimiters 使嵌套分隔符更容易理解
(use-package rainbow-delimiters
    :ensure t
    :hook ((prog-mode . rainbow-delimiters-mode)))

;; Company mode（补全）
(use-package company
    :ensure t
    :config
    (global-company-mode +1))

;; 用于与 swift-lsp 接口的包。
(use-package lsp-mode
    :ensure t
    :commands lsp
    :hook ((swift-mode . lsp)))

;; lsp-mode 的 UI 模块
(use-package lsp-ui
    :ensure t)

;; sourcekit-lsp 支持
(use-package lsp-sourcekit
    :ensure t
    :after lsp-mode
    :custom
    (lsp-sourcekit-executable (find-sourcekit-lsp) "查找 sourcekit-lsp"))
```

我们还将添加一些包来让事情看起来更漂亮和更现代；如果你愿意，你也可以安装一个主题或更改字体 - 自定义 Emacs 的可能性几乎是无穷的：

```lisp
;; Powerline
(use-package powerline
  :ensure t
  :config
  (powerline-default-theme))

;; Spaceline
(use-package spaceline
  :ensure t
  :after powerline
  :config
  (spaceline-emacs-theme))
```

最后，让我们关闭启动画面，因为我们不想每次启动 Emacs 时都看到它，我们还将禁用工具栏：

```lisp
;;; 不显示启动画面
(setq inhibit-startup-screen t)

;;; 禁用工具栏
(tool-bar-mode -1)
```

输入 `C-x C-s` 保存你的新 `.emacs` 文件，然后重启 Emacs。

## 结论

我们能够设置 Emacs 来编辑 Swift 代码，具有语法高亮和与 SourceKit-LSP 的集成，以及对现代最佳实践的支持，如使用 `.editorconfig` 文件让项目轻松设置自己的制表符宽度和格式化偏好。

## 文件

这是包含完整配置的 `.emacs` 文件：

```lisp
;;; 添加 MELPA 作为包源
(require 'package)
(setq package-enable-at-startup nil)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/"))
(package-initialize)

;;; 引导 `use-package'
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))

(eval-when-compile
  (require 'use-package))

;;; 定位 sourcekit-lsp
(defun find-sourcekit-lsp ()
  (or (executable-find "sourcekit-lsp")
      (and (eq system-type 'darwin)
           (string-trim (shell-command-to-string "xcrun -f sourcekit-lsp")))
      "/usr/local/swift/usr/bin/sourcekit-lsp"))

;;; 我们想要为 Swift 开发安装的包

;; .editorconfig 文件支持
(use-package editorconfig
    :ensure t
    :config (editorconfig-mode +1))

;; Swift 编辑支持
(use-package swift-mode
    :ensure t
    :mode "\\.swift\\'"
    :interpreter "swift")

;; Rainbow delimiters 使嵌套分隔符更容易理解
(use-package rainbow-delimiters
    :ensure t
    :hook ((prog-mode . rainbow-delimiters-mode)))

;; Company mode（补全）
(use-package company
    :ensure t
    :config
    (global-company-mode +1))

;; 用于与 swift-lsp 接口的包。
(use-package lsp-mode
    :ensure t
    :commands lsp
    :hook ((swift-mode . lsp)))

;; lsp-mode 的 UI 模块
(use-package lsp-ui
    :ensure t)

;; sourcekit-lsp 支持
(use-package lsp-sourcekit
    :ensure t
    :after lsp-mode
    :custom
    (lsp-sourcekit-executable (find-sourcekit-lsp) "查找 sourcekit-lsp"))

;; Powerline
(use-package powerline
  :ensure t
  :config
  (powerline-default-theme))

;; Spaceline
(use-package spaceline
  :ensure t
  :after powerline
  :config
  (spaceline-emacs-theme))

;;; 不显示启动画面
(setq inhibit-startup-screen t)

;;; 禁用工具栏
(tool-bar-mode -1)
```
