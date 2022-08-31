---
title: 域名解析不生效有哪些原因
date: 2022-05-15 16:26:19
tags: ["一些经验"]
categories: ["一些经验"]
---

域名解析不生效的表现是使用ping命令无法获取正确的域名解析IP地址。

<!-- more -->

解析不生效的原因包括:
* 本地网络故障
* 云解析服务器的解析记录异常
* 域名解析记录在DNS被修改或者缓存
* 域名未通过实名认证

以域名 example.com 为例，排除解析不生效可采用如下流程：

## 1. 检查本地网络是否正常
ping其他域名，检查域名解析是否生效?
  
- 若生效，则排除本地网络问题。
- 若不生效，则表示本地网络故障，请联系宽带运营商解决网络故障问题。

## 2. 检查域名解析是否生效

打开终端，执行以下命令：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220514173049.png)
验证NS类型解析：用于指定解析服务商的 DNS 地址。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220514173539.png)
查询指定权威DNS的域名解析是否生效。

- 如果命令执行结果显示解析不生效，这时可以登录对应的云解析服务管理控制台，检查解析记录是否异常
- 如果命令执行结果显示解析生效，表示域名解析在云解析服务器正常

## 3. 检查域名解析记录是否被修改或者缓存

1. 检查域名是否修改过DNS服务器\
   修改DNS服务器需要24小时~48小时生效
3. 检查域名记录是否被本地电脑缓存
    - Windows操作系统:执行ipconfig /flushdns命令刷新DNS缓存
    - Linux/Unix操作系统:不会缓存DNS解析记录。 如果安装了nscd缓存服务，执行service nscd restart重启服务刷新缓存。
4. 检查运营商提供的本地 DNS 服务器是否缓存了解析记录\
   域名解析记录的缓存时间通常在一个小时之内，之后重新使用ping命令检查 解析是否效。
5. 检查本地DNS是否被劫持，解析记录是否被修改\
   执行`dig example.com@8.8.8.8`或者 `dig example.com@114.114.114.114`命令，检查公共DNS解析是否生效，建议把本地dns改成公共dns。

## 4. 检查域名是否完成实名认证

如果域名未进行实名认证，则域名会被注册局会暂停解析，解析不生效。更多阅读为什么域名解析成功但网站仍然无法访问？

## 参考链接
* [解析不生效有哪些原因？](https://support.huaweicloud.com/dns_faq/dns_faq_003.html)