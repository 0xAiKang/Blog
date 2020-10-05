---
title: >-
  nginx 超时问题——upstream timed out (110: Connection timed out) while reading response header from upstream
date: 2020-10-05 20:28:45
tags: ["Nginx", "PHP"]
categories: ["Nginx"]
---

今天早上起来，发现后台登录不上，打开控制台发现几乎所有请求都超时了。

<!-- more -->

打开nginx 的异常日志可以看到全是相同的异常：

> upstream timed out (110: Connection timed out) while reading response header from upstream

从这个异常日志可以分析出，由于nginx 代理去获取上游服务器的响应超时了，那么究竟是什么原因导致它会超时呢？

通常会导致请求超时可能有以下几个原因：
1. 接口比较复杂，响应时间慢，导致超时。
2. 处理请求的进程异常。
3. 代理服务器与上游服务器的网络问题。

因为请求一直都是那些请求，所以第一种可能性可以排除。
另外子进程数量设置的是比较大，所以第二种应该也可以排除。

对于服务器的网络问题，如果条件允许，可以直接从根本上解决，另外也可以通过设置超时时间来延缓请求超时。

在server 中添加以下配置：
```
large_client_header_buffers 4 16k;
client_max_body_size 30m;
client_body_buffer_size 128k;

proxy_connect_timeout 240s;
proxy_read_timeout 240s;
proxy_send_timeout 240s;
proxy_buffer_size 64k;
proxy_buffers   4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```
然后重启Nginx。

### 参考链接
* [nginx 设置超时时间-Nginx 官网](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout)