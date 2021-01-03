---
title: string and encoding
date: 2021-01-03 23:20:47
tags: ["Golang"]
categories: ["Golang"]
---

因为计算机只能处理数字，如果需要处理文本，就需要先将文本转换为数字才能处理。最早的计算机在设计时采用**8个比特（bit）作为一个字节（byte）**，所以，一个字节能表示的最大的整数就是255（二进制 11111111=十进制255 ）。

由于计算机是美国人发明的，因此，最早只有127个字符被编码到计算机里，也就是大小写英文字母、数字和一些符号，这个编码表被称为ASCII编码。

但是要处理中文显然一个字节是不够的，至少需要两个字节，而且还不能和ASCII编码冲突，所以，中国制定了`GB2312`编码，用来把中文编进去。

可以想到的是，全世界有上百种语言，各国有各国的标准，就会不可避免地出现冲突，结果就是编码方式和解码方式不同，就会导致乱码。

因此，Unicode字符集应运而生。Unicode把所有语言都统一到一套编码里，这样就不会再有乱码问题了。

不过新的问题因此又出现了：如果统一成 Unicode 编码，乱码问题虽然是从此消失了，但是，如果你写的文本基本上全部是英文的话，用Unicode编码比ASCII编码需要多一倍的存储空间，在存储和传输上就十分不划算。

所以，本着节约的精神，又出现了把Unicode编码转化为“可变长编码”的UTF-8编码。UTF-8编码把一个Unicode字符根据不同的数字大小编码成1-6个字节，常用的英文字母被编码成1个字节，汉字通常是3个字节，只有很生僻的字符才会被编码成4-6个字节。

ASCII、Unicode和UTF-8 三者的关系是：
1. **Unicode 是一种包含所有语言的字符集编码（替代ASCII编码）**
2. **UTF-8 是 Unicode 的实现方式之一**

### 字符编码在计算机中的工作方式
在计算机内存中，统一使用Unicode编码，当需要保存到硬盘或者需要传输的时候，就转换为UTF-8编码。

用记事本编辑的时候，从文件读取的UTF-8字符被转换为Unicode字符到内存里，编辑完成后，保存的时候再把Unicode转换为UTF-8保存到文件：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210103161913.png)

浏览网页的时候，服务器会把动态生成的Unicode内容转换为UTF-8再传输到浏览器：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210103161935.png)

所以你看到很多网页的源码上会有类似`<meta charset="UTF-8" />`的信息，表示该网页正是用的UTF-8编码。

### Go 语言的字符串
Go 语言的字符串与其他编程语言的差异：
1. string 是数据类型，不是引用或者指针类型（其零值不是空，是一个空字符串）
2. string 是只读的 byte slice，`len`函数获取的是它所包含的 `byte`数
3. string 的 byte 数组可以存放任何数据（二进制）

通过一个实际例子来理解Go 的string、Unicode、UTF8：

```
package main

import "testing"

func TestString(t *testing.T)  {
	var s3 = "中"
	
	// rune 这个数据类型可以取出字符串中的 Unicode 编码
	r := []rune(s3)
	
	// byte 这个数据类型可以取出字符串的 UTF8 存储
	b := []byte(s3)
	
	t.Log(b)			// [228 184 173]
	t.Logf("中 的Unicode 编码：%x", r[0])
	t.Logf("中 的UTF8 存储：%X", s3)		// [0xE4, 0xB8, 0xAD]
}
```

字符 `中` 字在 Unicode 中的编码是`0x4E2D`，它的物理存储形式依赖于 UTF8规则，它在内存被存储为了`E4B8AD`，放在 string 对应的 byte切片中，分别对应三个 byte：`[0xE4, 0xB8, 0xAD]`。

### 其他问题

> 在我们的日常生活中用到的是十进制，计算机用的是二进制，那么为什么还会出现十六进制呢？

这是因为使用二进制表示数据太长了，可读性十分差，正好十六是二的四次方，所以一位十六进制可以表示四位二进制。

### 参考连接
* [字符串和编码——廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1016959663602400/1017075323632896)
* [字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
