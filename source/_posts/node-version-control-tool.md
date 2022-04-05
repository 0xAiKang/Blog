---
title: Node 版本控制工具
date: 2022-04-05 10:47:51
tags: ["Node"]
categories: ["Node"]
---

主流的 Node.js 版本管理工具有 n 和 nvm，两者都是使用 shell 脚本实现，本文会逐一介绍。

## n
[n](https://github.com/tj/n) 是Node的一个模块，作者是TJ Holowaychuk（鼎鼎大名的[Express]框架作者），就像它的名字一样，它的理念就是简单。

### 安装
```bash
sudo npm install -g n
```

### 版本切换

```bash
n x.x.x
```

### 安装某个版本

```bash
n x.x.x
```

### 安装最新版本

```bash
n latest
```

### 安装稳定版本

```
n stable
```

### 删除某个版本

```bash
n rm x.x.x
```

### 指定某个版本来执行文件

```bash
n user x.x.x some.js
```

## nvm
[nvm](https://github.com/nvm-sh/nvm) 全称Node Version Manager。

### 安装
nvm 项目提供了一个 install.sh 脚本帮助用户快速安装，可以使用 cURL 或 Wget 等工具下载。
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

或者

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

运行上面命令中的任意一条，就会下载并运行 v0.39.1 版本的 nvm，默认安装位置为 ~/.nvm，并会在一些配置文件中添加如下代码片段，例如 ~/.bash_profile、~/.zshrc、~/.profile 或 ~/.bashrc。
```bash
export NVM_DIR="([ -z "{XDG_CONFIG_HOME-}" ] && printf %s "{HOME}/.nvm" || printf %s "{XDG_CONFIG_HOME}/nvm")"
[ -s "NVM_DIR/nvm.sh" ] && \. "NVM_DIR/nvm.sh" # This loads nvm
```

下载、编译和安装 node 的最新版本：
```bash
nvm install node  # "node" 是最新版本的别名
```

也可以指定安装版本，例如 v14.17.4
```bash
nvm install 14.17.4
```

如果不知道哪些版本可以正常使用，可以先列出所有可用版本：
```bash
nvm ls-remote
```

### 版本切换

使用指定版本：
```bash
nvm use 14.17.4
```

需要注意的是这个只是临时使用该版本，当环境发生变化之后，node 又会恢复之前的版本。

### 查看正在使用的版本

查看当前所使用的 node 版本：
```bash
nvm current
```

### 查看当前已经安装的版本

```bash
nvm ls
```

### 指定某个版本来执行文件

```bash
nvm run x.x.x some.js
```

### 其他命令

如何使得某个版本变为默认版本：
```bash
nvm alias default v14.17.4
```

获取对应版本的安装路径：
```bash
nvm which 14.17.4
```

需要注意的是，npm 的版本切换，没有具体的命令，只要控制 node 版本就行，npm 版本会随之变化。