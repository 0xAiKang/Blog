---
title: Mac 下有哪些好用的终端工具
date: 2021-01-12 22:33:12
tags: ["Tutorial", "Mac"]
categories: ["Tutorial", "Mac"]
---

这篇笔记主要是用来整理自己一直在使用的一些较为好用的终端工具/扩展。

<!-- more -->

因为我个人的终端配置是`ZSH` + `iTerm2`，所以本文的部分`ZSH` 扩展可能不适用于其他`Shell`用户。

## brew 
经常与终端打交道的用户，对这个一定不陌生，它就是类似`Ubuntu`下的`apt-get`这样的包管理工具。

通常我需要搭建一个全新的开发环境时，它一定是第一个需要安装的工具。

安装 brew（[brew 官网](https://brew.sh/)）
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

常用命令如下：
常用命令如下：
| 命令 | 描述 |
|--- | --- |
|brew search package | 搜索软件包|
|brew install package | 安装软件包|
|brew uninstall package | 卸载软件包|
|brew list | 列出已安装清单|
|brew help | 获取帮助|

## OSX 扩展
`osx` 扩展是`zsh` 提供的一个控制终端和访达（功能之一）的扩展工具。

其中最为常用是`ofd`命令，将当前`shell`窗口在访达中打开。

另一个较为常用的命令是`cdf`，可在`shell`中直接跳转至当前访达窗口所在的路径（如果存在多个访达窗口，那么跳转至最前面的那个）。

其他常用命令如下：

命令 | 描述
---|---
tab | 在当前目录打开一个新窗口
split_tab | 在当前窗口打开一个水平窗口
vsplit_tab | 在当前窗口打开一个垂直窗口
ofd | 在访达窗口中打开当前目录
pfd | 返回最前面的访达窗口的路径
pfs | 返回当前查找程序选择
cdf | cd 到当前访达窗口所在的路径
pushdf | pushed 到当前访达目录
quick-look | 快速查看指定文件
man-preview | 在预览应用程序中打开特定的手册页
showfiles | 显示隐藏文件
hidefiles | 隐藏隐藏的文件
rmdsstore | 以递归方式删除目录中的.DS_Store文件

## tmux
`tmux` 是一个终端下窗口分割的工具，有关它的具体介绍，请查阅[这篇笔记](https://www.0x2beace.com/tmux-quick-start/)。

## autojump
autojump - 目录快速跳转命令行工具，从此告别`cd... cd...`。

autojump 是一个`Windows`、`Linux`、`macOS` 都能使用的命令行工具，这是仅介绍`macOS` 的安装方式。

```
brew install autojump
```
使用`brew`安装完成之后，还需要进行配置，以下方法二选一：
* 在 `~/.bash_profile` 文件中加入语句 `[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh`。
* 在 `~/.zshrc` 文件中，修改 `plugins=(git)` 插件配置行，以开启 `zsh` 对 `autojump` 插件的支持 `plugins=(git autojump)`。

### 常用命令
命令 | 描述
--- | ---
j foo | 跳转到包含 foo 的目录
jc bar | 跳转到包含 bar 的子目录
jo file | 在访达中打开包含 file 的目录
autojump --help | 打开帮助列表

## Spaceship ZSH
Spaceship ZSH——是一个极简、强大和可定制的`ZSH`提示符。

我是在无意间发现的这个终端工具的，先来看一下实际效果。

![image](https://user-images.githubusercontent.com/10276208/36086434-5de52ace-0ff2-11e8-8299-c67f9ab4e9bd.gif)

### 特点
Spaceship ZSH 有很多很棒的特点，这里仅仅列举一些我所看见的。

* 颜值即正义
* 展示当前Git 仓库的状态
* 展示各种语言的当前版本
* 展示最后一条命令的总执行时间

### 安装
Spaceship ZSH 的安装方式有多种，这里仅介绍通过`oh-my-zsh`的安装方式，其他方式可参考[官网](https://denysdovhan.com/spaceship-prompt/)。

1. 克隆仓库
```
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
```
2. 将`spaceship.zsh-theme` 链接到`oh-my-zsh` 的主题目录
```
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```
3. 编辑`~/.zshrc`
```
ZSH_THEME="spaceship"
```

## tldr
tldr 是一个比man 更好用的命令行手册。

它衍生出了各种语言的客户端，这里直接使用官网推荐的方式进行安装：
```
npm install -g tldr
```

安装完成之后，第一次使用`tldr`命令需要下载相关依赖：
```
tldr tar
Page not found. Updating cache...
Error: connect ECONNREFUSED 127.0.0.1:443
```
如果出现上面这个输出，表示命令行需要使用代理，如果不知道如何设置，可以参考[这篇笔记](https://www.0x2beace.com/how-to-make-terminal-commands-go-through-proxy/)。

正常输出如下：
```
tldr tar
✔ Page not found. Updating cache...
✔ Creating index...

  tar

  Archiving utility.
  Often combined with a compression method, such as gzip or bzip.
  More information: https://www.gnu.org/software/tar.

  - [c]reate an archive from [f]iles:
    tar cf target.tar file1 file2 file3

  - [c]reate a g[z]ipped archive from [f]iles:
    tar czf target.tar.gz file1 file2 file3

  - [c]reate a g[z]ipped archive from a directory using relative paths:
    tar czf target.tar.gz --directory=path/to/directory .

  - E[x]tract a (compressed) archive [f]ile into the current directory:
    tar xf source.tar[.gz|.bz2|.xz]

  - E[x]tract a (compressed) archive [f]ile into the target directory:
    tar xf source.tar[.gz|.bz2|.xz] --directory=directory

  - [c]reate a compressed archive from [f]iles, using [a]rchive suffix to determine the compression program:
    tar caf target.tar.xz file1 file2 file3

  - Lis[t] the contents of a tar [f]ile [v]erbosely:
    tar tvf source.tar

  - E[x]tract [f]iles matching a pattern:
    tar xf source.tar --wildcards "*.html"
```
上面那个node 的客户端不是交互式的，如果需要自动的，可以使用 [tldr++](https://github.com/isacikgoz/tldr)，这是一个Go 语言编写的交互式客户端。

### 参考链接
* [安装 zsh](https://github.com/robbyrussell/oh-my-zsh)
* [如何启用 zsh 的插件](https://github.com/ohmyzsh/ohmyzsh#enabling-plugins)
* [OSX 插件](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/osx)
* [Spaceship ZSH](https://github.com/denysdovhan/spaceship-prompt#installing)
* [autojump——自动跳转文件目录](https://github.com/wting/autojump)
* [tldr——比man 更好用的命令行手册](https://github.com/tldr-pages/tldr)