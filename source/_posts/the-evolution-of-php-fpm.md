---
title: PHP-FPM 进化史
date: 2020-12-02 21:42:16
tags: ["PHP", "PHP-FPM"]
categories: ["PHP"]
---

最近有幸读到[一篇文章](http://blog.leanote.com/post/weibo-007/%E4%BB%8ECGI%E5%88%B0FastCGI%E5%88%B0PHP-FPM)，一文将CGI 的进化史讲的特别详细，虽然我自己之前也整理过 [CGI、FastCGI、PHP-FPM 相关的笔记](https://www.0x2beace.com/what-is-the-relationship-between-php-fpm-and-nginx/)，但是并没有从原理的角度来认识 CGI。

<!-- more -->

## CGI 的诞生
早些年的Web 应用很简单，客户端通过浏览器发起请求，服务端直接返回响应。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201202210831.png)

随着互联网的发展，简单的Web 应用已经不能满足开发者们了。
我们希望Web服务器有更多的功能，飞速发展的同时还能让不同语言的开发者也能加入。

[CGI协议](https://www.ietf.org/rfc/rfc3875)协议的诞生就是 Web服务器和其他领域的开发者在保证遵守协议的基础上，剩下的可以自由发挥，而实现这个协议的脚本叫做CGI 程序。

CGI协议规定了需要向CGI脚本设置的环境变量和一些其他信息，CGI程序完成某一个功能，可以用PHP，Python，Shell或者C语言编写。

在没有CGI 之前，其他语言如果需要接入Mysql 或者Memcache，还需要使用C 语言，但有了CGI协议，我们的Web处理流程可以变成下图这样：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201202211748.png)

## FastCGI 的诞生
CGI程序存在致命的缺点：每当客户端发起请求，服务器将请求转发给CGI，WEB 服务器就请求操作系统生成一个新的CGI解释器进程(如php-cgi），CGI进程则处理完一个请求后退出，下一个请求来时再创建新进程。

我们知道，执行一个PHP程序的必须要先解析`php.ini`文件，然后模块初始化等等一系列工作，每次都反复这样非常浪费资源。

[FastCGI协议](http://andylin02.iteye.com/blog/648412)在CGI协议的基础上，做出了如下改变：
1. FastCGI被设计用来支持常驻（`long-lived`）应用进程，减少了`fork-and-execute`带来的开销
2. FastCGI进程通过监听的socket，收来自Web服务器的连接，这样FastCGI 进程可以独立部署
3. 服务器和FastCGI监听的socket 之间按照消息的形式发送环境变量和其他数据

我们称实现了FastCGI协议的程序为FastCGI程序，FastCGI程序的交互方式如下图所示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201202212619.png)

## PHP-FPM 的诞生
FastCGI 程序固然已经很好了，但我们的需求总是有点苛刻，它还是存在一些明显缺点的：
1. 当我们更改配置文件(`php.ini`)后，`php-cgi`（FastCGI 程序） 无法平滑重启
2. 我们fork的进程个数和请求量正比，请求繁忙时 fork 进程多，动态调整 `php-cgi`还没做到

上面提及php-cgi 实现的FastCGI问题官方没有解决，幸运的是有第三方帮我们解决了，它就是 `php-fpm`。

它可以独立运行，不依赖php-cgi，换句话说，它自己实现了FastCGI协议并且支持进程平滑重启且带进程管理功能。

### 参考链接
* [从CGI到FastCGI到PHP-FPM](http://blog.leanote.com/post/weibo-007/%E4%BB%8ECGI%E5%88%B0FastCGI%E5%88%B0PHP-FPM)