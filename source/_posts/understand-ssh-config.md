---
title: 了解 SSH Config
date: 2020-09-02 23:58:02
tags: ["Linux"]
categories: ["Linux"]
---

很早就接触到了SSH，起初并不知道有`ssh config`这样一个东西存在，基本上是摸着石头过河，中间遇到过不少问题，走过不少弯路。

最后总结出来了两个解决办法，今天无意间发现原来其中有一个这么好用的工具一直都被我忽略了。

<!-- more -->

## 什么是SSH Config
> 先决条件：在使用ssh 之前，需要先安装好`Openssh`、`SSH1`或者是`SSH2`。（Linux、Mac用户请忽略）

`~/.ssh/config` 是通过ssh 连接远程服务器时使用的配置文件。

## 为什么要使用SSH Config
例如：使用SSH 进行远程连接，一般会这样做：
```
$ ssh Boo@18.182.201.142
```
在简单地连接情况下，它并不麻烦。但是当端口号不是默认值（22）时，当密钥对不是默认名称时，连接就变得复杂了。
```
# 指定端口连接
$ ssh Boo@18.182.201.142 -p 2222

# 非默认名称密钥认证
$ ssh -i ~/.ssh/id_rsa_aliyun Boo@18.182.201.142

# 以上两种情况综合
$ ssh -i ~/.ssh/id_rsa_aliyun Boo@18.182.201.142 -p 2222
```

此时，使用`ssh config`就变得很有用了。
```
# vim ~/.ssh/config
Host aliyun
    HostName 18.182.201.142
    Port 2222
    User Boo
    IdentityFile ~/.ssh/id_rsa_aliyun
```
现在在连接使用如下命令：
```
$ ssh aliyun
```
是不是非常的方便！就算此时手上有多台服务器需要管理，只要配置好对应的`~/.ssh/config`参数，就可以很轻松的进行连接了。

但需要注意的是：有关ssh 的配置不能分成多个文件，只能写在这一个文件中`~/.ssh/config`（如果你有更好的办法）。

SSH 的配置文件同样适用于其他程序，如：`scp`，`sftp`等。

## 常用的配置选项
### 配置文件格式
* 空行和以'＃'开头的行是注释。
* 每行以关键字开头，后跟参数。
* 配置选项可以用空格或可选的空格分隔，只需要一个=。
* 参数可以用双引号（"）括起来，以指定包含空格的参数。

### 常用关键字
SSH Config 的关键字不区分大小写，但是参数区分大小写。

- [x] Host：可以理解为远程主机名的别名，最终指明这个名称进行连接，如：`ssh aliyun`
- [x] HostName：需要远程连接的主机名，通常都是IP。
- [x] Port：指定连接端口
- [x] User：指定连接用户
- [x] IdentityFile：指明远程连接密钥文件

> 注：Host 关键字可以包含以下模式匹配：

* `*`- 匹配零个或多个字符。例如，Host *将匹配所有主机，同时`192.168.0.*`匹配`192.168.0.0/24`子网中的所有主机。
* ? - 恰好匹配一个字符。该模式Host `10.10.0.?`将匹配`10.10.0.[0-9]`范围内的所有主机。
* !- 在模式的开头将否定其匹配例如，Host `10.10.0.*` `!10.10.0.5`将匹配`10.10.0.0/24`子网中的任何主机，除了`10.10.0.5`。

#### 配置文件加载顺序
* 全局配置文件：`/etc/ssh/ssh_config`

* 用户配置文件：`~/.ssh/config`

ssh 客户端按以下优先顺序读取其配置：
1. 从命令行指定的选项
2. 用户的ssh 配置文件
3. 全局的ssh 配置文件

如果希望SSH 客户端忽略ssh 配置文件中指定的所有选项，可以使用：
```
$ ssh -F user@example.com
```

## 恢复连接
常用SSH 的小伙伴可能都知道，使用SSH 连接到远程服务器之后，如果一段时间没有输入任何指令，很有可能会断开与服务器的连接，需要重连就会变得很麻烦。

此时，ssh config 又变得很有用了。
```
＃定期向服务器发送实时报告（每60秒，可以自定义）
ServerAliveInterval 60

# 如果想要针对某个连接单独使用，需要放在Host 指令下，全局则放在最头部
Host aliyun
    ServerAliveInterval 60
```

### 可能感兴趣的内容
* [如何在Linux 中更改SSH 端口](https://linuxize.com/post/how-to-change-ssh-port-in-linux/)
* [关于 ~/.ssh/config](https://qiita.com/passol78/items/2ad123e39efeb1a5286b#sshconfig%E3%81%A8%E3%81%AF)
* [使用 SSH 配置文件](https://linuxize.com/post/using-the-ssh-config-file/)
* [ssh_config 指令详解](https://man.openbsd.org/OpenBSD-current/man5/ssh_config.5)
* [SSH 官网](https://www.ssh.com/)
* [OpenSSH 客户端SSH 配置文件](https://www.ssh.com/ssh/config/)
