---
title: 手把手教你如何创建启动 Azure 实例
date: 2020-08-12 23:49:19
tags: ["Linux", "云", "Tutorial"]
categories: ["Linux", "Tutorial"]
---

这篇笔记用来整理如何创建启用 Azure 实例。因为这方面可以找到的资料比较少，所以整理一下。

一是方便自己以后回顾，二是给其他人作为参考。

<!-- more -->

## 准备
因为本文是创建微软云，所以首先你得有一个微软账号。

打开 [Microsoft Azure](https://azure.microsoft.com/zh-cn/) 进行登录，登录成功之后，进入[云服务管理后台](https://portal.azure.com/#home)。

## 创建实例

点击创建资源。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812135350.png)

可以搜索你想创建的云服务类型，这里我选择的是 `Ubuntu Server 18.04 LTS`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812135625.png)

点击创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812135731.png)

放心，这里的创建并不是正真意义上的创建。接下来需要为机器预设配置。

下面对常见的配置进行简单说明：
* `资源组`：用来分配一些权限以及策略。
* `虚拟机名称`：你希望用什么名称来称呼这台机器（通常是英文）
* `区域`：选择机器所在地区
* `映像`：选择操作系统
* `大小`：选择一个合适的负责类型，可以理解成机器的硬件配置。
* `身份验证类型`：通常有两种：ssh 密钥和密码，强烈建议使用密钥而不使用密码（密哦存在被暴力破解的风险）。
* `用户名`：微软云默认没有给`root` 用户，这里需要指定用户名称。
* `公共入站端口`：通常是只开启`HTTP (80)`、`HTTPS (443)`、`SSH (22)` 。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812141205.png)

完成基本配置之后，点击`下一步：磁盘`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812141834.png)

Azure 默认只有一个用于短期存储的临时盘，而临时盘通常都很小。

默认的磁盘很小，如果想扩大有两种方式：
* 创建新的磁盘，需要手动挂载。
* 更改默认磁盘的大小。

配置完磁盘之后，点击`下一步：网络`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812142034.png)

网络配置，公用ip 可以选择无，后面再去新建。

然后点击`下一步：管理`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812142317.png)

管理、高级、标记这一块，如果没有特殊需求可以直接使用默认配置。

最后点击`查看+创建`，可以看到预设的配置信息，如果符合预期，点击创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812142806.png)

下载私钥并保存好。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812152220.png)

此时，虽然已经创建好虚拟机，但是还不能直接使用，因为没有配置IP。

## 关联IP
Azure 和 AWS 不同，它并没有弹性IP 的概念，如果需要配置IP，需要在搜索栏中搜索`公共IP地址`，

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812143323.png)

点击第一个搜索结果。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812143437.png)

点击添加。

配置IP 基本信息，然后点击创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812143641.png)

此时，只是创建了内网IP，并没有与外网IP 地址进行关联，

点击刚才新建的公共 IP 地址，点击配置。

资源类型选择网络接口，网络接口与对应的实例进行关联。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812153504.png)

关联成功之后，就可以进行连接了。

### 连接
1. 打开终端
2. 请确保你对私钥具有只读访问权限。

```
chmod 400 <私钥>
```
3. 运行以下示例命令以连接到 VM。

```
ssh -i <私钥路径> user@ip_address
```
* `user`：表示VM 用户
* `ip_address`：表示外网IP 地址

### 扩大默认磁盘大小
上面简单提到过，如果想要扩大默认磁盘的大小，有两种方式：
1. 添加新磁盘。这种方式需要手动挂载，如果对linux 并不熟悉，这种方式不推荐新手用户使用。
2. 更改默认磁盘大小。

第二种方式并不能直接更改，需要先将服务器停掉（注意⚠️：不是删除）。

搜索磁盘，点击第一个搜索结果。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812145110.png)

点击需要扩大的磁盘实例，注意：只能扩大，不能缩小。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200812145326.png)

然后点击保存即可。

### 总结
至此，就已经完成了Azure 的创建了，这方面需要学习的还有很多，这里只是简单的整理了一下自己遇到的问题。

有些地方可能没说清楚，但如果能帮到你那真是太好了
