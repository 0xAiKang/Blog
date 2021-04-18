---
title: 测试文章
date: 2021-04-18 13:18:21
tags: 
---

刚拿到一台服务器时，通常会禁用root 用户登录，而使用其他普通用户，这时就需要创建一个新用户。

<!-- more -->

## 添加用户

创建一个新用户：
```
$ useradd boo
```

设置密码：
```
$ passwd boo
```
## 提权

此时此用户已经可以正常使用了，但是还没有提权，所以很多事情做不了，这时可以把该用户加入`sudo` 用户组，通过`sudo`命令来进行提权。

```
$ usermod -G sudo boo
```

一般直接就加入成功了，但是有些发行版本默认并没有`sudo`用户组，所以这时需要先添加用户组。

```
$ groupadd sudo
```

手动添加完用户组之后，还需要修改`sudoers`配置文件，这里有几种方式，根据实际情况进行选择：
1. 允许`sudo` 组的成员执行任何命令

```
$ sudo visudo // 或者 sudo vim /etc/sudoers

// 添加以下内容
# the 'sudo' group has all the sudo privileges
%sudo ALL=(ALL:ALL) ALL 
```

2. 直接允许该用户执行任何命令
```
$ sudo visudo 

// 添加以下内容，注意：没有% 
# Allow boo to run any commands anywhere
boo ALL=(ALL:ALL) ALL 
```

通常还是建议将用户添加至`sudo` 用户组，然后赋予`sudo` 组成员权限，而不是直接对具体某个用户进行提权。

## 总结

查看所有用户的列表：
```
$ cat /etc/passwd
```

查看所有用户组：
```
$ cat /etc/group
```

查看当前登入用户的组：
```
$ groups
```

查看指定用户所在的组：
```
$ groups usernmae
```

添加用户：
```
$ useradd username
```
设置(重置)密码：
```
$ passwd username
```
添加用户组：
```
$ groupadd group_name
```

将某个用户添加到某个组：
```
$ usermod -G group_name username
```

编辑sudoers 配置文件：
```
$ sudo visudo
```


