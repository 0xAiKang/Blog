---
title: git 设置和取消代理
date: 2022-05-02 14:46:04
tags: ["Git"]
categories: ["Git"]
---

在使用 Git 时，经常会需要克隆仓库，有时候是国内的仓库，有时候是国外的仓库，如果直接强制让终端走代理，那么当克隆国内仓库时，速度可能特别慢。

这个时候其实可以只针对部分域名进行代理设置，而其他域名则不用走代理。

<!-- more -->

### https 代理

针对所有 https 请求生效：
```bash
git config --global https.proxy https://127.0.0.1:1080
```

取消设置代理：
```bash
git config --global --unset https.proxy
```

只针对 `github.com` 生效
```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```

取消设置代理
```bash
git config --global --unset http.https://github.com.proxy
```

### ssh 代理
需要修改 `~/.ssh/config`文件，如果没有，新建一个。

macOS 下，同样仅为 `github.com` 设置代理：
```config
Host github.com
    User git
    ProxyCommand nc -v -x 127.0.0.1:1086 %h %p
```

如果是在 Windows 下，设置代理命令会有所不同：
```config
Host github.com
    User git
    ProxyCommand connect -S 127.0.0.1:1086 %h %p  
```

## 参考链接
* [git 设置和取消代理](https://gist.github.com/laispace/666dd7b27e9116faece6)