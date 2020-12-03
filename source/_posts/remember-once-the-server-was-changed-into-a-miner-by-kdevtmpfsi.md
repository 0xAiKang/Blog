---
title: 记一次服务器被 kdevtmpfsi 变矿机
date: 2020-12-03 22:04:11
tags: ["Linux"]
categories: ["Linux"]
---

昨天有台测试服务器被告知服务异常，进服务器之后才发现是因为docker 异常退出了。

<!-- more -->

将docker 运行起来之后，发现有个不认识的进程 `kdevtmpfsi` 占用CPU 异常的多，Google 一下才知道，好家伙，服务器被当成矿机了。

通常是因为Redis 对外开放 `6379`端口，且没设置密码或者密码过于简单导致。

直接 kill 并不能将其结束掉，它还有守护进程及可能存在的定时任务。

### 1. 首先查找文件
```
$ find / -name kinsing      // 守护进程
$ find / -name kdevtmpfsi   // 挖矿进程
```
如果Redis 是运行在本地，上面两个文件通常是在`/tmp/`目录下。

如果Redis 是以容器的方式运行，则通常是在`/var/lib/docker/overlay2/`下。

### 2. 将其删除
```
$ rm -f kinsing kdevtmpfsi
```

### 3. 干掉进程
```
$ ps -aux | grep kinsing kdevtmpfsi
$ kill -9 pid
```

### 4. 查看定时任务
```
$ crontab -l
```

定时任务还可能存在于以下地方：
1. `/etc/crontab`
2. `/var/spool/cron/`
3. `/var/spool/cron/crontabs/`

通常做完以上操作，机器就恢复正常了。但如果你像我一样不幸，怎么都杀不掉（通常是 Docker + Redis），那么可能需要考虑将容器停掉或者再起一个，病毒藏的太深了，笔者在写这篇笔记时，也没找到。