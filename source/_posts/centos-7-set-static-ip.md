---
title: Centos 7 设置静态 IP
date: 2021-05-07 22:05:55
tags: ["Linux", "Centos"]
categories: ["Linux"]
---

通常本地的虚拟机，默认都是动态IP，这就意味着，每次重启机器，IP 地址都会发生变化，虽然不影响正常使用，但是每次重启都发生变化，这就导致还需要看一眼，才知道当前新的IP 是多少，那么有没有什么办法可以永久设置成静态IP 呢，答案是有的。

<!-- more -->

系统版本：
```
$ cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)
```

查看当前网卡的名称：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210507115101.png)

> 一台电脑可能有多个网卡，如何判断哪一个是当前正在使用的？

就看哪个网卡的IP 刚好是该机器当前的IP。

比如在上面的例子中，机器当前的IP 是`192.168.1.100`，那么只要确定某个网卡的IP 也是 `192.168.1.100`，那这个网卡就是我们要找的了。

编辑对应的配置文件：
```
$ cd /etc/sysconfig/network-scripts
$ vim ifcfg-网卡名称
```

设置静态IP 配置文件如下：
```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"         # 使用静态IP地址，默认为动态，即dhcp
IPADDR="192.168.241.100"   # 设置的静态IP地址
NETMASK="255.255.255.0"    # 子网掩码
GATEWAY="192.168.1.1"      # 网关地址
DNS1="114.114.114.114"     # DNS服务器
DEFROUTE="yes"

IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"

NAME="ens33"
UUID="95b614cd-79b0-4755-b08d-99f1cca7271b"
DEVICE="ens33"

ONBOOT="yes"               #是否开机启用
```

重启网络：
```shell
$ service network restart
```

如果没有生效，可以尝试编辑`/etc/resolv.conf`，加入以下配置：
```shell
nameserver 114.114.114.114  # 和上面的DNS 服务器保持一致
```
再次重启网络。

需要注意的是：这种配置是永久生效的，即使下次重启电脑，IP 地址也不会发生变化。

至此，就完成了设置静态IP 的全部配置。
