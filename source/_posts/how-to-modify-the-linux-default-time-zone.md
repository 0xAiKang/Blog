---
title: 如何修改 Linux 默认时区
date: 2020-08-09 21:23:30
tag: ["Linux"]
categories: ["Linux"]
---

在上一篇笔记中，我们知道了如何在Linux 中查看系统默认时区，这篇笔记来学习以下如何修改默认时区。

<!-- more -->

在Linux 服务器或系统上保持正确的时间始终是一个好习惯，它可能具有以下优点：
* 由于Linux 中的大多数任务都是按时间控制的，因此可以保持系统任务的及时运行。
* 在系统上记录事件和其他信息的正确时间等等。

在Linux 中设置时区，有几种方式。

### 0x1. 使用tzselete 命令
1. 使用`tzselete` 命令选择所在时区。
2. 将时区所在的配置文件`TZ='Asia/Shanghai'; export TZ` 添加到`~/.profile`文件。
3. 使用`source ~/.profire`命令，使时区设置生效。

### 0x2. 使用timedatectl 命令
Ubuntu 系统提供了`timedatectl` 命令，非常方便的供我们查看设置Linux 系统时区。

```
$ timedatectl set-timezone "Asia/ShangHai"
```

如果你忘记了你想要的时区叫什么名字，那么可以使用下面的命令查看所有可用时区：
```
$ timedatectl list-timezones
```

因为 Linux 的时间分为两种：
1. 硬件时间：由 BIOS（或CMOS）所负责。
2. 系统时间：由 Linux 所负责，系统时间在系统开关机后读取硬件时间后，再由 Linux 管理时间。

### 0x3. 设置硬件时间

```
$ ls -al | grep localtime
lrwxrwxrwx  1 root root         27 Jul 24 00:57 localtime -> /usr/share/zoneinfo/Etc/UTC
```
可以看到默认链接的是`UTC`，所以需要手动更改链接时区文件。

```
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
```

查看硬件时间
```
$ hwclock -r
```

将系统时间改为硬件时间
```
$ hwclock --hctosys
```

需要想清楚的是，时间戳本身是永远不变的，无论在哪个时区同一时刻所生成的时间戳一定是一样的。

会发生变化的只有时区，而时间戳则是根据时区的不同而解析出来的时间不同。

### 参考链接
* [How to Set Time, Timezone and Synchronize System Clock Using timedatectl Command](https://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/)
* [Linux 查看设置系统时区](https://www.cnblogs.com/kerrycode/p/4217995.html)
* [Linux 时间以及时区](https://david50.pixnet.net/blog/post/45228135-%5B%E7%AD%86%E8%A8%98%5Dlinux%E6%99%82%E9%96%93%E5%8F%8A%E6%99%82%E5%8D%80)
