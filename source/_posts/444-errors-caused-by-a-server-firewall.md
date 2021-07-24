---
title: 记一次服务端防火墙引起的 444 错误
date: 2021-07-24 16:01:07
tags: ["防火墙"]
categories: ["防火墙"]
---

这周接手一个需求，需要做一个Http“代理”，将请求转发至不同的场景。

<!-- more -->

本来不是多复杂的需求，实现起来也没花多久，反而是在部署测试上面，耗费不少时间。

本地测试一切正常之后，变将代码部署至服务端，因为服务器使用宝塔，部署过程也很快。

可是问题就出在了，测试过程中。

起初我只是察觉，代理服务端收到请求之后，一直没有将请求转发出去，于是查看请求日志，发现满屏的`http 444`。

这个时候，我只是比较困惑，本地发送相同的请求都是正常的，怎么来自客户端的请求就有问题了。

于是，查了一下这个 [444](https://httpstatuses.com/444) 状态码。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210724152825.png)

它实际上并不是一个标准的 HTTP Status Codes，而是Nginx 自定义的一个Code，通常用于**服务端没有返回信息给客户端并且关闭了连接**的场景。

> 服务端的Nginx 为什么会关闭连接？通常由以下两类原因引起：

1. 客户端发送恶意的请求
2. 客户端发送格式错误的请求

想清楚这两点之后，我对比了一下本地发送的请求，与客户端发送的请求之间的差异：

本地：
```bash
curl --location --request POST 'http://xxx.xxx.com/route' \
--header 'User-Agent: PostmanRuntime/7.28.2' \
--form 'params=xxxx' \
--form 'params=xxxx'
```

客户端：
```bash
curl --location --request POST 'http://xxx.xxx.com/route' \
--header 'User-Agent: Apache-HttpClient/4.4.1' \
--form 'params=xxxx' \
--form 'params=xxxx'
```

除了`User-Agent` 不同，其余信息均一致。

为了验证我的猜想，打开宝塔的Nginx 防火墙站点日志，果然，全部被拦截了：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210724154555.png)

至于为什么UA 中携带了`Apache-HttpClient/4.4.1` 这几个关键字就被拦截了，可以通过宝塔默认的`User-Agent过滤` 规则中找到答案：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210724155612.png)

可以很清晰地看到，关键词过滤规则中，有一个`Apache-HttpClient`。

知道原因之后就好办了，将关键词过滤的开关给关了，服务即可正常访问。