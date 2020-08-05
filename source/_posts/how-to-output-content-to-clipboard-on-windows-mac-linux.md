---
title: how to output content to clipboard on windows/mac/linux
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