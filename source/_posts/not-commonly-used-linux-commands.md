---
title: 不常用Linux命令
date: 2020-08-25 20:53:33
tags: ["Linux", "Linux Commands"]
categories: ["Linux"]
---

这篇笔记的目的是记录那些不太常用但却很实用的 Linux 命令。

<!-- more -->

## Wget
wget 命令用于文件的下载，

##### 下载单个文件
```
# 下载Ubuntu 18.04 桌面版和服务端版
$ wget https://mirror.xtom.com.hk/ubuntu-releases/18.04.2/ubuntu-18.04.2-live-server-amd64.iso
$ wget https://mirror.xtom.com.hk/ubuntu-releases/18.04.2/ubuntu-18.04.2-desktop-amd64.iso
```
wget默认会以最后一个符合”/”的后面的字符来命令，对于动态链接的下载通常文件名会不正确。

为了解决这个问题，我们可以使用参数-O来指定一个文件名：

##### 下载单个文件并重命名
```
$ wget -O file.zip http://www.minjieren.com/download.aspx?id=1080
```

##### 后台下载
当需要下载比较大的文件时，使用参数`-b`可以隐藏在后台进行下载：

```
$ wget -b http://www.minjieren.com/wordpress-3.1-zh_CN.zip
```
可以使用以下命令来察看下载进度：

```
$ tail -f wget-log
```

## Curl

## Scp
scp 命令用于文件传输，在不能使用 XShell 这类工具时，scp能很好的解决文件上传的问题。

##### 上传文件
```
$ scp -r /c/User/Desktop/dirname username@34.92.117.222:/tmp/dirname

```

##### 下载文件
```
$scp -r  Boo@34.92.117.222:/tmp/dirname /c/Users/Boo/Desktop/dirname
```

如果存在端口号：

注意：`-P`参数是大写。
```
scp -P 58812 root@103.232.86.239:/tmp/runfast_0603.sql ~/File/
```

其中 `-r` 参数表示目录，`username` 表示服务器对应用户，`@` 后面接服务器地址。

注意：不要直接使用 root 用户，因为总是会提示你权限不足。另外使用非 root 用户时，需要注意文件夹权限的问题。

## zip
zip 命令用于对文件进行打包处理，也就是我们常说的压缩。文件经压缩之后会生成一个具有`.zip`扩展名的压缩文件。

将当前目录的`dir`目录下的所有文件及文件夹压缩为 example.zip
```
$ zip -r -q example.zip dir
```
将当前目录下的所有文件及文件夹压缩为 example.zip
```
$ zip -r -q *
```
将指定文件目录的所有文件及文件夹压缩为 example.zip
```
$ zip -r -q exmaple.zip /tmp/dir
```

## unzip
unzip 命令用于解压缩由 zip 命令压缩的“.zip”压缩包。

查看压缩文件
```
$ unzip -v dir.zip
```

将压缩文件在当前目录下解压
```
$ unzip example.zip
```

将压缩文件`example.zip`在指定目录`/tmp`下解压缩，如果已有相同的文件存在，要求 unzip命令不覆盖原先的文件。
```
$ unzip -n example.zip -d /tmp
```
将压缩文件`example.zip`在当前目`dir`下解压，如果已有相同的文件，不询问，直接覆盖。
```
$ unzip -o example.zip -d 
```
`-o` 参数表示不必先询问用户，unzip执行后覆盖原有的文件；
`-d` 参数指定文件解压缩后所要存储的目录；
`-n` 参数解压缩时不要覆盖原有的文件；

## tar
tar 命令可以为linux 文件和目录创建档案。

利用tar命令，可以把一大堆的文件和目录全部打包成一个文件。

需要明确的两个概念是：打包和压缩是不同的两件事。
* 打包：是指将一大堆文件或目录变成一个总的文件；
* 压缩：则是将一个大文件通过压缩算法变成一个小文件。

为什么要区分这两个概念呢？这源于Linux中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）。

### 打包
仅打包，不压缩。
```
$ tar -cvf test.tar 20200323.log
```
`test.tar`这个文件名是自定义的，只是习惯上我们使用`.tar`作为包文件。

### 打包并压缩
打包，且压缩。`-z`参数表示以`.tar.gz`或者`.tgz`后缀名代表`gzip`压缩过的`tar`包。
```
$ tar -zcvf test.tar.gz 20200323.log
```

打包，且压缩。`-j`参数表示以`.tar.bz2`后缀名作为`tar`包名。
```
$ tar -jcvf test.tar.bz2 20200323.log
```
### 查看包内容
```
$ tar -ztvf test.tar.gz
```
因为使用`gzip`命令压缩的`test.tar.gz`，所以查看压缩包时需要加上`-z`参数。

> 如何只解压部分文件？

```
$ tar -ztvf test.tar.gz 20200323.log
```
这种方式仅限于取一个文件。

### 解压
在该目录下直接解压：
```
$ tar -zxvf test.tar.gz 
```

解压至指定文件夹：
```
$ tar -zxvf test.tar.gz -C log
$ ls log
20200323.log
```

## gzip
`.gz`压缩包（不带tar），需要使用gzip 命令去解压。

```
gzip test.gz -d /<filename>
```
`-d` 参数用于指定解压位置

## 杂项
如何查看Linux 的发行版本？
```
$ lsb_release -a
```
## crontab
crontab 命令被用来提交和管理用户的需要周期性执行的任务，与windows下的计划任务类似。

## w
w命令用于显示已经登陆系统的用户列表，并显示用户正在执行的指令。

不带任何参数，会显示当前登入系统的所有用户
```
$ w
 10:54:39 up 14 days, 22:39, 2 users,  load average: 0.18, 0.09, 0.08
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
wangyh   pts/0    113.87.129.118   10:27   25:10   2.77s  2.76s top
Boo      pts/1    113.87.129.118   10:54    1.00s  0.00s  0.00s w
```
第一行显示的字段信息分别是：
- [x] 10:50:39：系统当前时间
- [x] up 2:02： 系统已运行时间
- [x] 2 user：当前在线用户个数
- [x] load average：系统的平均负载，3个数值分别对应系统在过去的1,5,10分钟内的负载程度，数值越大，表明系统的负载越大。

第二行几个字段分别表示：
- [x] USER ： 登陆用户的账户名
- [x] TTY： 用户登陆所使用的终端
- [x] FROM： 显示用户从何处登陆，用户的IP地址
- [x] LOGIN@：显示用户登陆入系统时的时间
- [x] IDLE：用户空闲时长，从上一次该用户的任务结束后开始计时，以hour为单位
- [x] JCPU：表示在某段时间内，当前用户所有的进程任务所消耗的CPU时间
- [x] PCPU：表示在某段时间内，当前用户正在执行的进程任务所消耗的CPU时间
- [x] WHAT：表示用户正在执行的任务

## who
who 命令用于查看目前登入系统的用户信息，与`w`命令类似。

显示当前登入系统中的所有用户信息
```
$ who
wangyh   pts/0        2019-04-19 10:27 (113.87.129.118)
Boo      pts/1        2019-04-19 10:54 (113.87.129.118)
```
#### 常用参数
`-m`：效果等同于执行`whoami`命令
`-q或--count`：只显示登入系统的帐号名称和总人数；
`-H`：增加显示用户信息状态栏

## last
last 命令用于查看用户最近的登入信息

输出最后10 条登入信息
```
$ last -3
Boo      pts/1        113.87.129.118   Fri Apr 19 10:54   still logged in
wangyh   pts/0        113.87.129.118   Fri Apr 19 10:27   still logged in
wangyh   pts/5        113.87.129.118   Fri Apr 19 10:24 - 10:27  (00:02)
```

查看指定用户的登入信息
```
$ last Boo -3
Boo      pts/1        113.87.129.118   Fri Apr 19 10:54   still logged in
Boo      pts/4        113.87.129.118   Fri Apr 19 10:23 - 10:26  (00:03)
Boo      pts/4        113.87.129.118   Fri Apr 19 10:14 - 10:22  (00:08)
```

## pkill
pkill命令可以按照进程名杀死进程，可以用于踢出当前登入系统的用户。


### 安全的踢出用户
可以使用`pkill`命令踢出当前正登入系统中的用户，但是这么做很危险，更好的解决办法是：
先查看终端号，然后查看该终端执行的所有进程，根据进程号来停止服务。
```
$ ps -ef| grep pts/0
$ kill -9 pid
```

## passwd
passwd 命令用于设置用户的认证信息，包括用户密码、密码过期时间等。

系统管理者则能用它管理系统用户的密码。只有管理者可以指定用户名称，一般用户只能变更自己的密码。

## ss
ss 命令用来显示处于活动状态的套接字信息。ss 命令可以用来获取socket 统计信息，它可以显示和netstat 类似的内容。但ss 的优势在于它能够显示更多更详细的有关TCP 和连接状态的信息，而且比netstat 更快速更高效。

显示所有的tcp 套接字
```
$ ss -t -a
```

显示Socket 摘要
```
$ ss -s
```

列出所有打开的网络连接端口
```
$ ss -l
```

找出打开套接字/端口应用程序
```
$ ss -pl | grep 6666
```

## 知识扩展
存放用户信息：
```
$ cat /etc/passwd
$ cat /etc/shadow
```
用户信息文件分析（每项用:隔开）：
```
jack:X:503:504:::/home/jack/:/bin/bash
jack　　//用户名
X　　//口令、密码
503　　//用户id（0代表root、普通新建用户从500开始）
504　　//所在组
:　　//描述
/home/jack/　　//用户主目录
/bin/bash　　//用户缺省Shell
```

存放组信息：
```
cat /etc/group
cat /etc/gshadow
```
用户组信息文件分析：
```
jack:$!$:???:13801:0:99999:7:*:*:
jack　　//组名
$!$　　//被加密的口令
13801　　//创建日期与今天相隔的天数
0　　//口令最短位数
99999　　//用户口令
7　　//到7天时提醒
*　　//禁用天数
*　　//过期天数
```

如果是普通用户执行passwd只能修改自己的密码。如果新建用户后，要为新用户创建密码，则用passwd用户名，注意要以root用户的权限来创建。
```
# 修改boo 用户的密码
$ passwd boo
```

### 参考链接：
* [Wget 命令](https://www.cnblogs.com/peida/archive/2013/03/18/2965369.html#4235146)
* [SCP 命令](http://man.linuxde.net/scp)
* [last 命令](http://man.linuxde.net/last)
* [who 命令](http://man.linuxde.net/who)
* [pkill 命令](http://man.linuxde.net/pkill)
* [ss 命令](http://man.linuxde.net/ss)
* [permission denied,please try again](https://blog.csdn.net/kinggaiwusi/article/details/76919854)
