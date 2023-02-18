---
title: ChatGPT Access Denied
date: 2023-02-18 11:44:20
tags: ["一些经验"]
categories: ["一些经验"]
---

因为 openai 屏蔽了国内访问，无论是ChatGPT 还是ChatGPT 应用，尽管开了全局代理，直接访问还是可能访问不了的，会重定向到下面这个页面：

<!-- more -->

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230216175714.png)
目前并不能通过在 ChatGPT 应用中，设置代理的方式来解决这个问题。

不过倒是可以通过更新 Clash 配置文件的方式，走规则代理。

## 解决方案
打开 Clash 配置文件夹，找到当前正在使用的配置文件，编辑它。

新增以下规则：
```yaml
name: 🚀 ChatGPT
  type: select
  proxies:
    - 🔰国外流量
    - 这边是对应的节点列表
```

放在任意一个规则下面就好了：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230216180435.png)
然后重载配置文件。

到这一步只是增加好了规则，使用时需要选择对应规则下面的节点，不然不会生效。

第一步，出站模式选择全局代理，同时选择 DIRECT 模式：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230216181132.png)

第二步，找到刚才新增的规则，选择该规则下面的节点即可：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230216181404.png)
再次访问 ChatGPT，页面正常。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230218113649.png)

## 参考链接
* [ChatGPT 关于在中国地区使用的问题汇总](https://github.com/lencx/ChatGPT/issues/83)