---
title: Git 变基命令详解
date: 2020-08-24 20:06:20
tags: ["Git"]
categories: ["Git"]
---

“变基”命令是git 常用命令中，比较冷门的，一方面是因为这个命令比较“危险”，如果用不好，很有可能会导致代码丢失。另一方面是因为这个命令不像 add、commit、pull、push 属于必须要执行的命令，就算不用它，也能干活。

<!-- more -->

## 场景重现

问题描述：有时候我们在本地提交完代码，下一个操作是需要推送到远程仓库，这时如果远程仓库已经有了更新的提交，那么当我们执行完`git push` 命令之后，不出意外会出现以下错误：

```
! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@gitlab.com:invest2/invest_home.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

这时错误的意思是：推送失败，你需要先将远程仓库最新的提交更新到本地仓库，然后才能 `git push`。

所以这个时候你有两个选择：
1. 使用`git pull` 自动合并
2. 使用`git fetch` 手动合并

前者虽然用起来很方便，但是自动合并会留下一次合并记录，类似这样：
```
Merge branch 'master' of bitbucket.org:maxt2013/invest_home
```

虽然这并不会影响什么，但如果你很重视 `commit logs`，那么这样的一次记录，是不被容忍的。

后者通过手动合并，确实可以做到没有多余的合并记录，但是每次手动合并有比较麻烦，那么有没有什么折中的方式，既可以不留下多余的记录，有比较省事。

答案是有的，它就是我们下面要介绍的“变基”。

## rebase
下面这条命令会将远程仓库中最新的提交合并到本地仓库，`--rebase`参数的作用是先取消 commit 记录，并把它们临时保存为补丁（patch），这些补丁放在 `.git/rebase`目录中，等远程仓库同步至本地之后，最后才将补丁合并到本地仓库。

```
git pull --rebase origin master
```

下面用图来解释具体发生了什么。

`git pull` 之前的情况：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200823134641.png)

使用 `git pull --rebase origin`：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200823134808.png)

最后使用 `git push`：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200823134934.png)

### 总结
如果你对 `commit logs`有强烈的控制欲望，那么变基命令是适合你的，如果你是使用git 的新手，或者你不在意 `commit logs`，那么直接使用 `git pull` 自动合并就好了。

### 参考链接
* [git push错误failed to push some refs to的解决](https://blog.csdn.net/MBuger/article/details/70197532)

