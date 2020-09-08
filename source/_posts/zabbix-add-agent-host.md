---
title: Zabbix 快速上手——添加Agent 主机
date: 2020-09-04 21:48:16
tags: ["Zabbix", "Linux"]
categories: ["Linux", "Zabbix"]
---

Zabbix-Server 安装完成之后，下一步需要添加主机才能看到数据。

<!-- more -->

## 安装Zabbix Agent
Zabbix Agent 的作用是将服务器的数据发送给 Zabbix Server，所以只需要在需要监控的主机上安装 Zabbix Agent 就够了。

因为我的环境是：`Ubuntu 18.04`、`Nginx`、`Mysql`、`PHP`，根据[官网](https://www.zabbix.com/cn/download)的选择对应的下载链接。

在有了`Mysql` 和 `Nginx`的情况下，这里我只选择安装 `Zabbix Agent`，如果没有的话，那就需要额外安装`zabbix-mysql`、`zabbix-nginx-conf`、`zabbix-frontend-php`。
```
$ wget https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+bionic_all.deb
$ dpkg -i zabbix-release_5.0-1+bionic_all.deb
$ apt update
$ apt install zabbix-agent
```

## 配置 Zabbix Agent

```
# vim /etc/zabbix/zabbix_agentd.conf

Server：Zabbix Server 的IP 地址
ServerActive：Zabbix Server 的IP 地址
Hostname：Zabbix Agent 这台主机的别名
```
核心的配置只有这三行，改完之后，重启以下 Zabbix Agent。

```
$ systemctl restart zabbix-agent
```

## 添加主机
完成以上配置之后，下一步需要做的就是打开 Zabbix 的Web 端，开始添加主机。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200904200443.png)

配置主机基础信息：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200904205302.png)

* 主机名称：zabbix_agentd.conf 中的Hostname
* 客户端IP：需要监控的主机的IP 地址
* 端口默认使用 10050

配置模版：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200904213331.png)

需要注意的是，如果没有配置模版，可能会导致没有数据。

然后点击添加即可。

打开监控面板，点击主机，正常情况下，主机状态应该是这样的。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200904214447.png)

至此就完成了Agent 的添加，点击最新数据或者图形可以看到相应的数据。

### 参考链接
* [安装zabbix-agent并添加到zabbix web中监控](https://blog.51cto.com/dyc2005/1971212)
* [Zabbix 使用 Zabbix-Agent 添加新的Linux服务器监控](https://blog.csdn.net/kk185800961/article/details/84105621)