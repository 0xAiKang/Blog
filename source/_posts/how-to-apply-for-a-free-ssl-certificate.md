---
title: 如何申请免费的SSL 证书
date: 2020-08-14 18:03:13
tags: ["HTTPS", "SSL", "HTTP"]
categories: ["Tutorial"]
---

这篇笔记用来记录如何申请免费的 SSL 证书，通过本文介绍的方式所申请的证书有效期只有三个月，请谨慎选择。

<!-- more -->

## 准备
像这类提供免费 SSL 证书的网站非常多，这里我选择的平台是 [FreeSSL.cn](https://freessl.cn/apply?domains=8188.com%252C*.8188.com&product=letsencrypt02&from=) 。

在正式开始之前，你得准备一个邮箱，[注册](https://freessl.cn/register) 一个 `FreeSSL.cn` 账号，然后登录。

将需要申请证书的域名填写在输入框中，选择多域名通配符，然后点击创建免费的SSL 证书。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200814172918.png)

我这里选择的是泛域名，根据你自己的实际情况，去创建相应子域名的证书：
* `example.com`：主域名
* `*.example.com`：泛域名

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200814174731.png)

选择浏览器生成。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200814175011.png)

点击确认创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200814174930.png)

### 添加TXT 记录

打开需要申请 SSL 证书的域名管理后台，找到 DNS 管理。

添加 TXT 验证，将刚才的记录值与TXT 记录添加到对应的TXT 类型。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200814180141.png)

注意⚠️：记录值区分大小写。

检测是否配置成功。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200814175520.png)

在完成验证之前不要离开当前页面，验证成功之后，点击验证。

如果配置成功没问题，就可以点击验证，下载证书就完成了。

注意⚠️：使用此方式获取的证书，有效期只有三个月。