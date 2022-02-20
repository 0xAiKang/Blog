---
title: 如何解决类似 curl:(7) Failed to connect to raw.githubusercontent.com port 443:Connection refused 的问题
date: 2022-02-20 19:11:04
tags: ["Mac"]
categories: ["Mac"]
---


有时使用Homebrew 安装工具时，会出现类似下面的错误：

<!-- more -->

```
sh -C "§(curl-fsSL https: //raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" 

curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```
  
这是因为github 的一些域名的 DNS 解析被污染，导致DNS 解析过程无法通过域名取得正确的IP地址。

## 解决方案
打开 https://www.ipaddress.com/ 输入访问不了的域名，或者在终端使用`ping` 命令：
```shell
➜  ~ ping raw.githubusercontent.com
PING raw.githubusercontent.com (199.232.68.133): 56 data bytes
64 bytes from 199.232.68.133: icmp_seq=0 ttl=49 time=226.190 ms
64 bytes from 199.232.68.133: icmp_seq=1 ttl=49 time=325.860 ms
c64 bytes from 199.232.68.133: icmp_seq=2 ttl=49 time=326.362 ms
64 bytes from 199.232.68.133: icmp_seq=3 ttl=49 time=277.042 ms
^C
--- raw.githubusercontent.com ping statistics ---
```

查询到正确的域名后，将其添加到对应的`hosts` 文件中即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220215222459.png)

添加完 `hosts` 配置之后，`homebrew` 就能正常了。

## 参考连接
* [如何解决类似 curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused 的问题](https://github.com/hawtim/hawtim.github.io/issues/10)