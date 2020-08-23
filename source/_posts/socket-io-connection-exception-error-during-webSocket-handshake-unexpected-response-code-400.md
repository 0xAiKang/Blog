---
title: ”Socket.io 连接异常：Error during WebSocket handshake Unexpected response code 400“
date: 2020-08-23 10:34:50
tags: ["Nginx", "Socket.io", "wss"]
categories: ["Socket.io", "Nginx"]
---

前段时间线上的生产环境遇到一个问题：`Error during WebSocket handshake: Unexpected response code: 400`。

起初我没太在意，以为就是正常的 `socket.io` 连接断开了。

直到我发现 `socker.io` 的通讯方式由原来的在一个连接中通讯变成了每一次推送都重起一个请求，我才意识到可能是哪里出问题了。

<!-- more -->

## nginx 作为wbsocket 代理
经过一番查找，了解到 nginx 在作为反向代理时，如果需要使用 `wss`，那么还需要额外加一段配置。

> NGINX supports WebSocket by allowing a tunnel to be set up between a client and a backend server. For NGINX to send the Upgrade request from the client to the backend server, the Upgrade and Connection headers must be set explicitly.  —— Nginx 官网

翻译过来就是：nginx 通过允许在客户端和后端服务器之间建立连接来支持 websocket 通讯，为了使 nginx 将升级请求从客户端发送到后端服务器，必须明确设置 Upgrade 和 Connection 标头。

```
location / {
  proxy_pass http://wsbackend;
  
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "Upgrade";
  proxy_set_header Host $host;
}
```
第一行是 nginx 反向代理的配置，后面四行才是这个问题的解决方案。

仔细想一想，因为本地没有 https 的概念，并没有发现这个问题，而线上是有配置证书的，所以暴露出了这个问题。

### 总结
`socket.io` 的请求并没有真正达到，请求发出之后中间为什么没有到达节点，这个是解决问题的关键。

为了使 nginx 正确处理 `socket.io` 所需要做的就是正确设置标头，以处理将连接从 http 升级到 websocket 的请求。

### 参考链接
* [Nginx 作为Websocket 反向代理](https://www.nginx.com/blog/websocket-nginx/)
