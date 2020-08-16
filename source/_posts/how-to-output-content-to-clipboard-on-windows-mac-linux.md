---
title: Windows/Mac/Linux 如何将内容输出到剪贴板
date: 2020-08-05 10:07:41
tags: ["Shell"]
categories: ["Shell"]
---

如何将输出直接复制至剪切板？在不同的系统中，所使用的命令是不同的。

<!-- more -->

### Mac
```
// 将输出复制至剪贴板
$ echo "hello mac" | pbcopy

// 将文件中的内容全部复制至剪贴板
$ pbcopy < remade.md

// 将剪切板中的内容粘贴至文件
$ pbpaste > remade.md
```

### Linux
Linux 用户需要先安装 `xclip`，它建立了终端和剪切板之间的通道。

```
// 查看剪切板中的内容
$ xclip -o
$ xclip -selection c -o

// 将输出复制至剪贴板
$ echo "hello xclip" | xclip-selection c

// 将文件中的内容全部复制至剪贴板
$ xclip -selection c remade.md

// 将剪切板中的内容粘贴至文件
$ xclip -selection c -o > remade.md
```


或者直接使用`xsel`命令：
```
// 将输出复制至剪贴板
$ echo "hello linux" | xsel

// 将文件中的内容全部复制至剪贴板
$ xsel < remade.md
```

### Windows
```
// 将输出复制至剪贴板
$ echo "hello windows" | clip

// 将文件中的内容全部复制至剪贴板
$ clip < remade.txt
```