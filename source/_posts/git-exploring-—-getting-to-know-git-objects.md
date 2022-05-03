---
title: .git 探秘 — 认识 Git 对象
date: 2022-05-03 10:42:20
tags: ["Git"]
categories: ["Git"]
---

从根本上来讲git是一个内容寻址（content-addressable）文件系统，并在此之上提供了一个版本控制系统的用户界面。

<!-- more -->

所有的 git 仓库的根目录下面都有个 `.git` 文件, 它默认是隐藏的，`.git` 文件夹结构如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220502145745.png)

各文件里面存储的内容：

| 文件夹|类型|作用|
| ------- | ------- | ------- |
| hooks | 文件夹 | 包含客户端或服务端的钩子脚本; 我们最常用的就是pre-commit钩子了|
| info |  文件夹 | 包含一个全局性排除文件 |
| logs | 文件夹 | 保存日志信息; git reflog 展示的内容 |
| objects | 文件夹 | 目录存储所有数据内容, 这就是实际意义上的 git数据库, 存数据的地方; 并且存了所有的历史记录 |
| refs | 文件夹 | 目录存储指向数据（分支、远程仓库和标签等）的提交对象的指针|
| config | 文件 | 包含当前项目特有的配置选项|
| description |  文件 | 用来显示对仓库的描述信息,文件仅供 GitWeb 程序使用，我们无需关心|
| HEAD | 文件 | 它文件通常是一个符号引用（symbolic reference），指向目前所在的分支。某些罕见的情况下，HEAD 文件可能会包含一个 git 对象的 SHA-1 值 |
| index | 文件夹 | 文件保存暂存区信息 |
| FETCH_HEAD | 文件 | git fetch; 这将更新git remote 中所有的远程repo 所包含分支的最新commit-id, 将其记录到.git/FETCH_HEAD文件中|
| packed-refs | 文件 | 对refs打包后(git gc)的存储文件, 与底层命令git pack-refs |

所有的数据对象均存储于项目下面的 `.git/objects` 目录中，那么`.git/objects` 文件夹里面究竟存了些什么？

## Git 对象
版本库中的每一个文件，不论是图片、源文件还是二进制文件，都被映射为一个 Blob 对象。除了 Blob 对象，在 Git 的文件系统中还存储着另外三种数据对象：
1. 树对象(tree)
2. 提交对象(commit)
3. 标签对象(tag)

### Blob 对象
Blob 是英文 Binary large object 的缩写，一个 Blob 对象就是一段二进制数据。

```bash
$ echo -n "print 'hello git'" > index.md
$ git add index.md
$ find .git/objects -type f
.git/objects/pack/pack-d9624c66acb04fcc36857b02d1d36590efb8d18f.pack
.git/objects/pack/pack-d9624c66acb04fcc36857b02d1d36590efb8d18f.idx
.git/objects/42/a3276feefd4f52d48aa831db535d6e81b6e0fb
```

查看数据对象的类型：
```bash
git cat-file -t 42a3276feefd4f52d48aa831db535d6e81b6e0fb
blob
```

查看数据对象的内容：
```bash
git cat-file -p 42a3276feefd4f52d48aa831db535d6e81b6e0fb
print 'hello git'%
```

为了把文件映射为 Blob 对象，Git 做了下面这些工作：
1. 读取文件内容，添加一段特殊标记到头部，得到新的内容，记为 content；
2. 对该 content 执行 SHA-1 加密，得到一个长度为40字符的 hash 值，例如 64fe72272a79bff953d7de2062d3f52b4679c659；
3. 取该 hash 值的前两位作为子目录，剩下的38位作为文件名，在本例中，子目录名是'64/'，文件名是'fe72272a79bff953d7de2062d3f52b4679c659'；
4. 对 content 执行 zip 压缩，得到新的二进制内容，存入文件中。

### Tree 对象
Git 使用一种与 UNIX 文件系统相似的方式来管理内容，Blob 相当于磁盘文件，Tree 则相当于文件夹。Tree 中既可以包含 Blob，也可以包含其他 Tree。

向版本库中提交当前的修改：
```bash
git commit -m "first commit"
```

会发现 `.git/objects` 目录下面多出了两个对象：

```bash
find .git/objects -type f
.git/objects/93/292574c965d7ecd25f933fdadb646dce75cc24
.git/objects/pack/pack-d9624c66acb04fcc36857b02d1d36590efb8d18f.pack
.git/objects/pack/pack-d9624c66acb04fcc36857b02d1d36590efb8d18f.idx
.git/objects/42/a3276feefd4f52d48aa831db535d6e81b6e0fb
.git/objects/30/31cddff73840bd5e7822c9b7f3b538e7e160bb
```

这两个对象的类型分别是 commit 和 tree：
```bash
git cat-file -t 93292574c965d7ecd25f933fdadb646dce75cc24
tree

git cat-file -t 3031cddff73840bd5e7822c9b7f3b538e7e160bb
commit
```

查看 93292574c965d7ecd25f933fdadb646dce75cc24 这个对象的内容：
```bash
git cat-file -p 93292574c965d7ecd25f933fdadb646dce75cc24
...
100644 blob 42a3276feefd4f52d48aa831db535d6e81b6e0fb	index.md
...
```

可见这颗树就相当于项目的根目录。

### Commit 对象
一个 Commit 对象代表了一次提交对象，它包含了下面这些信息：
- 何人何时作了该次提交
- 该次提交的简略说明
- 一棵树
- 父级 Commit 对象

其中，这颗树也被称作项目快照（snapshort），通过项目快照，我们可以把项目还原成项目在该次提交时的样子。一般来说，commit 对象总有一个父级 commit 对象，一个又一个 commit 对象通过这种方式链接起来，就构成了一条提交历史。第一次提交的 commit 对象没有父级 commit 对象，分支合并所产生的新的 commit 对象可以有两个或者多个父级 commit 对象。

例如，3031cddff73840bd5e7822c9b7f3b538e7e160bb 这个 Commit 对象的内容为：
```bash
git cat-file -p 3031cddff73840bd5e7822c9b7f3b538e7e160bb
tree 93292574c965d7ecd25f933fdadb646dce75cc24
parent 3c46f2d56ee15502cabeda97d6f44a95289c8804
author Boo <aikangtongxue@gmail.com> 1651542437 +0800
committer Boo <aikangtongxue@gmail.com> 1651542437 +0800

first commit
```

此时版本库中，Commit、Tree、Blob 三者之间的关系如下图所示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220503103534.png)
### Tag 对象
Tag 指向一次特征提交。

在 Git 中有两种 tag，第一种 tag 并不在 `.git/objects` 目录下面创建新的对象，只是在 `.git/refs/tags` 目录中新建一个文件，文件的内容就是所指向的 commit 对象的 hash 值：

```bash
$ git tag v0.1
$ find .git/refs/tags -type f
.git/refs/tags/v0.1
$ cat .git/refs/tags/v0.1
3031cddff73840bd5e7822c9b7f3b538e7e160bb
```

另一种 tag 则会在 `.git/objects` 目录下面创建对象，这种 tag 被称作注解标签（annotated tag）：
```bash
$ git tag -a v0.2 -m "Version 0.2"
$ find .git/objects -type f
.git/objects/93/292574c965d7ecd25f933fdadb646dce75cc24
.git/objects/pack/pack-d9624c66acb04fcc36857b02d1d36590efb8d18f.pack
.git/objects/pack/pack-d9624c66acb04fcc36857b02d1d36590efb8d18f.idx
.git/objects/42/a3276feefd4f52d48aa831db535d6e81b6e0fb
.git/objects/30/31cddff73840bd5e7822c9b7f3b538e7e160bb
.git/objects/cd/542f02e1b07b78a11534c68b1d334365effc65
```

查看数据对象类型及内容：
```bash
$ git cat-file -t cd542f02e1b07b78a11534c68b1d334365effc65
tag
$ git cat-file -p cd542f02e1b07b78a11534c68b1d334365effc65
object 3031cddff73840bd5e7822c9b7f3b538e7e160bb
type commit
tag v0.2
tagger Boo <aikangtongxue@gmail.com> 1651543206 +0800

Version 0.2
```

## 总结
在 Git 的底层，有四种数据结构，它们分别是：

- Blob
- Tree
- Commit
- Tag

Git 把版本库中的每一个文件都转换为一个 blob 对象进行存储，而用 tree 对象来表达文件的层次结构。

Commit 对象代表了一次提交操作，它包含了当前的项目快照以及提交人和提交日期等诸多信息。所有的 commit 对象串接起来，组成一个有向无环图。从版本控制的角度看，这些 commit 对象构成了一个完整的版本提交记录；从项目开发的角度看，它们描述了项目是如何从无到有一点一滴地构建起来的。

Tag 对象指向一个 commit 对象，我们可以通过 tag 对象快速访问到项目的某一次特征提交。

## 参考链接
* [Git 之术与道 -- 对象](https://www.jianshu.com/p/fa31ef8814d2)
* [Git 内部原理 - Git 对象](https://git-scm.com/book/zh/v2/Git-内部原理-Git-对象)
* [git仓库清理--"保姆级"教程](https://juejin.cn/post/7024922528514572302)