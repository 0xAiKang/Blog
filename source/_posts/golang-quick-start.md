---
title: Go 语言学习笔记
date: 2020-12-08 23:53:16
tags: ["Golang"]
categories: ["Golang"]
---

最近开始学习Go 语言，记录一下学习笔记，具体可以访问[Go 语言学习笔记](https://github.com/0xAiKang/go_learning_note)

<!-- more -->

### 切片
slice 的本质是一个数据结构，实现了对数组操作的封装。

go 提供了一种类似“动态数组”结构的数据类型，这种类型就是切片。

声明
```
// 语法
var identifier []type

// 声明一个为 int64 类型的切片 
var slice []int64
```

初始化：
```
// 初始化一个 int64 类型的切片
array = make ([]int64, 10)

// 初始化数组
array = [10] int64 {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

#### 切片常见操作

对元素进行添加：
```

```

数组和切片存在一些区别：
* 声明数组时，是需要指定长度，而切片不用指定长度。
* 初始化操作不一样。