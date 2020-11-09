---
title: Linux 查看系统、硬件信息
date: 2020-11-09 22:33:03
tags: ["Linux"]
categories: ["Linux"]
---
以下命令都是基于Ubuntu。

<!-- more -->

### 系统相关

#### 查看内核/操作系统/CPU信息
```
$ uname -a 
```

#### 查看操作系统版本
```
$ head -n 1 /etc/issue
```

#### 查看机器型号
```
$ dmidecode | grep "Product Name" 
```

#### 查看主机名
```
$ hostname
```

#### 列出所有PCI设备
```
$ lspci -tv 
```

#### 列出所有USB设备
```
$ lsusb -tv 
```

#### 列出加载的内核模块

```
$ lsmod
```

#### 查看环境变量
```
$ env
```

### 资源

#### 查看内存使用量和交换区使用量
```
$ free -m 
```

#### 查看各分区使用情况

```
$ df -h 
```

#### 查看总内存量
```
$ grep MemTotal /proc/meminfo
```

#### 查看空闲内存量
```
$ grep MemFree /proc/meminfo
```

#### 查看系统运行时间、用户数、负载
```
$ uptime
```

#### 查看系统负载
```
$ cat /proc/loadavg 
```

### CPU

#### 查看CPU 统计信息
```
$ lscpu
```

#### 查看单个CPU 信息
```
cat /proc/cpuinfo
```

### 磁盘和分区

#### 查看磁盘空间信息
```
$ df -h
```

#### 查看挂接的分区状态
```
mount | column -t
```

#### 查看所有分区
```
$ fdisk -l
```

#### 查看所有交换分区
```
$ swapon -s
```

### 网络

#### 查看所有网络接口的属性
```
$ ifconfig
```

#### 查看防火墙设置
```
$ iptables -L   
```

#### 查看路由表
```
$ route -n    
```

#### 查看所有监听端口

```
$ netstat -lntp  
```

#### 查看所有已经建立的连接
```
$ netstat -antp  
```

#### 查看网络统计信息
```
$ netstat -s  
```

### 进程
#### 查看所有进程
```
$ ps -ef
```

#### 实时显示进程状态
```
$ top
```

### 用户

#### 查看活动用户
```
$ w
```

#### 查看指定用户信息
```
$ id <用户名>   
```

#### 查看用户登录日志

```
$ last
```

#### 查看系统所有用户

```
$ cut -d: -f1 /etc/passwd
```

#### 查看系统所有组
```
$ cut -d: -f1 /etc/group
```

#### 查看当前用户的计划任务
```
$ crontab -l
```

### 服务

#### 列出所有系统服务
```
$ chkconfig --list
```

#### 列出所有启动的系统服务
```
$ chkconfig --list | grep on
```

### 参考链接
* [Linux 查看CPU信息，机器型号，内存等信息](https://my.oschina.net/hunterli/blog/140783)