---
title: 解决Firefox “已阻止载入混合活动内容”
date: 2020-07-18 15:15:30
tags: ["https"]
categories: [""]
---

最近需要将项目迁移至一台新的服务器，其中涉及到多个站点的`http`与`https`之间的转换。

网站起初不能正常访问时，我没在意，以为是网络延迟（因为服务器放在国外），直到我打开控制台发现了如下异常：

<!-- more -->

![异常内容](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200718144533.png)

这时我才意识到并不是网络延迟的问题，而是项目没有配置好。

## 什么是混合内容
> 当用户访问使用HTTPS的页面时，他们与web服务器之间的连接是使用SSL加密的，从而保护连接不受嗅探器和中间人攻击。
如果HTTPS页面包括由普通明文HTTP连接加密的内容，那么连接只是被部分加密：非加密的内容可以被嗅探者入侵，并且可以被中间人攻击者修改，因此连接不再受到保护。当一个网页出现这种情况时，它被称为混合内容页面。 —— [MDN](https://developer.mozilla.org/zh-CN/docs/Security/MixedContent)

通俗一点解释就是：`https` 的页面中混合着`http` 的请求，而这种请求不会被浏览器正常接受的，也被称作为混合内容页面。

## 解决方案
既然已经明白了为什么会产生这个问题，那么要解决起来也就非常简单了。

### 让Firefox暂时不阻止
1. 打开新标签页，在地址栏输入 `about:config`，进入`FireFox`高级配置页面。
2. 搜索`security.mixed_content.block_active_content`，将默认值`true`更改为`false`。

这种方式仅适用于本地调试。

### 避免在HTTPS页面中包含HTTP的内容
更直接有效的方式应该是约定好项目中的协议，统一使用`https`或者`http`。

### 参考连接
* [什么是混合内容——MDN](https://developer.mozilla.org/zh-CN/docs/Security/MixedContent)
* [https访问遇到“已阻止载入混合内容”](https://segmentfault.com/a/1190000015722535)