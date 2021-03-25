---
title: Linux 如何挂载新硬盘
date: 2021-03-25 22:40:48
tags: ["Linux", "Ubuntu"]
categories: ["Linux", "Ubuntu"]
---

如何将一块新的硬盘挂载到Linux 操作系统呢？

<!-- more -->

下面以`Ubuntu 18.04`的发行版作为演示。

首先查看系统当前硬盘分配情况：
```
$ cd /dev && ls sd* -al
```
![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214146.png)

默认情况下，系统硬盘标记为`/dev/sda`，`sda1`、`sda2`这些表示对应硬盘下的分区名称。

查看当前系统硬盘挂载情况：
```
$ df -h 
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214212.png)

可以到看，该系统当前一共挂载了两块硬盘，分别是：
1. 分区名称为 `/dev/sda2`的系统盘 10G，挂载点为`/`。
2. 分区名称为 `/dev/sdb1`的临时盘 2.5G，挂载点为`/mydata`。

现在来为该系统添加第三块硬盘，并尝试挂载到指定目录。

## VirtualBox 添加磁盘
添加硬盘之前，需要先将机器给停掉，右键设置，点击存储

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214327.png)

创建虚拟盘：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214341.png)

按照默认选择VDI 就好：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214403.png)

根据自身情况，选择动态分配或者固定大小

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214421.png)

这里选择分配三个G，然后点击创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214503.png)

将新硬盘加入进来，然后启动机器。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214613.png)

## 分区

连接上机器之后，再次查看所有系统硬盘：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214640.png)

可以看到这次多了一个叫做`sdc` 的硬盘，首先需要对该硬盘进行分区，然后才能挂载。

这里我只需要新增一个主分区，执行以下命令：
```
$ (echo n; echo p; echo 1; echo ; echo ; echo w) | sudo fdisk /dev/sdc
```

这条命令最终会做以下几件事情：
1. `echo n` 新增分区
2. `echo p` 新建主分区
3. `echo 1` 新增一个主分区
4. `echo ` 表示『回车』确定
5. `echo 2` 写入并退出
6. 将以上输出作为输出通过管道符传递给`fdisk`命令
7. `/dev/sdc` 表示需要分区的硬盘

将文件系统写入分区：
```
sudo mkfs -t ext4 /dev/sdc1
```

## 挂载硬盘

将新硬盘挂载到指定目录：
```
$ sudo mkdir /boo && sudo mount /dev/sdc1 /boo
```
再次使用`df -h`命令查看磁盘情况，可以到看新硬盘已经挂载到指定目录下了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210325214815.png)

最后记得设置开机挂载，使用`blkid` 命令获取硬盘UUID：
```
$ sudo -i blkid
```

输出内容类似：
```
/dev/sdc1: UUID="f8025940-19bc-4943-9711-b431f478838e" TYPE="ext4" PARTUUID="d746a3a1-01"
```

编辑`/etc/fstab` 文件，添加以下内容：
```
UUID=9da67a01-aaae-4979-93fd-9916f010731a /boo ext4 defaults 0 0
```
至此就完成了硬盘挂载的所有操作了。

## 参考链接
* [虚拟机VirtualBox怎么添加新的虚拟硬盘](https://blog.csdn.net/love__coder/article/details/8270856)
* [Azure: 给 ubuntu 虚机挂载数据盘](https://www.imooc.com/article/28638)


