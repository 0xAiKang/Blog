---
title: Vim 安装 molokai 配色方案
date: 2020-07-18 14:36:04
tags: ["Vim", "Tutorial"]
categories: ["Tutorial"]
---

像[solarized](https://github.com/altercation/solarized)、[gruvbox](https://github.com/morhetz/gruvbox)、 [molokai](https://github.com/tomasr/molokai)、这些都是大名鼎鼎的VIM 配色方案，本文只介绍如何安装 `molokai` 。

<!-- more -->

按照顺序执行完上面的命令，即可使用最经典的配色方案了。
```
cd ~
mkdir .vim && cd .vim
git clone https://github.com/tomasr/molokai.git
cp -rf molokai/colors/ ./colors
echo colorscheme molokai >> ~/.vimrc
echo set t_Co=256 >> ~/.vimrc
echo set background=dark  >> ~/.vimrc
```

实际效果：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200718142924.png)