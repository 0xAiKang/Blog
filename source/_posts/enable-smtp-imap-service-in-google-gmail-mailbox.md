---
title: 谷歌 Gmail 邮箱启用 SMTP/IMAP 服务
date: 2023-10-19 09:39:40
tags: ["Tutorial"]
categories: ["Tutorial"]
---

Gmail 可以通过其他邮箱进行创建，因此使用 Gmail 发送 E-Mail 的场景还是挺多的，例如有一个可以登录 Gmail 的商业邮箱需要配置 SMTP，那么就可以使用这篇笔记中介绍的方式。

发送之前，这时需要启用一些邮箱配置，才能正常发送。

在启用具体的服务之前，先来了解一下什么是 SMTP/IMAP 服务。

<!-- more -->

## IMAP 与 SMTP
IMAP（Internet Message Access Protocol）和 SMTP（Simple Mail Transfer Protocol）是两种用于电子邮件通信的不同协议，它们在电子邮件的处理和传输方面具有不同的功能和角色。

### IMAP
IMAP 是一种接收邮件协议，用于从邮件服务器上获取电子邮件。IMAP 允许用户从多个设备上访问他们的电子邮件，同时保持邮件服务器上的邮件状态同步。

特点:
* 保持邮件服务器与多个客户端同步
* 允许查看邮件的标题、正文和附件
* 支持文件夹/标签
* 邮件保留在邮件服务器上，直到用户删除它们

### SMTP
SMTP 是一种发送邮件协议，用于将电子邮件从发件人的电子邮件客户端传递到接收人的邮件服务器。SMTP仅用于发送邮件，不用于接收或存储邮件。它是电子邮件系统的邮寄服务。

特点:
* 用于将邮件传递到接收人的邮件服务器
* 不存储邮件，只传递邮件
* 需要发件人的 SMTP 服务器和接收人的 SMTP 服务器之间的通信

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231017181717.png)

## 开启 IMAP

首先登录 Gmail，打开[主面板](https://mail.google.com/)。

先点击小齿轮，然后点击查看所有设置。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231018111519.png)

在 Tab 栏找到『转发和 POP/IMAP』，任何邮箱初始 IMAP 都是关闭的，这里点启用 IMAP 最后保存更改就可以了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231018112005.png)

## 开启 STMP
上面我们已经开启了谷歌的 IMAP 服务，谷歌的邮箱机制是 IMAP 一旦开通，SMTP也就自动开通了，设置里没有看到开启 SMTP 服务的地方没有关系。

到这里还不能直接使用，下一步需要进入[谷歌账号](https://myaccount.google.com/)页面，[获取谷歌应用专用密码](https://myaccount.google.com/apppasswords)。

获取谷歌应用专用密码之前，需要开启两步验证，否则会出现以下提示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231018113733.png)

开启两步验证：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231018113935.png)

开启两步验证之后，进入到两步验证的菜单里，点击 App passwords 或者是直接点击[获取谷歌应用专用密码](https://myaccount.google.com/apppasswords)。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231018114942.png)

下一步开始添加应用密码，输入应用名称，然后点击 Create 即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231018115135.png)

生成的密码类似这样，会代替邮箱的密码，用于邮件发送：
```txt
wwww xxxx yyyy zzzz
```

首次生成后，需复制保存好，如果忘记就只能删除重新生成。

## 配置

### IMAP 配置

* IMAP server address: imap.gmail.com
* IMAP port (SSL): 993

### SMTP 配置

* SMTP server address: smtp.gmail.com
* SMTP name: Your name
* SMTP username: Your Gmail address
* SMTP password: 生成的应用专用密码，实际使用时，需要去除字符之间的空格
* SMTP port (TLS): 587
* SMTP port (SSL): 465

实际使用时，按照上面的配置信息进行填写即可。

## 参考链接
* [电子邮件客户端 (POP/IMAP)](https://support.google.com/mail/topic/7280141)
* [使用应用专用密码登录](https://support.google.com/mail/answer/185833)
* [在使用 POP 的其他电子邮件客户端上查阅 Gmail 邮件](https://support.google.com/mail/answer/7104828)