---
title: Docker 固定容器 IP
date: 2023-04-04 15:50:56
tags: ["Docker"]
categories: ["Docker"]
---

Docker提供了三种网络模式，分别是null、host和bridge。

<!-- more -->

查看 Docker 网络模式列表：
```bash
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
7fa40584b191   bridge    bridge    local
31f5de3d2e94   host      host      local
bd4a2614a570   none      null      local
```

* `null`：是最简单的模式，也就是没有网络，但允许其他的网络插件来自定义网络连接。
* `host`：模式直接使用宿主机网络，相当于去掉了容器的网络隔离（其他隔离依然保留），所有的容器会共享宿主机的IP地址和网卡。这种模式没有中间层，通信效率高，但缺少了隔离，运行太多的容器也容易导致端口冲突。
* `bridge`：桥接模式，它有点类似现实世界里的交换机、路由器，只不过是由软件虚拟出来的，容器和宿主机再通过虚拟网卡接入这个网桥，那么它们之间也就可以正常的收发网络数据包了。不过和host模式相比，bridge模式多了虚拟网桥和网卡，通信效率会低一些。

Docker 默认使用的模式是 bridge。

默认是不会分配固定 IP 的，也就是每次重启容器时，容器的 IP 将由启动顺序决定，这就会导致容器提供的服务是不可靠的。

不用使用默认的 bridge 网络模式，进行固定 IP 分配，否则会提示：
```bash
docker: Error response from daemon: user specified IP address is supported on user defined networks only.
```

## 创建自定义网络

因此需要先创建一个自定义网络：
```bash
# 命令格式：sudo docker network create --subnet=[自定义网络广播地址]/[子网掩码位数] [自定义网络名]
$ sudo docker network create --subnet=172.20.0.0/24 bridge0
```

`--subnet` ：设置前 24 位为网络位，后 8 位为主机位，该网段可用 IP 地址：`172.20.0.1` 到 `172.20.0.254`

## 固定容器 IP

```bash
sudo docker run -it \
  --name network-test \
  --net bridge0 \
  --ip 172.20.0.2 \
  ubuntu:latest /bin/bash
```

* `--net`：指定自定义网络
* `--ip`：选定自定义网络下固定 IP 地址


查看容器 IP：
```bash
$ docker inspect --format='{{.Name}} - {{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)

/network-test - 172.20.0.2
```