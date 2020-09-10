---
title: 当 Docker 容器无法正常启动时如何修改配置文件
date: 2020-09-10 21:12:13
tags: ["Docker", "Linux"]
categories: ["Linux", "Docker"]
---

在容器无法正常启动的情况下，如何修改其配置文件？

问题描述：因为错误的配置文件导致容器运行异常，无法正常启动，通常情况下只有进入容器才能修改配置文件，所以在不能进入容器的情况下该怎么办呢？

<!-- more -->

这种情况下，有两种方式去修改：
2. Docker 容器的配置文件一般在 `/var/lib/docker/overlay/`目录下，可以找到该目录下对应的配置文件进行修改。
2. 把容器中的配置文件复制到主机中，修改完之后，再移动到容器中。

## 方式一

1. 查询日志
```
docker logs <容器名称/容器id>

ERROR: mysqld failed while attempting to check config
command was: "mysqld --verbose --help"
2020-09-03T12:15:54.644699Z 0 [ERROR] unknown variable 'realy-log=slave-relay-bin'
2020-09-03T12:15:54.650119Z 0 [ERROR] Aborting
```
由于异常日志可以得知是因为我将`relay-log` 写成了 `realy` 导致容器无法正常启动。

2. 查找文件
```
$ find / -name mysqld.cnf

/var/lib/docker/overlay2/02e1644bc1a4dc1adc9a0300e1815f364416570d69b715fb3b7de0a06cf0c495/diff/etc/mysql/mysql.conf.d/mysqld.cnf
/var/lib/docker/overlay2/02e1644bc1a4dc1adc9a0300e1815f364416570d69b715fb3b7de0a06cf0c495/merged/etc/mysql/mysql.conf.d/mysqld.cnf
/var/lib/docker/overlay2/4f128d7fb1200f722b0d2cfe3606149fe72987a7a16bc78551a2b1fe6c6c6572/diff/etc/mysql/mysql.conf.d/mysqld.cnf
/var/lib/docker/overlay2/a68f1af4adf982b037f1bd37d61082fde1fa2b0e26ea0e2fe146edcb69b198ea/diff/etc/mysql/mysql.conf.d/mysqld.cnf
```
这里可能会出现多个配置文件，这是因为每一次重启Mysql 容器都会保留一个配置文件，所以理论上，直接修改第一个配置文件，就是当前Mysql 所使用的配置文件。

3. 修改配置文件

4. 重启容器即可。

## 方式二
如果第一种方式没生效，那可以尝试第二种方式。

1. 复制容器中的配置文件到主机：
```
# 语法：docker cp <容器名称/容器id>:<配置文件在容器中的路径> <需要复制到主机的路径>

$ docker cp mysql:/etc/mysql/mysql.conf.d/mysqld.cnf ~/mysqld.cnf
```

2. 修改主机中的配置文件

3. 将该配置文件mv 到容器中：
```
# 语法：docker cp <配置文件在主机中的路径> <容器名称/容器id>:<配置文件在容器中的路径>

$ docker cp ~/mysqld.cnf mysql:/etc/mysql/mysql.conf.d/mysqld.cnf  
```
4. 重启配置文件即可。

总结：两种方式均可以有效解决上述问题，当然这类方式仅适用于容器是因错误的配置文件导致无法正常启动的情况。

### 参考链接
* [Docker修改无法启动的容器的配置文件](https://blog.csdn.net/LinHenk/article/details/88111616)
