---
title: 记一次由 Nginx fastcgi_temp 权限不足引起的问题
date: 2021-08-04 23:13:23
tags: ["Nginx", "PHP-FPM", "PHP"]
categories: ["Nginx"]
---

今天在正式环境收到同事反馈的一个问题：主页可以打开，但是部分列表始终无法访问。

<!-- more -->

一开始，我以为是接口抛出异常了，但是打开浏览器控制台看到响应 状态为“200 OK”，同时可以看到`Console` 下有一个`error`。

异常信息为：`NET::ERR_INCOMPLETE_CHUNKED_ENCODING 200 (OK)`。

## 解决过程

由Http Status Code 可以判断出，这个错误跟接口没有关系，接口如果有问题，状态码一定不会是 200 。

既然和接口没有关系，那就极可能是服务端环境导致的问题了。

 想明白这一点之后，马上查看Nginx 近期的错误日志。
 
```
2021/08/04 14:59:37 [crit] 11712#0: *244402125 open() "/www/server/nginx/fastcgi_temp/0/94/0000005940" failed (13: Permission denied) while reading upstream, client: 116.24.95.208, server: api.nwppm.com, request: "POST /xxxx/xxxx/index HTTP/1.1", upstream: "fastcgi://unix:/tmp/php-cgi-74.sock:", host: "api.nwppm.com", referrer: "http://xxxx.xxxx.com/"
```

果然没猜错，就是因为文件夹权限不足引起的异常。

## 解决方案
找到原因后就好办了，既然是因为`factcgi_temp` 文件夹权限不足引起的，那么解决方案就是更新权限：

```bash
chmod -R 740 /www/server/nginx/fastcgi_temp
```

##  问题还原
问题虽然解决了，但还不知道是什么原因造成的，如果就此放下，估计今晚都会睡不着觉的...

在搞清楚这个问题之前，先来回顾一下`PHP-FPM` 与 Nginx 的协程流程：
1. 当客户端发起一个请求到服务端，Nginx 首先会判断该请求是静态还是动态？
2. 如果是静态，直接返回对应的静态资源。
3. 如果是动态，FastCGI 会将该请求转发给本地 `9000` 端口（9000 是 PHP—FPM 所监听的端口）或者 sock。
4. PHP-FPM 主进程接收到请求之后，
会分配一个空闲的 Worker 进程去处理这个请求，处理完成之后将数据返回给 FastCGI，再由 Nginx 返回给客户端。

以上是一个完整的HTTP 请求处理流程。其中Worker 进程处理完请求之后，会将数据返回给 FastCGI，再由Nginx 返回给客户端，这里就不得不提到Nginx 的Buffer 机制：

> 对于来自 FastCGI Server 的 Response，Nginx 将其缓冲到内存中，然后依次发送到客户端浏览器。缓冲区的大小由 fastcgi_buffers 和 fastcgi_buffer_size 两个值控制。

以下面这个配置进行说明：

```
fastcgi_buffers      8 4K;
fastcgi_buffer_size  4K;
```

* `fastcgi_buffers` 控制 nginx 最多创建 8 个大小为 4K 的缓冲区
* `fastcgi_buffer_size` 则是处理 Response 时第一个缓冲区的大小，不包含在前者中

所以总计能创建的最大内存缓冲区大小是 8*4K+4K = 36k。而这些缓冲区是根据实际的 Response 大小动态生成的，并不是一次性创建的。比如一个 8K 的页面，Nginx 会创建 2*4K 共 2 个 buffers。

当 Response 小于等于 36k 时，所有数据当然全部在内存中处理。如果 Response 大于 36k 呢？`fastcgi_temp` 的作用就在于此，多出来的数据会被临时写入到文件中，放在这个目录
下面。

## 参考链接
- [分析 fastcgi_temp 错误以及 Nginx 的 Buffer 机制](https://blog.csdn.net/crx05/article/details/70210323)