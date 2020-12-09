---
title: Linux iptables 常用规则整理
date: 2020-12-09 21:19:00
tags: ["Linux", "Ubuntu", "iptables"]
categories: ["Linux"]
---

因为手上一直管理着两台实体机（服务器），而实体机的是没有“软防”这个概念的，“硬防”规则只能自己去设定。

<!-- more -->

而Linux 原始的防火墙工具iptables 过于繁琐，上手曲线较陡，所以这篇笔记就用来整理 Linux 的 iptables 相关知识。

## iptables 是什么
我们常常会听到这样的说法：“iptables 是一个防火墙”，其实不是，它也不是一个系统服务，所以不能使用如下命令启动/停止/重启。

```
systemctl start/stop/restart iptables
```

iptables 其实只是一个命令行工具，它用来操作 netfilter 内核防火墙，所以真正应用的防火墙应该是**netfilter**。

当拿到一台Linux 后，iptables就在那里，默认情况下它允许所有流量。

```
$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

## 允许特定端口访问
访问过程如下：
1. 将此规则附加到输入链（-A INPUT），以便查看传入流量
2. 检查是否为TCP（-p tcp）
3. 如果是，检查输入是否进入端口（--dport ssh）
4. 如果是，接受输入（-j ACCEPT）

```
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # 允许访问22端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # 允许访问80端口
iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # 允许访问443端口
iptables -A FORWARD -j REJECT                    # 禁止其他未允许的规则访问
```

## 禁止特定端口访问
```
iptables -A INPUT -p tcp --dport 6379 -j DROP    # 禁止6379端口传入流量
```
如果想要屏蔽UDP流量而不是TCP流量，只需将上述规则中的 `tcp` 修改为 `udp` 即可。

## 禁用防火墙
如果需要临时/永久禁用iptables 防火墙，则可以使用以下命令清除所有规则：
```
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F
```
设置完成之后，不用重启任何服务，其防火墙规则已经刷新了（允许所有流量）。

### 参考链接
* [25 个有用的 iptables 防火墙规则](http://www.codebelief.com/article/2017/08/linux-25-useful-iptables-firewall-rules/)
* [如何在Ubuntu上启动/停止iptables？](https://ubuntuqa.com/article/10698.html)
* [IptablesHowTo](https://help.ubuntu.com/community/IptablesHowTo#Configuration%20on%20startup)
* [iptables - Linux](https://wangchujiang.com/linux-command/c/iptables.html)
