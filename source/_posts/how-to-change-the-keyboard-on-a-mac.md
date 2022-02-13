---
title: Mac 如何给键盘改键
date: 2022-02-13 17:24:33
tags: ["Hexo", "Tutorial"]
categories: Tutorial
---

在Mac 上，可以使用 [Karabiner-Elements](https://github.com/pqrs-org/Karabiner-Elements) 进行改键，可以同时给笔记本和外接键盘进行改键，非常方便。

* 系统：macOS Monterey 12.1
* 外接键盘：Keychron K6

## 简单改键
Caps Lock 键占据了非常重要的位置，这个是没有办法接受的，所以需要将其替换成 Control 键。

打开 Karabiner-Elements，给予各种授权之后，直接在 `Simple modifications` 下面，选择需要替换的键。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220213162744.png)

## 复杂改键
复杂改键通常是以组合的方式去修改，可以在 [Karabiner-Elements complex_modifications rules](https://ke-complex-modifications.pqrs.org/) 上查找自己想要的规则。

因为我的外接键盘是 Keychron K6，所以直接搜索键盘名称，就可以看到很多已经写好的规则，点击 Import。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220213162435.png)
导入之后，可以自行选择启用某个规则。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220213171820.png)