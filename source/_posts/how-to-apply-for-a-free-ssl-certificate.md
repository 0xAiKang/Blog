---
title: 如何申请免费的SSL 证书
date: 2020-08-14 18:03:13
tags: ["HTTPS", "SSL", "HTTP"]
categories: ["Tutorial"]
---

这篇笔记用来记录如何申请免费的 SSL 证书，通过本文介绍的方式所申请的证书有效期只有三个月，请谨慎选择。

<!-- more -->

## 准备
像这类提供免费 SSL 证书的网站非常多，这里我选择的平台是 [FreeSSL.cn](https://freessl.cn/) 。

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

## 后记

因为 FreeSSL.cn 后面改版了，之前是直接生成对应的 `*.key` 和 `*.pem` 文件，现在则变成了 ACME 自动化申请。

前两步还是一样的，只是到了第三步，不一样。

现在需要选择一种部署方式，来获取证书：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230228143014.png)

Cerbot 的部署方式其他笔记已经提到了，因此这里介绍 `acme.sh` 的方式。

首先需要安装 `acme.sh`，可以直接使用下面的命令进行安装：
```bash
curl https://get.acme.sh | sh -s email=my@example.com
```

如果上面官方下载地址失败 或者 太慢，可以选用国内的备用地址：
```bash
curl https://gitcode.net/cert/cn-acme.sh/-/raw/master/install.sh?inline=false | sh -s email=my@example.com
```

安装完成之后，当前目录下会多出一个 `.acme.sh` 文件，查看该文件夹：
```bash
[root@localhost]$ ls -al
total 248
drwx------ 8 ec2-user ec2-user    191 Feb 28 14:20 .
drwx------ 7 ec2-user ec2-user    335 Feb 28 12:34 ..
-rw-rw-r-- 1 ec2-user ec2-user    305 Feb 28 14:21 account.conf
-rwxrwxr-x 1 ec2-user ec2-user 221245 Feb 28 12:29 acme.sh
-rw-rw-r-- 1 ec2-user ec2-user     96 Feb 28 12:29 acme.sh.env
drwxrwxr-x 3 ec2-user ec2-user     29 Feb 28 12:34 ca
drwxrwxr-x 2 ec2-user ec2-user   4096 Feb 28 12:29 deploy
drwxrwxr-x 2 ec2-user ec2-user   4096 Feb 28 12:29 dnsapi
-rw-rw-r-- 1 ec2-user ec2-user    460 Feb 28 14:21 http.header
drwxrwxr-x 2 ec2-user ec2-user   4096 Feb 28 12:29 notify
```

其中 `.acme.sh` 就是一会需要用到的 shell 脚本。

获取证书：
```bash
acme.sh --issue -d my@example.com  --dns dns_dp --server https://acme.freessl.cn/v2/DV90/directory/z8s58p0gs74t0839d3mu
```

将示例命令中的域名替换成自己的域名即可。

正常执行完成之后，当前文件夹下，会多出一个以域名为名的文件：
* fullchain.cer：证书（域名证书.crt + 根证书(root_bundle).crt）
* my.example.com.key：密钥

有了这两个文件之后，就可以轻松配置 ssl 了。