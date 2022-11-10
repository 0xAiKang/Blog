---
title: Go 语言学习笔记——unsafe.Sizeof
date: 2022-10-29 22:10:51
tags: ["Go"]
categories: ["Go"]
---

在 Go 语言中，`len` 函数可以用于获取一个变量的长度，`unsafe.Sizeof` 函数用于获取一个数组变量的总大小。

<!-- more -->

`len` 函数比较简单好理解，使用过程中基本上不会遇到问题，这篇笔记主要介绍 `unsafe.Sizeof`。

## 一个问题

先来看看这段示例代码：
```go
package main

import (
	"fmt"
	"unsafe"
)

type run interface {
}

type Person struct {
	Name   string
	Age    int
	Gender int
	//Height int
}

type Empty struct {
}

func structFunc() {
	person1 := Person{
		Name:   "yumi",
		Age:    23,
		Gender: 0,
		//Height: 165,
	}

	// 结构体占用内存大小取决于组成结构体的字段大小之和，因为空结构体没有字段，所以总大小是零。
	empty := Empty{}
	fmt.Println("空结构体的总大小是：", unsafe.Sizeof(empty)) // 0

	fmt.Println("初始化结构体的总大小是：", unsafe.Sizeof(person1)) // 32

	person2 := Person{}
	fmt.Println("零值初始化的结构体的总大小是：", unsafe.Sizeof(person2)) // 32
	fmt.Println("------------------------")
}

func interfaceFunc() {
	var interface1 interface{}
	fmt.Println("空接口的总大小是：", unsafe.Sizeof(interface1)) // 16

	var interface2 run
	fmt.Println("接口的总大小是：", unsafe.Sizeof(interface2)) // 16
	fmt.Println("------------------------")
}

func pointerFunc() {
	var ep int

	i := "123"
	ip := &i

	s := "hello"
	sp := &s

	fmt.Println("空指针的总大小是：", unsafe.Sizeof(ep))       // 8
	fmt.Println("整型指针的总大小是：", unsafe.Sizeof(ip))      // 8
	fmt.Println("string 指针的总大小是：", unsafe.Sizeof(sp)) // 8
	fmt.Println("------------------------")
}

func arrayFunc() {
	nums := [...]int{1: 2, 3: 4}
	fmt.Println("数组的长度：", len(nums)) // 4

	for _, v := range nums {
		println("v = ", v) // [0, 2, 0, 4]
	}

	fmt.Println("数组的总大小是：", unsafe.Sizeof(nums)) // 32
	fmt.Println("------------------------")
}

func stringFunc() {
	str := "这句话共七个字"
	fmt.Println("string 的长度：", len(str)) // 3 * 7 = 21

	for _, v := range str {
		fmt.Printf("v = %c \n", v)
	}

	fmt.Println("string 的总大小是：", unsafe.Sizeof(str)) // 16
	fmt.Println("------------------------")
}

func sliceFunc() {
	nums := []int{1, 3, 5, 7, 9, 11}
	fmt.Println("切片的长度：", len(nums)) // 6

	for _, v := range nums {
		println("v = ", v) // [1, 2, 5, 7, 9, 11]
	}

	fmt.Println("切片的总大小是：", unsafe.Sizeof(nums)) // 24
	fmt.Println("------------------------")
}

func mapFunc() {
	m := map[string]int{
		"yumi": 23,
		"amy":  19,
		"lucy": 22,
		"ben":  24,
		"may":  28,
	}

	fmt.Println("map 的长度：", len(m)) // 5

	for k, v := range m {
		println("k = ", k) // yumi、amy、lucy、ben、amy
		println("v = ", v) // 23、19、22、24、28
	}

	fmt.Println("map 的总大小是：", unsafe.Sizeof(m)) // 8
	fmt.Println("------------------------")
}

func main() {
	arrayFunc()

	sliceFunc()

	stringFunc()

	mapFunc()

	structFunc()

	interfaceFunc()

	pointerFunc()
}
```

执行之后，得到的输出如下：
```go
数组的长度： 4
v =  0
v =  2
v =  0
v =  4
数组的总大小是： 32
------------------------
切片的长度： 6
v =  1
v =  3
v =  5
v =  7
v =  9
v =  11
切片的总大小是： 24
------------------------
string 的长度： 21
v = 这 
v = 句 
v = 话 
v = 共 
v = 七 
v = 个 
v = 字 
string 的总大小是： 16
------------------------
map 的长度： 5
k =  may
v =  28
k =  yumi
v =  23
k =  amy
v =  19
k =  lucy
v =  22
k =  ben
v =  24
map 的总大小是： 8
------------------------
空结构体的总大小是： 0
初始化结构体的总大小是： 32
零值初始化的结构体的总大小是： 32
------------------------
空接口的总大小是： 16
接口的总大小是： 16
------------------------
空指针的总大小是： 8
整型指针的总大小是： 8
string 指针的总大小是： 8
------------------------
```

会不会有些意外？

切片、string、map 的总大小怎么有点奇怪？怎么是 24、16 和 8？

## sizeof

回答这个问题之前，先来看看 `unsafe.sizeof` 这个函数的定义：
```go
// $GOROOT/src/unsafe/unsafe.go

// Sizeof takes an expression x of any type and returns the size in bytes
// of a hypothetical variable v as if v was declared via var v = x.
// The size does not include any memory possibly referenced by x.
// For instance, if x is a slice, Sizeof returns the size of the slice
// descriptor, not the size of the memory referenced by the slice.
// The return value of Sizeof is a Go constant.
func Sizeof(x ArbitraryType) uintptr
```

大意就是：Sizeof 接受任何类型的表达式 x，并返回一个假设变量 v 的字节大小，就好像 v 是通过 var v=x 声明的，该大小不包括 x 可能引用的任何内存。\
例如，如果 x 是一个切片，Sizeof 返回**切片描述符的大小，而不是切片引用的内存大小**。

什么是描述符呢？\
以切片为例，就是它本身并不真正存储字符串数据，而仅是由一个**指向底层存储的指针**、**切片长度**和**切片最大容量**组成。

看到这里会不会清晰一些，有没有想起什么，没错，切片的数据结构刚好是由这三部分组成：
* array：指向底层数组的指针，类型为 uintptr，占用八个字节
* len：切片的长度，类型为 int，占用八个字节
* cap：切片的最大容量，类型为 int，占用八个字节

所以，在上面的示例代码中，切片变量的总大小就是 `8 + 8 + 8 = 24` 个字节。

string 类型也是一样的分析方式，它的描述符是由一个**指向底层存储的指针**和**字符串的长度**组成。

string 的数据结构：
* Data：指向底层存储的指针，类型为 uintptr，占用八个字节
* Len：字符串的长度，类型为 int，占用八个字节

string 变量的总大小就是 `8 + 8 = 16` 个字节。

结构体作为直接存储自身数据的类型，它和数组又有所不同，这是因为结构体是由若干个字段（field）聚合而成，每个字段都有自己的类型，所以结构体占用内存大小取决于组成结构体的各字段大小之和。

还是上面的示例代码，Person 结构体是由一个 string 类型、两个 int 类型组成，所以它占用的内存大小是 `16 + 8 + 8 = 32` 个字节。

注意，这里的string 就是上面的 string ，所以是 16 个字节。

因为空结构体中没有字段，所以总大小是零。

## 总结

对于整型、数组、结构体这类类型，它们的内存表示就是它们自身的数据内容，所以计算占用内存大小时，就是组成它们数据本身的大小。

而对于切片、string、map 等类型来说，它们的内存表示则是它们数据内容的“描述符”，所以计算占用内存大小时，需要以描述符的大小为准。

前者作为函数参数传递时，常被提到有性能开销，而后者则没有，也正是这个原因。