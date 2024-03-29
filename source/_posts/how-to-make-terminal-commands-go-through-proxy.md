---
title: 如何让终端命令走代理？
date: 2020-08-18 23:39:58
tags: ["Skill"]
categories: ["Skill"]
---

问题描述：今天本来打算使用Homebrew 更新一个工具，但是输入完`brew updata` 之后，就一直是`Updating Homebrew...`

这个时候，我产生了几个疑问：
1. 为什么卡着不动了，明明是有网络的啊。
2. 难道是因为Homebrew 需要访问国外的源？
3. Shadowsocks 明明是开着全局代理，为什么没有用？
4. 如何让终端命令走代理，或者说如何让 Homebrew 走代理更新？

## 方案

首先先回答一下上面那些问题，因为国内网络环境进一步恶劣，使得从根本上造成了这个问题的产生。因为`Shadowshocks`的全局代理虽然对浏览器是有效，但对命令行无效。

所以这一切的问题可以总结成一个问题：如果能让终端命令走代理就好了。

### 临时生效
好在Homebrew 是支持全局代理的，所以我们只需要在当前命令行环境中加入代理配置就好了。

```
export ALL_PROXY=socks5://127.0.0.1:1080

// 1080 是本地 socks5 监听端口
```

> 如何知道终端命令有没有走代理？

有一个很简单的方法，那就是通过Curl 命令：
```
curl https://www.google.com
```
如果走了本地代理，那么很快终端就会有输出，如果没有走则会提示403 端口请求超时。

### 永久生效
需要注意的是，上面的配置仅仅只是临时的，如果重启一下终端，这个配置就失效了，那么有没有办法可以永久生效呢？

当然是有的，只需要将环境变量写入终端中。
```
# bash
echo export https_proxy=http://127.0.0.1:1080 http_proxy=http://127.0.0.1:1080 all_proxy=socks5://127.0.0.1:1080 >> ~/.bash_profile

# zsh
echo export https_proxy=http://127.0.0.1:1080 http_proxy=http://127.0.0.1:1080 all_proxy=socks5://127.0.0.1:1080 >> ~/.zprofile
```

这样，Homebrew 就能通过 `Shadowsocks` 来更新了。

## 参考链接
* [让 Homebrew 走代理更新](https://www.logcg.com/archives/1617.html)
* [如何让Homebrew 走代理更新？](https://www.cnblogs.com/xjnotxj/p/7478614.html)