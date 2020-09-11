---
title: 如何自动申请免费的SSL 证书
date: 2020-09-11 16:18:20
tags: ["SSL", "Certbot", "HTTPS"]
categories: ["Tutorial"]
---

上次介绍了如何通过第三方网站申请免费的SSL 证书，但有效期只有三个月，三个月之后又需要再次申请，记得还好，如果忘了可能还会造成不必要的损失。

<!-- more -->

[Let's Encrypt](https://letsencrypt.org/) 是一个免费提供的SSL 证书的CA，虽然每次签发的有效期都只有三个月，但是发证是自动化的，发证速度较快，并且可以通过脚本来自动续签，为个人网站使用HTTPS提供了一个不错的选择。

Let’s Encrypt （以下简称LE）的证书签发主要使用基于 ACME协议 的证书自动管理客户端来实现。

LE官方推荐的客户端是 [Certbot](https://certbot.eff.org/) ，本文中就是使用 Certbot 来获取和续签证书。

## LE 是如何自动签发证书的
假设现在要申请CA 证书的域名是 `example.com`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200911120519.png)

首先由WebServer（也就是我们用户端的服务器）的管理客户端（如Certbot）发送请求到LE，让LE来验证客户端是否真的控制example.com这个域名，接下来LE会提出一些验证动作（原文challenges），比如让客户端在一个很明显的路径上放指定的文件。同时，LE还会发出一个随机数，客户端需要用这个随机数和客户端自己的私钥来进行签名。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200911120904.png)

WebServer上的客户端完成LE指定的域名验证动作并且将加密后的签名后，再次发送请求到LE要求验证，LE会验证发回来的签名是否正确，并且验证域名验证动作是否完成，如下载指定的文件并且判断文件里面的内容是否符合要求。

这些验证都完成以后，可以申请证书了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200911120943.png)

完成验证后，客户端生成自己的私钥以及 [Certificate Signing Request（CSR）](https://tools.ietf.org/html/rfc2986) 发送到LE服务器，LE服务器会将CA证书（也是公钥）发放到你的服务器。

这样就完成了CA证书的自动化发放了。

### 使用Certbot 获取证书
LE 的CA 证书发放原理看着还挺麻烦的，但如果使用 Certbot 客户端，整个过程还是挺简单的。

在正式获取证书之前，推荐先去[Certbot 官网](https://certbot.eff.org/instructions)选择适合自己的系统环境。

我这边系统环境是`Nginx` + `Ubuntu 18.04 LTS`，所以下面介绍的安装流程只适用于**Ubuntu + Nginx**。

#### 1. 安装 snap

[snap](https://zh.wikipedia.org/zh-hans/Snappy_(%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8)) 是Canonical公司发布的全新的软件包管理方式，它类似一个容器拥有一个应用程序所有的文件和库，各个应用程序之间完全独立。使用snap 包的好处就是它解决了应用程序之间的依赖问题，使应用程序之间更容易管理。但是由此带来的问题就是它占用更多的磁盘空间。

```
$ sudo apt update
$ sudo apt install snapd
```

#### 2. 安装 certbot
在安装 Certbot 之前，最好先移除历史快照。

```
$ sudo apt-get remove certbot
```

进行安装：
```
$ sudo snap install --classic certbot
```

#### 3. 生成证书
安装完成之后，下一步需要做的就是生成证书了，这里有两种方式：
1. 生成证书并自动配置
```
$ sudo certbot --nginx
```

2. 生成证书手动配置

```
$ sudo certbot certonly --nginx
```

我选择的是手动配置，大概流程如下：
1. 输入常用邮箱，用来接收通知和恢复密钥。
2. 同意使用协议。
3. 输入需要做授权的域名，多个域名用空格隔开。
4. 等待验证通过。

如果其中某个域名验证失败，则不会生成密码。

一切正常的话，可以看到`/etc/letsencrypt/live/your_sites/`目录下多了四个文件：
* `cert.pem` ： 公钥，服务器证书
* `chain.pem` ： 中间证书
* `fullchain.pem` ： 前两个的合集
* `privkey.pem` ： 私钥

其中配置Nginx SSL 只需要用到`fullchain.pem` 和`privkey.pem`：
```
server {
    listen       443 ssl;
    server_name www.example.com;

    ssl on;
    ssl_certificate /etc/letsencrypt/live/www.exampl.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.example.com/privkey.pem;
    ...
}
```

至此，就已经完成了生成证书到配置的全部过程了。

### 自动续签
如果快要到期了，可以使用`certbot renew`对证书进行更新，需要注意的是，如果证书尚未过期，则不会更新。

可以配合`conrtab`使用，每半个月的凌晨三点自动续签一次。
```
$ 0 3 15 * * certbot renew 
```

### 参考链接
* [Let’s Encrypt免费SSL证书获取以及自动续签](http://blog.cngal.org/index.php?controller=post&action=view&id_post=10)