---
title: Linux 如何生成密钥对进行 ssh 免密登录
date: 2020-08-29 11:46:41
tags: ["SSH", "SSHD", "Linux"]
categories: ["Linux"]
---

最近因为项目快要上线了，服务器从测试环境转到了生产环境，登录方式也从原来的密码认证替换成了密钥认证。

<!-- more -->

这么做的目的是为了防止服务器密码被暴力破解。

> ssh 是什么？

ssh 是一种协议，它可以基于密码进行认证，也可以基于密钥去认证用户。

## 生成密钥对
这里我们使用 `RSA` 类型的加密类型来创建密钥对。

```
ssh-keygen -f ~/.ssh/your_key_name
```
1. `-f` 参数表示指定密钥对生成位置与名称
2. 密钥对通常放在 `$HOME/.ssh` 目录下
3. 回车即可创建密钥对，如果不需要为密钥对进行加密，那么可以一路回车。

创建成功之后，可以看到 `.ssh` 目录下多了两个文件，分别是：
* `your_key`：密钥对的私钥，通常放在客户端。
* `your_key.pub`：密钥对中的公钥，通常放在服务端。

## 将本地的公钥传到服务器上

注意：这里是将`your_key.pub` 公钥文件上传至你需要连接的服务器，而不是`your_key`私钥文件。
```
ssh-copy-id -i ~/.ssh/your_key.pub user@<ip address> -pport
```
`-i` 参数表示使用指定的密钥，`-p`参数表示指定端口，ssh 的默认端口是 22，如果没有更改默认端口，则可以省略。

这里需要输入一次密码进行确认，如果成功之后，会看到以下内容：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200829105200.png)

> 本地的公钥文件上传在服务器的哪里？

在该用户的`.ssh/authorized_keys` 文件中。
```
cat ~/.ssh/authorized_keys
```

## 通过密钥对进行免密登录

现在我们可以使用以下命令登录到服务器中了：
```
ssh -p port -i ~/.ssh/your_key user@<ip address>
```
不出意外，就可以不用输入密码而直接成功登录了。

如果你仍然需要输入密码或者遇到其他问题了，可以从以下方向进行排查。

### 常见问题：
1. 如果没有使用默认的密钥名称（id_rsa），则在连接主机时需要加上`-i` 参数，指定对应密钥的名称。否则由于默认私钥与远程主机中的自定义公钥不匹配，自然无法基于密钥进行认证，会再次提示你输入密码。
2. 服务端的`$HOME/.ssh`目录的正常权限是700，服务端`$HOME/.ssh/authorized_keys`文件的权限默认为600。
3. 上传密钥时使用的是：公钥（.pub），进行密钥认证时使用的是：私钥。

### 配置ssh config
上面的命令虽然可以实现免密登录，但是命令太长了，就算是复制粘贴也有可能会出错。

那有没有什么好的办法，解决这个问题呢？

当然是有的啦。

在`$HOME/.ssh` 目录下，创建一个名为`config`的文件。
```
vim $HOME/.ssh/conifg
```

加入以下配置：
```
Host alias
    User user
    HostName ip address
    Port port
    IdentityFile ~/.ssh/your_key
    ServerAliveInterval 360
```
参数说明：
* Host：可以理解成别名，配置完成之后，最后就通过 `ssh alias` 进行登录。
* User：远程主机的用户名称
* HostName：远程主机的地址
* Port：端口号
* IdentityFile：私钥文件的路径
* ServerAliveInterval：保持客户端与服务端会话在短时间内不会断开。

当然，如果你是使用`ssh 客户端`，那就不用配置这些。

### 禁用通过密码认证
如果上面的配置都无误，可以正常通过密钥进行免密登录，那么最后需要做的一件事情就是关闭服务端的通过密码进行身份认证。

```
vim /etc/ssh/sshd_config

# 将yes 改为 no
PasswordAuthentication yes
```

然后重启 sshd 服务。
```
service sshd restart
```

以上就是有关如何用自定义的密钥对进行免密认证的全部过程了。