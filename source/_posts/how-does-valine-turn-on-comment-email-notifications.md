---
title: Valine 如何开启评论邮件通知
date: 2021-04-05 20:22:51
tags: ["Hexo"]
categories: ["Hexo"]
---

事情是这样的，昨天无意在博客上看到一条留言，留言时间是两天之前，我才意识到目前的评论系统缺少通知 =_=!。

<!-- more -->

没有通知这怎么能行呢？因为我用的是一款叫做[Valine](https://valine.js.org/) 的评论系统，马上Google 了一下，于是有了这篇笔记。

所以这篇笔记的内容，可能不适用其他评论系统。

--------

`Valine Admin` 是 Valine 评论系统的后端功能补充和增强，主要实现评论邮件通知、评论管理、垃圾评论过滤等功能。支持完全自定义的邮件通知模板，基于Akismet API实现准确的垃圾评论过滤。

在正式开始之前，首先得注册一个[LeanCloud](https://leancloud.cn/) 账号。

> LeanCloud 是什么？

它是一站式后端云服务提供商，到时候我们的评论系统就是要部署在这个云服务上。

## 创建云引擎

注册成功之后，进入控制台，新建一个应用：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404104053.png)

选择开发板就好。

然后进入刚创建好的应用，依次点击设置=> 应用Key，可以看到`AppID` 和`AppKey`，这两个东西很重要，

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404104646.png)

打开博客主题的配置文件，在对应的位置分别填上`appId`和`appKey`：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404105011.png)

下一步需要绑定域名，这里需要绑定的域名，就是你的博客的域名，国内版可能有多个域名绑定供选，这里选择云引擎就好。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404105507.png)

到时候这个域名就是进入我们的评论系统的入口。

需要先完成CNAME 域名解析，绑定才会生效。

进入你的域名管理后台，添加一条`CNAME` 记录，主机名称就是刚才绑定的子域名。

域名解析没那么快，等待的时间可以开始配置云引擎。

进入云引擎=>设置，添加云引擎环境变量：

|变量|示例|说明|
|-|-|-|
|SITE_NAME	|小艾的自留地|	[必填]博客名称|
|SITE_URL|	https://0x2beace.com|	[必填]首页地址|
|SMTP_SERVICE|	QQ|	[新版支持]邮件服务提供商，支持 QQ、163、126、Gmail 以及 更多|
|SMTP_USER|	xxxxxx@qq.com|	[必填]SMTP登录用户|
|SMTP_PASS|	ccxxxxxxxxch|	[必填]SMTP登录密码|
|SENDER_NAME	|Boo|	[必填]发件人|
|SENDER_EMAIL|	xxxxxx@qq.com	|[必填]发件邮箱|
|ADMIN_URL|	https://xxx.0x2beace.com/|	[建议]Web主机二级域名（云引擎域名），用于自动唤醒|
|BLOGGER_EMAIL|	xxxxx@gmail.com	| [可选]博主通知收件地址，默认使用SENDER_EMAIL|
|AKISMET_KEY|	xxxxxxxx|	[可选]Akismet Key 用于垃圾评论检测，设为MANUAL_REVIEW开启人工审核，留空不使用反垃圾|

点击保存之后，切换到云引擎=>部署，部署模式选择部署项目-Git部署，分支master，手动部署目标环境为生产环境，Git 仓库填入：`https://github.com/DesertsP/Valine-Admin.git`，点击部署即可。


## 评论管理

### 注册管理员

如果这时域名解析已经完成，那么访问：`https://云引擎域名/`，应该可以看到如下界面：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404111837.png)

这是还没有管理员账号，需要先通过`https://云引擎域名/sign-up/`注册一个。

至此就已经可以管理我们的评论了，但是目前还没有邮件通知。

## 设置定时任务
这里设置定时任务的目的就是，每天定时检查是否存在漏发的邮件。

进入云引擎=> 定时任务，创建两个定时任务：
1. 选择self-wake云函数，Cron表达式为`0 */30 0-16 * * ?`，表示每天早0点到晚16点每隔30分钟访问云引擎。
2. 选择resend-mails云函数，Cron表达式为`0 0 0 * * ?`，表示每天0点检查过去24小时内漏发的通知邮件并补发。

## 邮件通知模版(可选配置)

邮件通知模板在云引擎环境变量中设定，可自定义通知邮件标题及内容模板。

|环境变量|	示例|	说明|
|-|-|-|
|MAIL_SUBJECT|	`${PARENT_NICK}`，您在${SITE_NAME}上的评论收到了回复|	[可选]@通知邮件主题（标题）模板
|MAIL_TEMPLATE|	见下文|	[可选]@通知邮件内容模板|
|MAIL_SUBJECT_ADMIN	| ${SITE_NAME}上有新评论了|	[可选]博主邮件通知主题模板|
|MAIL_TEMPLATE_ADMIN|见下文|	[可选]博主邮件通知内容模板|

邮件通知包含两种，分别是被@通知和博主通知，这两种模板都可以完全自定义。默认使用经典的蓝色风格模板（样式来源未知）。

默认被@通知邮件内容模板如下：
```
<div style="border-top:2px solid #12ADDB;box-shadow:0 1px 3px #AAAAAA;line-height:180%;padding:0 15px 12px;margin:50px auto;font-size:12px;"><h2 style="border-bottom:1px solid #DDD;font-size:14px;font-weight:normal;padding:13px 0 10px 8px;">您在<a style="text-decoration:none;color: #12ADDB;" href="${SITE_URL}" target="_blank">            ${SITE_NAME}</a>上的评论有了新的回复</h2> ${PARENT_NICK} 同学，您曾发表评论：<div style="padding:0 12px 0 12px;margin-top:18px"><div style="background-color: #f5f5f5;padding: 10px 15px;margin:18px 0;word-wrap:break-word;">            ${PARENT_COMMENT}</div><p><strong>${NICK}</strong>回复说：</p><div style="background-color: #f5f5f5;padding: 10px 15px;margin:18px 0;word-wrap:break-word;"> ${COMMENT}</div><p>您可以点击<a style="text-decoration:none; color:#12addb" href="${POST_URL}" target="_blank">查看回复的完整內容</a>，欢迎再次光临<a style="text-decoration:none; color:#12addb" href="${SITE_URL}" target="_blank">${SITE_NAME}</a>。<br></p></div></div>
```

默认博主通知邮件内容模板如下：
```
<div style="border-top:2px solid #12ADDB;box-shadow:0 1px 3px #AAAAAA;line-height:180%;padding:0 15px 12px;margin:50px auto;font-size:12px;"><h2 style="border-bottom:1px solid #DDD;font-size:14px;font-weight:normal;padding:13px 0 10px 8px;">您在<a style="text-decoration:none;color: #12ADDB;" href="${SITE_URL}" target="_blank">${SITE_NAME}</a>上的文章有了新的评论</h2><p><strong>${NICK}</strong>回复说：</p><div style="background-color: #f5f5f5;padding: 10px 15px;margin:18px 0;word-wrap:break-word;"> ${COMMENT}</div><p>您可以点击<a style="text-decoration:none; color:#12addb" href="${POST_URL}" target="_blank">查看回复的完整內容</a><br></p></div></div>
```
这里有个问题就是部分变量不再可用，如果使用了未定义的变量，发送邮件时会抛出异常：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404113532.png)

我选择去掉了部分变量，这就导致了邮件部分内容是缺失的：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210404113944.png)

这里没继续往下深究了，能用就行，至此就完成了所有配置。如果你遇到一些奇怪的问题，可以看看以下建议对你是否有用：

> 常见问题

1. LeanCloud 分国内版和国际版，如果你和我一样不喜欢域名备案，使用的是国际域名服务商提供的域名，那么注册LeanCloud 时，请选选择国际版。
2. 域名解析如果长时间未生效，请检查添加`CNAME` 纪录，`ttl` 不要选择一小时，选择六百秒。
3. `SMTP_PASS` 不是QQ 邮箱的密码，而是`SMTP服务`的密钥，如果不知道如何获取，可以看[这里](https://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=331)。
4. 修改完变量，需要重启应用，否者不会生效。

## 参考链接
* [Valine Admin](https://github.com/DesertsP/Valine-Admin)
* [Valine Admin 配置手册](https://deserts.io/valine-admin-document/)