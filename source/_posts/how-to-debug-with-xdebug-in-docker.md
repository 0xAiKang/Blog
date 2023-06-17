---
title: 如何在 Docker 中使用 Xdebug 进行调试
date: 2023-06-17 11:18:12
tags: ["PHP", "Xdebug"]
categories: ["PHP"]
---

关于如何使用 Xdebug 在本地进行调试，在之前的[这篇笔记](https://www.0x2beace.com/use-phpstorm-to-configure-xdebug-under-windows-and-mac/)中，已经详细介绍过了。

而这篇笔记要介绍的是，如何在 Docker 中使用 Xdebug 进行调试。

<!-- more -->

有两种方式，一种是通过主动开启 Debug 监听，另一种则是配置环境变量。

## ini 配置
调试之前需要先安装好 Xdebug 扩展，这里就不过多介绍如何安装了。

编辑 `php.ini` 配置文件，增加以下配置内容：
```ini
xdebug.mode=develop,debug
xdebug.remote_enable = 1
xdebug.remote_port=9003
xdebug.idekey=ClientHost
xdebug.client_host=host.docker.internal
```

* `xdebug.remote_enable`：开启远程调试
* `xdebug.remote_port`：调试监听的端口
* `xdebug.idekey`：Xdebug idekey，需要与 PHPStorm 里的配置保持一致
* `xdebug.client_host`：宿主机地址

## PHPStorm 配置

添加远程调试：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111449.png)

Server 配置：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111501.png)

注意：需要勾选使用路径映射，然后下面对应项目在 Docker 容器中的路径。

使用 PHPStorm 的Web 服务器调试工具验证一下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111514.png)

如果能看到以上输出，则可以开始调试了。

## 方式一

开启调试之前，需要先点击右上角的 Debug 按钮，开启调试监听。

正常即可看到，下方的调试器，正在等待 ide key 传入连接。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111528.png)
如果看到的不是等待 ide key 传入连接，通常是因为路径映射有问题，检查一下

如果传入请求的 `XDEBUG_SESSION_START` 参数的值，正好是自定义的 ide key，断点便会进入。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111539.png)

使用这种方式进行调试，一定记得打开调试，否则断点不会进入。

## 方式二
下面这种方式，需要在容器中配置环境变量：

```bash
PHP_IDE_CONFIG='serverName=ClientHost'
```

环境变量中，指定需要使用的 Server Name。

然后在需要调试的地方，加上断点即可，无需点击 Debug 按钮。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111550.png)

如果没有配置环境变量，则会提示如下信息：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230617111617.png)