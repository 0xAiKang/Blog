---
title: 记录一次运营商劫持 DNS
date: 2022-08-20 10:15:05
tags: ["一些经验"]
categories: ["一些经验"]
---

最近手机上访问 `github.com`，直接就是打不开了，起初并没有多想，可能只是网络较差吧。

<!-- more -->

知道最近写博客时，需要把图片上传到图床上，但是失败了。

错误日志如下：
```
RequestError: Error: connect ECONNREFUSED 127.0.0.1:443
    at new RequestError (/Applications/PicGo.app/Contents/Resources/app.asar/node_modules/request-promise-core/lib/errors.js:14:15)
    at Request.plumbing.callback (/Applications/PicGo.app/Contents/Resources/app.asar/node_modules/request-promise-core/lib/plumbing.js:87:29)
    at Request.RP$callback [as _callback] (/Applications/PicGo.app/Contents/Resources/app.asar/node_modules/request-promise-core/lib/plumbing.js:46:31)
    at self.callback (/Applications/PicGo.app/Contents/Resources/app.asar/node_modules/request/request.js:185:22)
    at Request.emit (events.js:200:13)
    at Request.onRequestError (/Applications/PicGo.app/Contents/Resources/app.asar/node_modules/request/request.js:877:8)
    at ClientRequest.emit (events.js:200:13)
    at TLSSocket.socketErrorListener (_http_client.js:402:9)
    at TLSSocket.emit (events.js:200:13)
    at emitErrorNT (internal/streams/destroy.js:91:8)
```

看到这个日志之后，我的第一反应是，怎么请求的是 `127.0.0.1` 呢？

然后在本地 ping 了一下 `github.com`：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220820101406.png)
咦，返回的地址竟然是 `127.0.0.1`，我还以为是设置了本地域名的关系，结果看了一下，并没有`github.com` 这个域名。

于是通过站长工具的 DNS 查询功能进行查询，结果如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220820101422.png)
广东电信，竟然把 DNS 给劫持了，这个 `127.0.0.1` 显然是运营商解析的。

于是将正确的地址，配置在 hosts 之后，再次通过图床进行上传，就可以了。