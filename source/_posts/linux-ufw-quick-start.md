---
title: linux ufw quick start
date: 2020-12-10 21:54:58
tags: ["Linux", "Ubuntu", "UFW", "防火墙"]
categories: ["Linux"]
---

之前已经了解了 iptables 是设置防火墙的命令行工具，但对于初学者而言，它的上手曲线太陡了。

<!-- more -->

[UFW](https://help.ubuntu.com/community/UFW) （即简单防火墙）相较 iptables，对于初学者而言，则易于上手得多。

UFW 默认安装在Ubuntu上。如果由于某种原因已将其卸载，则可以使用如下命令进行安装：
```
$ sudo apt install ufw 
```

开启 `IPV6`
```
$ sudo vim /etc/default/ufw

IPV6=yes
```

## 查看UFW状态
Ubuntu 默认没有开启 UFW。
```
$ sudo ufw status
```
* `inactive`：表示防火墙关闭状态 
* `active`：表示防火墙开启状态

## 开启UFW
```
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
初次开启 UFW 没有任何规则（如果之前已经添加过UFW 规则，则还是存在的），如需查看以开启哪些规则，同样使用`ufw status`命令。

## 关闭UFW
```
$ sudo ufw disable
```

## 重置所有规则
```
$ sudo ufw reset
```

## 允许指定端口
```
$ sudo ufw allow http   // sudo ufw allow 80
```

### 指定特定 IP

使用UFW时，还可以指定IP地址。
例如，如果要允许来自特定IP地址的连接，则可以使用如下命令：
```
$ sudo ufw allow from <ip_address>
```

还可以通过添加`to any port`端口号来指定允许IP地址连接的特定端口。

例如，如果要允许 `203.0.113.4` 连接到端口22（SSH），则可以使用如下命令：
```
$ sudo ufw allow from 203.0.113.4 to any port 22
```

## 禁止指定端口
```
$ sudo ufw deny https   // sudo ufw deny 443
```

## 删除指定规则

正式删除具体规则之前，先使用如下命令查看对应编号：
```
$ sudo ufw status numbered
```

删除指定编号对应的规则：
```
$ sudo ufw delete <id>
```

## 检查UFW状态和规则
```
$ sudo ufw status verbose
```

## 重新载入配置
```
$ sudo ufw reload
```

注意事项⚠️：
1. 修改了某条规则之后，需要让UFW 重新加载配置，设定规则才会生效。
2. 谨慎禁用 ssh，否则可能会导致自己也连接不上。
3. 在启用 UFW 之前，最好检查或者重置一下规则。

### 参考链接
* [How To Set Up a Firewall with UFW on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)
* [在 Ubuntu 中用 UFW 配置防火墙](https://linux.cn/article-8087-1.html)