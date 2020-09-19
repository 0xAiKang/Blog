---
title: Mysql 常见异常分析
date: 2020-09-17 20:52:29
tags: ["Mysql"]
categories: ["Mysql"]
---

本文用来整理 Mysql 使用过程中遇到的一些问题。

<!-- more -->

## Mysql 无法正常启动
异常描述：Mysql Server 无法正常启动，Client 连接Mysql 异常如下：

> ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)

首先，这个错误意味着 `/var/run/mysqld/mysqld.sock` 不存在，而该文件之所以不存在，可能是因为没有安装 `mysql-server`，也可能是因为该文件被移动了。

如果是需要连接本机的Mysql（mysql -hlocalhost -uroot -p），那么需要先安装 `mysql server`：
```
apt-get install mysql-server -y
```

如果Mysql 服务确实有在本地运行，那么请检查`/etc/mysql/mysql.conf.d/mysqld.cnf` 配置文件，是否存在以下配置：
```
socket = /var/run/mysqld/mysqld.sock
```

如果只是需要连接其他主机，那么在本机上不安装 `Mysql Server` 也可以，但需要保证“其他主机”的Mysql 已经正常启动。

```
mysql -h<hostname> -uroot -p
```

总结：
最有可能的情况是需要连接的Mysql 服务根本没有启动，要么没有在与从终端运行MySQL客户端的主机相同的主机上运行，小概率是因为配置文件错误导致。

## Mysql 用户验证失败
异常描述：Mysql 创建完该用户之后，赋予权限并设置密码，但是总是会提示如下异常：

> ERROR 1045 (28000): Access denied for user 'zabbix'@'172.17.0.1' (using password: YES)

出现该异常信息可能有以下几种情况：
1. 用户名密码错误
2. 该用户权限不足

## Mysql 断开连接

异常描述：Mysql 偶尔会自己断开连接，然后必须重启Mysql 服务才能正常运行。

> ERROR 2013 (HY000): Lost connection to MySQL server at 'reading initial communication packet', system error: 102

目前并没有找到合适的解决方案，不过能大致确定以下几个方向：
1. 反向DNS 解析，避免使用`localhost`
2. 允许使用所有连接？

localhost 对应socket？127.0.0.1 对应 TCP/IP？

### 参考链接
* [错误2002(HY000)：无法通过Socket‘/var/run/mysqld/mysqld.sock’连接到本地MySQL服务器(2)](https://cloud.tencent.com/developer/ask/35881)
* [ERROR 2013 (HY000): Lost connection to MySQL server at 'reading authorization packet', system error: 0](https://stackoverflow.com/questions/21091850/error-2013-hy000-lost-connection-to-mysql-server-at-reading-authorization-pa)