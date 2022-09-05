---
title: 记一次服务器被 kdevtmpfsi 变矿机
date: 2020-12-03 22:04:11
tags: ["Linux", "运维"]
categories: ["Linux"]
---

昨天有台测试服务器被告知服务异常，进服务器之后才发现是因为docker 异常退出了。

<!-- more -->

将docker 运行起来之后，发现有个不认识的进程 `kdevtmpfsi` 占用CPU 异常的多，Google 一下才知道，好家伙，服务器被当成矿机了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201204164931.png)

直接 kill 并不能将其结束掉，它还有守护进程及可能存在的定时任务。

### 1. 首先查找文件
```
$ find / -name kinsing      // 守护进程
$ find / -name kdevtmpfsi   // 挖矿进程
```
如果Redis 是运行在本地，上面两个文件通常是在`/tmp/`目录下。

如果Redis 是以容器的方式运行，则通常是在`/var/lib/docker/overlay2/`（容器的 `/tmp/` 目录）下。

### 2. 将其删除
```
$ rm -f kinsing kdevtmpfsi
```
这里被感染的容器也不一定是Redis ，比如我的则是PHP，所以需要进入到被感染的容器内才能找到。

### 3. 干掉进程
```
$ ps -aux | grep kinsing
$ ps -aux | grep kdevtmpfsi
$ kill -9 pid
```

### 4. 查看定时任务
```
$ crontab -l
```

存在定时任务的不一定是当前用户，可以使用以下命令查找其他用户是否存在任务：
```
$ for user in $(cut -f1 -d: /etc/passwd); do crontab -u $user -l; done
```

定时任务还可能存在于以下地方：
1. `/etc/crontab`
2. `/var/spool/cron/`
3. `/var/spool/cron/crontabs/`

至此就完成了病毒的清理，网上千篇一律的全是这种处理方式，但这个方式并不适合我，我尝试了很多次，无论我怎么删除，病毒还是存在。

因为病毒是依赖于容器生存的，于是我便将容器停止掉，通过`docker logs` 实时查看容器最后10条日志：

```
docker logs -f -t --tail 10 <容器id/容器名称>
```

十分钟之后，总算让我逮到了：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201204172650.png)

虽然目睹了全过程，但这时我依然无能为力，因为我不知道上面那些命令是如何自动启动的。

尝试了各种方式，但都无解，十分钟之后病毒还是会出来，最终我只能把这个被感染的容器给弃用了，重新起一个新的容器。

### 总结
`kdevtmpfsi`病毒的产生，通常是因为Redis 对外开放 `6379`端口，且没设置密码或者密码过于简单导致。

所以服务器一定要设置好防火墙，像`3306`、`6379` 这种常用端口，尽量减少对外开放的机会。

### 参考链接
* [Linux.Packed.753](https://vms.drweb.cn/virus/?i=19722604&lng=cn)