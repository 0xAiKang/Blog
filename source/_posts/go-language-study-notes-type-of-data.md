---
title: Go 语言学习笔记——数据类型
date: 2022-10-25 22:18:12
tags: ["Go"]
categories: ["Go"]
---

作为一个站在巨人的肩膀上成长起来的现代编程语言，Go 语言中的数据类型大部分都是由C 语言演变而来的，它继承了前辈语言的优点，又改进了前辈语言中的不足，下面来对比看一下。

<!-- more -->

## C 语言数据类型

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/C语言数据类型.png)
## Go 语言数据类型

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/Go语言数据类型.png)

可以看到 Go 语言的数据类型更丰富，除了基本的数据类型都支持，像 C 语言中没有的字符串类型，Go 语言也原生支持了。

## 基本数据类型
认识预定义基本类型、各自占用字节大小以及默认值，有助于开发跨平台应用时无需过多考虑符号和长度差异：

|类型|长度（字节）|默认值（零值）|说明|
| ------- | ------- | ------- | ------- |
| bool | 1 | false |  |
| byte | 1 | 0 | uint8 |
| int, unit | 4,8 | 0 |  默认整数类型，依据目标平台，32 位或 64 位 |
| int8, uint8 | 1 | 0 | -128 ～ 127， 0～255（byte 是uint8 的别名）|
| int16, unit16 | 2 | 0 | -32768~32767, 0~65535 |
| int32, unit32 | 4 | 0 | -21 亿~21 亿, 0~42 亿（rune 是int32 的别名）|
| int64, unit64 | 8 | 0 | |
| float32 | 4 | 0.0 |  |
| float64 | 8 | 0.0 | 默认浮点数类型 |
| complex64 | 8 |  |  |
| complex128 | 16 |  |  |
| rune | 4 | 0 | Unicode Code Point, int32 |
| uintptr | 4,8 | 0 | 无符号整型，用于存放一个指针 |
| string |  | "" | 字符串，默认值为 空字符串，而非 NULL |
| array |  |  |  数组 |
| struct |  |  |  结构体 |
| function |  | nil |  函数 |
| interface |  | nil | 接口 |
| map |  | nil |  字典，引用类型 |
| slice |  | nil |  切片，引用类型 |
| channel |  | nil |  通道，引用类型 |