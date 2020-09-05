---
title: Zabbix 快速上手
date: 2020-09-05 08:49:02
tags: ["Linux", "Zabbix", "监控系统"]
categories: ["Linux", "Zabbix"]
---

因为一些特殊原因，部分环境不是搭建在云上面，而是在托管的实体机上面，这就导致原本很多云可以帮我们做的事情，现在只能自己去做了。
比如：监控系统。

本着**不想当运维的前端不是一个好全栈**的思想，我迫切需要自己搭建一套监控系统来解放我自己的双手👐️。

<!-- more -->

我希望这套监控系统是怎样的？
1. 免费开源
2. 入门相对容易
3. 支持多平台分布式监控

综合以上需求，最后我选择了 Zabbix。

网上找了一圈，并没有发现合适的入门教程，要么是教程太老了，要么是写的不够详细，学习曲线很陡，光是部署就很费劲，而Zabbix 重要的不是部署，而是学会如何使用。

所以这篇笔记就是用来记录如何快速部署 Zabbix。

## 认识 Zabbix
[Zabbix](https://www.zabbix.com/) 是一个企业级的分布式开源监控方案。

一个完整的监控系统是由服务机（zabbix server）和客户机（zabbix zgent）组成，运行大概流程是这样的：
 `zabbix agent` 需要安装到被监控的主机上，它负责定期收集各项数据，并发送到 `zabbix server` 端，zabbix server将数据存储到自己的数据库中，`zabbix web `根据数据在前端进行展现和绘图。这里 agent 收集数据分为主动和被动两种模式：
* 主动：agent 请求server 获取主动的监控项列表，并主动将监控项内需要检测的数据提交给 server/proxy 。
* 被动：server 向agent请求获取监控项的数据，agent返回数据。

工作原理：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200905083643.png)

## 安装
系统环境：
* Ubuntu 18.04 LTS
* Mysql 5.7
* PHP 7.2
* Nginx 
* Zabbix 5.0

### 1. 安装数据库
在正式安装之前，这里推荐先去[官网](https://www.zabbix.com/cn/download)找到符合自己的 Zabbix 服务器平台。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200902093733.png)

根据自己的实际环境来找到属于自己的下载链接，比如我是`Zabbix 5.0 + Ubuntu 18.04 + Mysql + Nginx`，所以我的安装方式应该是：

```
$ wget https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+bionic_all.deb
$ dpkg -i zabbix-release_5.0-1+bionic_all.deb
$ apt update
```

### 2. 安装Zabbix server，Web前端，agent
```
$ apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-agent
```
* Zabbix Server：用来接收并处理 Zabbix agent 传过来的数据
* Web 前端：Zabbix 的交互界面
* Zabbix agent：需要被监控的主机

上面安装的是 `mysql-server`，并没有安装 `mysql-client`，所以你可能需要手动安装:
```
$ apt install mysql-client
```

### 3. 初始数据库
Mysql 默认用户是root，这里不推荐直接使用 root 用户去管理 zabbix 数据库，所以还是使用官方推荐的方式，创建一个新的用户去管理：

```
$ mysql -hlocalhost -uroot -p

mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> flush privileges;
mysql> quit;
```
这里默认Mysql 是运行在本地机器上，如果Mysql 运行在容器中，而Zabbix 又运行在本机上，可能会出现一些异常（我遇到了但没能解决）。

导入初始架构和数据。
```
$ zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

### 4. 配置数据库
为Zabbix server配置数据库，

```
# vim /etc/zabbix/zabbix_server.conf

DBPassword=password
```

### 5. 配置Web 
```
# vim /etc/zabbix/nginx.conf

# 去掉前面的注释，换成你自己的端口或者域名。
# listen 80;
# server_name example.com;
```

### 6. 配置时区

```
# vim /etc/zabbix/php-fpm.conf

php_value[date.timezone] = Asia/Shanghai
```

### 7. 启动服务

启动Zabbix server和agent 进程，并为它们设置开机自启：
```
$ systemctl restart zabbix-server zabbix-agent nginx php7.2-fpm
$ systemctl enable zabbix-server zabbix-agent nginx php7.2-fpm
```

一切准备就绪之后，就可以访问了：`http://server_ip_or_name`，如果你上面配置的不是80 端口，那得记得加上对应的端口。如果你不能正常访问，那可能是因为防火墙没有允许该端口。

初次进来，需要配置相关参数，确认无误之后，点击 Next step。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200902105706.png)

Zabbix 默认的用户名和密码是`Admin`、`zabbix`，顺利登录到后台之后，记得修改默认登录密码。

## 配置中文语言包
如果需要设置中文版的环境，需要做一些额外的配置。

```
$ vim /usr/share/zabbix/include/locales.inc.php
```
将zh_CN 后面参数改为 true。

如果在选择语言时，发现还是不能选择，并且提示：

> You are not able to choose some of the languages, because locales for them are not installed on the web server.

这是因为你系统里没中文环境，查看当前的所有系统语言环境
```
$ locale -a 
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200902110702.png)

### 1. 安装中文包/
```
apt-get install language-pack-zh-hant language-pack-zh-hans
```

### 2. 配置环境变量
增加语言和编码的设置：
```
# vim /etc/environment

LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh:en_US:en"
```

### 3. 替换Zabbix 语言包
```
$ cd cd /usr/share/zabbix/locale/zh_CN/LC_MESSAGES
$ wget https://github.com/echohn/zabbix-zh_CN/archive/v0.1.0.zip
$ unzip master.zip
$ rm frontend.mo
$ cp zabbix-zh_CN-master/frontend.mo frontend.mo
```

### 4. 解决乱码问题

```
$ wget https://github.com/chenqing/ng-mini/blob/master/font/msyh.ttf
$ vim /usr/share/zabbix/include/defines.inc.php

# 找到 define('ZBX_GRAPH_FONT_NAME', 'graphfont');
# 将graphfont 替换成 msyh
```

### 5. 更新mibs 库

```
$ apt-get install snmp-mibs-downloader
```

### 6. 重启服务
```
$ systemctl restart zabbix-server zabbix-agent php7.2-fpm
```

至此Zabbix 的完整部署过程就全介绍完了。

### 参考链接
* [Zabbix 3.0 for Ubuntu 14.04 LTS 安装](https://www.cnblogs.com/zangdalei/p/5712951.html)
* [下载安装Zabbix——Zabbix 官网](https://www.zabbix.com/cn/download?zabbix=5.0&os_distribution=ubuntu&os_version=18.04_bionic&db=mysql&ws=nginx)
* [企业级分布式监控系统--zabbix](https://yq.aliyun.com/articles/611489)