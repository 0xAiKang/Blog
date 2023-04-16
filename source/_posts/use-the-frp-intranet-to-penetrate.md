---
title: 使用 frp 内网穿透
date: 2023-04-16 15:22:09
tags: ["Tutorial"]
categories: ["Tutorial"]
---

如何使用 frp 进行内网穿透？

<!-- more -->

## 为什么需要内网穿透
我们通常在局域网内部署服务器和应用，当需要将本地服务提供到互联网外网连接访问时，由于本地服务器本身并无公网IP，就无法实现，这时候就需要内网穿透技术。

想要通过公网访问内网的服务，除了内网穿透技术，还有其他方案可以实现，其他方案不在本文讨论范围，这篇笔记只介绍 frp 内网穿透。

## frp 是什么
[frp](https://github.com/fatedier/frp) 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

这篇笔记只讨论 HTTP 和 TCP 相关的内容。

## 准备工作

在正式开始之前，需要做好以下准备工作：
1. 一台公网可以访问的主机（作为服务端）
2. 一台内网可以访问的主机（作为客户端）
3. 一个可以解析的域名

## 下载/安装 frp
frp 的下载安装非常简单，直接去到[发布页](https://github.com/fatedier/frp/releases)，根据服务端/客户端主机系统架构，选择合适的版本，直接下载解压即可。

解压完成之后，文件列表很简单：
* frpc：客户端可执行文件
* frpc.ini：客户端对应配置文件
* frpc_full.ini：客户端对应示例配置文件
* frps：服务端可执行文件
* frps.ini：服务端对应配置文件
* frps_full.ini：服务端对应示例配置文件

frp 主要由 客户端(frpc) 和 服务端(frps) 组成，服务端通常部署在具有公网 IP 的机器上，客户端通常部署在需要穿透的内网服务所在的机器上。

### 运行

前台运行：
```bash
$ ./frpc -c frpc.ini
```

后台运行：
```bash
$ nohup ./frpc -c frpc.ini &> run.log &
```

## 示例
frp 的原理是，用户通过访问服务端的 frps，由 frp 负责根据请求的端口或其他信息将请求路由到对应的内网机器，从而实现通信。

因此端口的访问必须顺畅，为了减少不必要的障碍，建议提前添加好防火墙端口入站规则（云服务器控制台以及主机自身防火墙）。

### 通过自定义域名访问内网 Web 服务

这个示例通过简单配置 HTTP 类型的代理让用户访问到内网的 Web 服务。

编辑 `frps.ini` 配置文件：
```ini
[common]
bind_port = 7000
vhost_http_port = 6010
token = my_token

dashboard_user = admin
dashboard_pwd = password
dashboard_port = 7500
```

配置说明：
* bind_port：指定接收客户端连接的端口，需要和客户端配置保持一致
* vhost_http_port：设置监听 HTTP 请求端口
* token：用于客户端和服务端连接的口令，需要和客户端配置保持一致
* dashboard_user：服务端控制面板账号
* dashboard_pwd：服务端控制面板密码
* dashboard_port：服务端控制面板所占用端口

编辑 `frpc.ini` 配置文件：
```ini
[common]
server_addr = x.x.x.x
server_port = 7000
token = my_token

[web]
type = http
local_port = 80
custom_domains = www.yourdomain.com
```

配置说明：
* server_addr：`x.x.x.x` 是 frps 所在的外网服务器的 IP
* server_port：指定接收客户端连接的端口，需要和服务端配置保持一致
* token：用于客户端和服务端连接的口令，需要和服务端保持一致
* type：指定协议类型
* local_port：内网机器上的Web 服务监听的端口
* custom_domains：绑定的自定义域名（解析到外网IP）

> 说明：`[web]` 这个配置项是自定义的，不是一定要定义成 `[web]`，也可以是 `[web1]`、`[web2]`，它只是一个配置项的名称。

配置文件编写完成之后，接下来就可以分别启动 `frps`、`frpc`了。

顺利的话，打开 frps 的控制面板，即可看到，http proxies 列表出现了一个名为 web 的记录。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230415131154.png)
通过浏览器访问 `http://www.yourdomain.com:6010`，即可访问到处于内网机器上 `80` 端口的服务。


### 通过 ssh 访问内网机器

## 参考链接
* [frp 官方文档](https://gofrp.org/docs/concepts/)