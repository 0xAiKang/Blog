---
title: Git Pull 命令详解
date: 2020-08-17 23:34:26
tags: ["Git"]
categories: ["Git"]
---

这片文章主要用来讲解`git pull`命令的一些细节。

<!-- more -->

## git pull

git pull 的作用是：取回远程主机某个分支的更新，再与本地指定分支自动合并。

### 描述

将远程主机中的更改合并到当前分支，在默认情况下`git pull`是`git fetch`命令和`git merge Fetch_HEAD`命令的合集，后面会详细介绍。

### 示例

这是git pull 的完整格式：
```
$ git pull [options] [<repository> [<refspec>…]] 
```

比如要取回``origin``主机的``fixbug``分支的最新提交，**并与本地的``master``分支合并**，就需要写成这个样子：
```
$ git pull origin fixbug:master 
```

如果远程分支要与当前分支合并，则冒号及其冒号后的分支可以省略，就变成了这个样子：
```
// 取回firebug 分支的最新提交并与当前分支合并
$ git pull origin fixbug 
```

上面的命令表示，取回``origin/fixbug``分支最新的提交，并于当前分支合并。

这里就等同于先``git fetch``获取所跟踪的远程分支的最新的提交，然后执行``git merge ``合并到当前分支。也就是下面两条命令。
```
// 自动从当前分支的跟踪分支上获取最新的提交
$ git fetch 

// 合并origin/fixbug分支到当前分支
$ git merge origin/fixbug
```

## git fetch

> 为什么这个分支是这种写法?

因为`git fetch`命令会获取当前追踪分支的最新更改，就等同于取回`origin/fixbug`分支到本地。

你可以使用``git branch -a`` 查看所有分支，会发现多了一个 `origin/fixbug`分支，前提是该分支已经建立了追踪关系。

而这个分支所包含的内容就是最新的提交或者其他某些更改。所以此时你需要通过合并这个长的比较奇怪的分支，来更新本地的工作区。

在某些场合，Git 会自动在本地分支与远程分支之间建立一种追踪关系（tracking）。比如，我们在clone 时，会发现所有本地分支默认与远程主机的同名分支，建立追踪关系。也就是说，本地的 master 分支自动追踪 `origin/master `分支。

Git 也允许手动添加追踪关系。
```
// 本地master分支与取回origin/fixbug分支建立关系。
$ git branch --set-upstream master origin/fixbug

```

如果当前分支与远程分支存在追踪关系。那么git pull 就可以省略远程分支名。
```
$ git pull origin
```

上面的分支是什么意思呢？就是表示本地的当前分支会自动与对应的``origin``主机的“追踪分支”进行合并。

如果当前分支只对应一种追踪分支，那么远程主机名都可以省略。
```
// 这也就成了我们常看见的原始命令。
$ git pull
```
上面的命令会自动的与唯一的追踪分支进行合并。

> 如何将远程分支作为本地的默认分支？

```
$ git branch --track <remote branch> remotes/origin/<remote branch>
```

这样就将远程的分支与本地同名分支建立了追踪关系。

可以使用`git config -e`命令查看。

当追踪关系只有一个时，那么使用`git pull` 命令，就可以直接更新`<remote branch>` 分支了。

如果合并需要采用``rebase``模式，可以使用``--rebase``选项。

## git rebase

> 这里说一个题外话，``rebase`` 是什么？有什么用？

`git rebase` 清除本地历史提交

```
$ git --rebase <远程主机名><远程分支名>:<本地分支名>
```

> git fetch 与 git pull 的区别。

git fetch 表示从远程获取最新的版本到本地，但是不会自动合并。其过程用命令表示就是：
```
$ git fetch origin master
$ git log -p master..origin/master
$ git merge origin/master
```

另一种写法就是：      
```
$ git fetch origin master:tem
$ git diff tem
$ git merge tem
```
上面这两种写法都是都是一个意思。唯一有所区别的就是使用 ``tem``分支代替了``origin/master``分支的存在。其含义是： 
1. 从远程``origin``主机的``master``主分支下载最新的版本到本地``origin/master``分支，或者``tem``分支。
2. 比较本地master分支与origin/master（tem）分支的差异。
3. 最后进行合并

git pull，相当于从远程获取最新的版本并合并到本地。       

```
$ git pull origin master 
```
上述命令其实相当于git fetch 和 git merge
在实际使用中，git fetch更安全一些，因为在merge前，我们可以查看更新情况，然后再决定是否合并。