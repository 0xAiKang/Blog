---
title: Docker Hub 快速上手
date: 2020-08-30 18:32:02
tags: ["Docker", "Tutorial", "Docker Hub"]
categories: ["Docker", "Tutorial"]
---

最近将常使用的镜像放在了Docker 仓库（Docker Hub）上。GitHub 是托管代码的地方，而[Docker Hub](https://hub.docker.com/) 则是托管镜像的地方。

<!-- more -->

目前大部分需求都可以直接在 Docker Hub 中下载镜像来实现，如果想使用自己仓库中的镜像，那么需要先[注册](https://hub.docker.com/)一个账号。

## 创建仓库
想要从 Docker Hub 使用自己的镜像之前，首先得[创建](https://hub.docker.com/repository/create)一个仓库，然后将目标镜镜像 push 到该仓库。

这个仓库可以是公开的也可以是私有的，这个并不影响你正常使用。

创建成功之后，就可以看到该仓库了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200830180907.png)

## 发布镜像
在发布之前，确保你本地存在目标镜像，可以使用 `docker images`来查看：

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
adminer             latest              c3588b6003bb        3 weeks ago         90.4MB
```

创建 Tag：
```
# 语法
docker tag local-image:tagname new-repo:tagname

# 实例
docker tag adminer:latest hoooliday/runfast:adminer
```
前面的 `tagname` 是本地镜像的标签名称，后面的`tagname` 是该镜像在仓库中的标签名称。

再次查看本地镜像：
```
$ docker images
hoooliday/runfast   adminer             c3588b6003bb        3 weeks ago         90.4MB
adminer             latest              c3588b6003bb        3 weeks ago         90.4MB
```

发布镜像：
```
# 语法
docker push new-repo:tagname

# 实例
docker push hoooliday/runfast:adminer
```

发布成功之后，可以打开 Docker Hub 在 Repositories 的列表中就看到刚才的镜像了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200830181829.png)

## 拉取镜像

首先需要在命令行中登录你的 docker hub 账号：
```
$ docker login
```

拉取自己的镜像，这里以 [adminer](https://hub.docker.com/_/adminer) 这个镜像为例：
```
docker run --link mysql:mysql --name adminer \
-d --restart=always \
-p 8006:8080 \
hoooliday/runfast:adminer
```
唯一需要注意的就是最后一行，如果想要使用官方最新版本的 adminer ，那就直接写成 adminer，但如果想要使用自己的镜像，那就需要写成 `username/repo:tagname` 的格式。

查看本地所有镜像：
```
$ docker images
hoooliday/runfast   adminer             c3588b6003bb        3 weeks ago         90.4MB
```

此持就完成了Docker 镜像的发布和拉取了，当然这只是 Docker Hub 所有功能中的冰山一角。