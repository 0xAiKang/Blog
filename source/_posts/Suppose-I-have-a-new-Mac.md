---
title: 假如我有一台新的 Mac
date: 2022-02-20 19:26:13
tags:tags: ["Mac"]
categories: ["Mac"]
---


从去年就想入手一台新的MacBookPro，中间因为一些原因，没有第一时间选择从官网下单，而是让朋友帮忙从其他渠道代购。

这一等就是好几个月，直到二月初才拿到货...

好了，现在我们真的有了一台新 Mac。正式开始之前，请自行准备科学上网环境，本文不使用任何代理源。

<!-- more -->

## 触控板和键位配置
### 三指拖移
Mac 的三指拖移手势能够大大的提高触摸板的使用频率，减少触控板左下角按键的左键功能使用，但默认是没有启用的。

所以我通常拿到一台新Mac 的第一件事情就是打开三指拖移。

按以下路径找到对应的设置面板：`系统设置偏好 -> 辅助功能 -> 指针选项 -> 触控板选项`


![](https://wxnacy.com/images/mac-chukong2.png)

点击启用即可。

### 键盘改键
新款的MacBook 的剪刀键盘，有好几个不常用键占据了非常重要的位置，这是肯定不能接受的。

具体的改键过程，请查阅[这篇笔记](https://www.0x2beace.com/how-to-change-the-keyboard-on-a-mac/)。

## 命令行工具

### Homebrew
经常与终端打交道的用户，对这个一定不陌生，它就是类似`Ubuntu`下的 `apt-get` 这样的包管理工具。

通常我需要搭建一个全新的开发环境时，它一定是第一个需要安装的工具。

打开终端，执行以下命令：
```php
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

常用命令如下：

命令 | 描述
--- | ---
brew search package | 搜索软件包
brew install package | 安装软件包
brew uninstall package | 卸载软件包
brew list | 列出已安装清单
brew services list | 列出使用brew 运行的服务
brew upgrade xxx|升级软件
brew info nginx|查看软件包详情
brew help | 获取帮助 |  
brew tap homebrew/php | 更新Homebrew 安装源
brew link php@7.3 | 手动创建符号链接Cellar
brew unlink php@7.3 | 取消链接

[homebrew](https://brew.sh/index_zh-cn) 的重要性就不必多说了，Mac 下必装工具之一。

如果在安装`homebrew` 的过程中，因为众所周知的原因导致安装失败，可以看一下下面两篇笔记，说不定会有帮助：
* [如何让终端命令走代理](https://www.0x2beace.com/how-to-make-terminal-commands-go-through-proxy/)
* [如何解决类似 curl:(7) Failed to connect to raw.githubusercontent.com port 443:Connection refused 的问题](https://www.0x2beace.com/How-to-solve-problems-like-curl-7-Failed-to-connect-to-raw-githubusercontent-com-port-443-Connection-refused/)

### Homebrew Cask
Homebrew Cask扩展了Homebrew，使得macOS GUI 应用程序的安装和管理更优雅、简单和快速。

看看官方的实例图：

![](https://camo.githubusercontent.com/f8b75a5e461338a90db6acf4db8f5bc9cf620bfba65a5a490ed10bd08f457b52/68747470733a2f2f692e696d6775722e636f6d2f464e4e4d36574c2e676966)

Homebrew Cask 先下载软件后解压到统一的目录中 `/opt/homebrew-cask/Caskroom`，然后再软链到 `~/Applications/` 目录下，省掉了自己下载、解压、拖拽安装等步骤，同样的，卸载相当简单和干净，一句命令就可以完成。

更多Homebrew Cask 的用法请[自行查阅](https://github.com/Homebrew/homebrew-cask/blob/master/USAGE.md)。

### zsh
zsh 是一种shell语言，兼容bash，提供强大的命令行功能，比如tab补全，自动纠错功能等。

安装 zsh：
```shell
brew install zsh
```

`oh-my-zsh`则是基于`zsh` 命令行，提供了主题配置，插件机制等便捷操作，将`zsh` 变得更加强大。
```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
个性化配置、别名配置、插件启用都在目录 `~/.zshrc` 下，可以自行扩展。

#### 让终端走代理
```shell
# zsh
echo export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890 >> ~/.zprofile

# bash
echo export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890 >> ~/.bash_profile
```


## 其他环境和插件

### Finder（访达）预览插件
用于Finder 快速显示文件的内容，这个工具的安装就是依靠`Homebrew Cask`。

```shell
brew install qlcolorcode qlstephen qlmarkdown quicklook-json qlimagesize suspicious-package apparency quicklookase qlvideo
```

* qlcolorcode: 代码文件预览时高亮
* qlstephen: 以纯文本的形式预览无拓展名或者未知拓展名的文件
* qlmarkdown: 预览渲染后的 markdown 文件
* quicklook-json: 预览格式化后的 json 文件
* ProvisionQL: ipa文件信息展示
* QuickLookAPK：apk文件信息展示

### macos 扩展
`macos` 扩展是`zsh` 提供的一个控制终端和访达（功能之一）的扩展工具。

启用方式很简单，直接编辑`~/.zshrc`，然后添加 `macos` 到插件列表即可：
```
plugins=(
    macos
)
```

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

### tmux
`tmux` 是一个终端下窗口分割的工具，有关它的具体介绍，请查阅[这篇笔记](https://www.0x2beace.com/tmux-quick-start/)。

### autojump
autojump - 目录快速跳转命令行工具，从此告别`cd... cd...`。

autojump 是一个`Windows`、`Linux`、`macOS` 都能使用的命令行工具，这是仅介绍`macOS` 的安装方式。

```
brew install autojump
```
使用`brew`安装完成之后，还需要进行配置，以下方法二选一：
* 在 `~/.bash_profile` 文件中加入语句 `[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh`。
* 在 `~/.zshrc` 文件中，修改 `plugins=(git)` 插件配置行，以开启 `zsh` 对 `autojump` 插件的支持 `plugins=(git autojump)`。

其他常用命令如下：

命令 | 描述
--- | ---
j foo | 跳转到包含 foo 的目录
jc bar | 跳转到包含 bar 的子目录
jo file | 在访达中打开包含 file 的目录
autojump --help | 打开帮助列表

## 参考链接
* [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)
* [Homebrew](https://brew.sh)
* [Homebrew Cask](https://github.com/Homebrew/homebrew-cask)
* [Quick Look plugins](https://github.com/sindresorhus/quick-look-plugins)
* [zsh-users](https://github.com/zsh-users)
* [MacOS plugin](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/macos)
* [autojump](https://github.com/wting/autojump)