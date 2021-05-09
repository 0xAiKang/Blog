---
title: 如何配置一个简洁高效的 Zsh
date: 2021-05-09 08:48:19
tags: ["Linux"]
categories: ["Linux"]
---

Shell 是类 Unix 系统中超级好用的工具，而 Zsh 是 Shell 中的佼佼者，但是现在网上一搜索 Zsh 的配置方案，遍地都是的互相复制粘贴的`oh-my-zsh` 配置方案。事实上 `oh-my-zsh` 并不好用，严重拖慢了 Zsh 的速度，下面分享一个简洁高效的Zsh 配置方案。

<!-- more -->

### 安装Zsh

这里直接从发行版的源中进行安装，简单、高效：
```shell
sudo apt-get update
sudo apt-get install zsh
```

### 安装插件及主题
两个插件一个主题：
* `zsh-autosuggestions`：这个是自动建议插件，能够自动提示你需要的命令。
* `zsh-syntax-highlighting`：这个是代码高亮插件，能够使你的命令行各个命令清晰明了。
* `zsh-theme-powerlevel10k` 这个主题提供漂亮的提示符，可以显示当前路径、时间、命令执行成功与否，还能够支持 git 分支显示等等。

一键安装：
```shell
sudo apt-get install zsh-autosuggestions zsh-syntax-highlighting zsh-theme-powerlevel9k
```

不出意外的话，会提示：
```
E: Unable to locate package zsh-autosuggestions
```

这是因为软件源中并没有`zsh-autosuggestions` 这个package，所以需要手动添加软件包。

这里可以直接进入[opensuse](https://software.opensuse.org/) 进行搜索，需要的软件包。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084534.png)

找到对应的发行版，点击Export Download：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084558.png)

这里提供两种方式供我们选择：

1. 添加软件源并手动安装
2. 直接抓取二进制软件包

直接给结论，第二种方式更简单些，直接下载.deb文件之后，就可以安装了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084624.png)

选择对应的操作系统以及版本，右键拷贝链接地址：

```shell
$  wget https://download.opensuse.org/repositories/shells:/zsh-users:/zsh-autosuggestions/xUbuntu_18.04/amd64/zsh-autosuggestions_0.5.0+1.1_amd64.deb
$ sudo dpkg -i zsh-autosuggestions_0.5.0+1.1_amd64.deb
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084643.png)

再次执行命令：
```shell
sudo apt-get install zsh-autosuggestions zsh-syntax-highlighting zsh-theme-powerlevel9k
```
至此，插件和主题就安装完成了。

### 更改默认Shell

```
$ chsh -s /usr/bin/zsh
```
注销并重新登录，再次登录成功时，默认启用了zsh。

### 配置插件和主题

第一次进入 Zsh 会自动出现一个配置界面，这个界面可以根据需要自定义 Zsh。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084659.png)

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084717.png)

配置界面中各个菜单代表的意思分别是：

1：设置命令历史记录相关的选项
2：设置命令补全系统
3：设置热建
4：选择各种常见的选项，只需要选择“On”或者“Off”
0：退出，并使用空白（默认）配置
a：终止设置并退出
q：退出


### 启用插件和主题
Zsh 的配置文件是 `~/.zshrc` 文件，这个文件在你的用户目录下 `~/`。删掉了这个文件，再次进入 Zsh时，会再次进入 Zsh 的配置界面。

将以下代码加入到 `~/.zshrc` 文件中，以启用插件和主题：
```
source /usr/share/powerlevel9k/powerlevel9k.zsh-theme
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

再次注销并登录，即可看到新的终端界面：
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210509084734.png)

### 原文地址
* [配置一个简洁高效的 Zsh](https://linux.cn/article-13030-1.html)