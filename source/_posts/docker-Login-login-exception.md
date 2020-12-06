---
title: docker Login login exception
date: 2020-12-06 22:51:20
tags: ["Linux", "运维"]
categories: ["Linux"]
---

今天刚好有空，把前天那个被挖矿病毒感染的容器给换一换。

<!-- more -->

### 问题描述

使用 `docker login` 登录时，总是会提示如下信息，可是我明明输入的是正确的账号密码。

```
Error saving credentials: error storing credentials - err: exit status 1, out: Cannot autolaunch D-Bus without X11 $DISPLAY
```

因为我使用的并不是最新的 `docker-ce` 版，而是老版本`docker.io`，所以起初我是怀疑版本出现了不兼容的问题吗？

其实不是，这是在 Ubuntu 下使用 docker 特有的 bug ，而修复办法不需要特意去卸载 `docker-compose` ，只要 “pass” 掉验证步骤。

### 问题解决
最终解决步骤如下：
#### 1. 安装 `gnupg2` 和 `pass`
```
sudo apt install gnupg2 pass
```

#### 2. 生成密钥
```
$ gpg2 --full-generate-key
```

#### 3. 查看密钥所在路径
```
$ gpg2 -k
```

#### 4. 使用 `pass` 加载验证
```
$ pass init "your key location path"
```

至此就已经pass 掉了验证步骤，可以使用 `docker login` 正常登录了。

### 参考链接
* [Docker login 报证书存储错误的解决办法](https://ug.epurs.com/post/docker-login-error-saving-credentials/)