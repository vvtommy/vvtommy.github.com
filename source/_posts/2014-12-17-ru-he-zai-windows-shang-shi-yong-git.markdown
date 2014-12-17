---
layout: post
title: "如何在 Windows 上使用 git"
date: 2014-12-17 17:24:15 +0800
comments: true
categories: git windows
---

由于 [git] 严重依赖 [shell], 而 Windows 下并没有功能完整的 [*nix shell]，所以目前 Windows 下安装 [git] 都采用整合 [shell] 的安装包的形式提供。

## msysgit
Windows 下比较常用的 [git] 整合安装包为[msysgit]. 它整合了 [MinGW](Minimalist GNU for Windows) 来提供 [shell] 功能，所使用的 [shell] 程序是 [bash](Bourne-Again shell，源于 [sh] 的名称 Bourne shell)。并且提供 [git] 的图形化工具 git-gui 和 git 分支管理的图形化工具 gitk。

它的优点是比较传统，网上很多资料中的截图都是使用的这款整合安装包。

缺点现在看来比较多: 

1. 由于使用了 Windows 自带的终端程序，导致对字符集支持不佳。
2. 除了 [bash] 之外没有其他 [shell] 可供替换
3. 与其他平台使用体验差异较大
4. [msysgit] 自身的 bug 和使用体验问题。它本身就与相对健全的 [Cygwin] 有较大的差距
5. 设计目标是"在 windows 上使用 [git]"，所以对于一款用与开发的 [shell] 而言，功能比较单薄。
6. 丑

截图:  
![msysgit bash](http://msysgit.github.io/img/gw1.png)

## babun

目前在 Windows 下推荐的整合工具是 [babun]。这款工具不同于[msysgit]，它并不是只针对"在 windows 上使用 [git]"而开发的，其目的是为了在 Windows 下提供一款功能齐备的终端工具，安装简单（相对于完全安装 [Cygwin]），提供 Mac、Linux下相同的体验，而这正是我们所需要的。它自带一个裁剪过的 [Cygwin]，并且使用 [zsh] 作为默认的 [shell] 解释器。 并且具有包管理工具，可以随时安装新的工具。其他功能请参阅 https://github.com/babun/babun 。

它具备如下功能:
- 按需剪裁的 [Cygwin]
- 一键安装工具，并且安装时不需要 Administrator 权限
- `pact` 包管理工具，方便的安装、管理各种应用程序、库
- 支持插件
- 支持 [git]
- 预装 [oh-my-zsh] 组件，其本身提供了难以置信的操作体验
- HTTP(s) 代理
- 自动更新

截图:  
![babun 1](https://raw.githubusercontent.com/babun/babun.github.io/master/images/screen_vim.png)
![babun 2](https://raw.githubusercontent.com/babun/babun.github.io/master/images/screen_vim.png)

[git]: http://git-scm.com
[git-flow]: https://github.com/nvie/gitflow
[msysgit]: http://msysgit.github.io
[MinGW]: http://www.mingw.org
[babun]: https://github.com/babun/babun
[Cygwin]: http://zh.wikipedia.org/zh/Cygwin
[zsh]: http://zh.wikipedia.org/wiki/Z_shell
[shell]: http://zh.wikipedia.org/wiki/殼層
[*nix shell]: http://zh.wikipedia.org/wiki/Unix_shell
[bash]: http://zh.wikipedia.org/wiki/Bash
[sh]: http://zh.wikipedia.org/wiki/Bourne_shell
[oh-my-zsh]: https://github.com/robbyrussell/oh-my-zsh
