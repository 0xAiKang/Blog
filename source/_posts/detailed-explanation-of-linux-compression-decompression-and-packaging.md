---
title: Linux 压缩、解压、打包详解
date: 2020-08-31 23:47:15
tags: ["Linux"]
categories: ["Linux"]
---

在Linux 中，解压、压缩、打包是日常会很频繁用到的几个操作，但是因为参数很多，没有记忆点，加上压缩文件的类型很多，如果不经常使用，是真的容易忘记。

<!-- more -->

所以这篇笔记就是用来整理常见的那些解压、压缩、打包的命令。

在正式学习之前，需要明确的两个概念，打包和压缩不是一回事：
* 打包：是指将一大堆文件或目录变成一个总的文件。
* 压缩：则是将一个大文件通过压缩算法变成一个小文件。

为什么要区分这两个概念呢？这源于Linux 中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）。

## tar
### 压缩/打包

仅打包，不压缩。
```
tar -cvf foo.tar foo
```

`foo.tar`这个文件名是自定义的，只是习惯上我们使用 `.tar` 作为包文件。

打包，且压缩。`-z `参数表示以 `.tar.gz` 或者 `.tgz` 后缀名代表 gzip 压缩过的 tar 包。
```
tar -zcvf foo.tar.gz foo
```

打包，且压缩。`-j` 参数表示以 `.tar.bz2` 后缀名作为tar包名。
```
tar -jcvf foo.tar.gz foo
```

### 解压
在当前目录下直接解压：
```
tar -zxvf foo.tar.gz
```
注意，如果这个目录下有同名的文件，不会询问，直接覆盖。

解压至指定文件夹：
```
tar -zxvf foo.tar.gz -C <dir name>
```

## gzip
gzip 命令用来压缩文件。文件经它压缩过后，其名称后面会多处 `.gz` 扩展名（不带 `.tar`）。

### 压缩
将当前目录的每个文件压缩成`.gz`文件：
```
gzip *
```

递归压缩指定目录的所有文件及子目录：
```
gzip -r <dir name>
```

### 解压

解压当前目录下的`foo.gz` 文件：
```
gzip -d foo.gz
```
解压完成之后，`foo.gz` 就变成了 `foo` 文件。

递归解压目录：
```
gzip -dr <dir name>
```
解压完成之后，`<dir name>` 目录下的所有 `.gz` 文件都会变成正常文件。

## zip
`zip` 可以用来解压缩文件，或者对文件进行打包操作。文件经它压缩后会另外产生具有 `.zip` 扩展名的压缩文件。

### 压缩

将当前目录下的指定目录，压缩为 `.zip`文件：
```
zip -q -r foo.zip <dir name>
```

将指定目录下的所有文件及其文件夹，压缩为`.zip` 文件：
```
zip -q -r foo.zip /<path to dir>
```
注意，产生的压缩文件在执行命令的那个目录下。

### 解压
unzip 命令用于解压缩由 zip 命令压缩的 `.zip`压缩包。

查看压缩包内容：
```
unzip -v foo.zip
```

将压缩文件在指定目录下解压缩，如果已有相同的文件存在，要求 unzip命令不覆盖原先的文件。

```
unzip -n foo.zip -d /<file to dir>
```

将压缩文件在当前目下解压，如果已有相同的文件，不询问，直接覆盖。

```
unzip -o foo.zip
```

### 总结
Linux 下的压缩解压其实并不复杂，只是不常用的情况下，很容器忘记。

如果你不知道在什么场景下，该使用什么命令，可以参照：
* 如果只有一个大文件，可以使用 `gzip` 或者 `zip`命令。
* 如果是一个完整的目录，里面有很多子目录以及文件，可以使用`tar`命令。
