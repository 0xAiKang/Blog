---
title: Go 语言零散学习笔记
date: 2022-11-10  22:10:22
tags: ["Go"]
categories: ["Go"]
---

Go 语言学习零散笔记整理。

<!-- more -->

## 零值初始化

同样是声明一个切片，这两种方式有什么区别？
```go
// sl1 这个变量声明了，还没初始化，是 nil 值，和nil比较返回true，底层没有分配内存空间
var sl1 []int            

// sl2 这个变量声明且初始化了，不是nil值，和nil比较返回false，底层分配了内存空间，有地址
var sl2 = []int{}
```

同样是声明一个 map，这两种方式有什么区别？
```go
// 变量 m1 只是声明了但是没有初始化，默认值为 nil，这个时候如果直接对 map 进行赋值，则会导致运行时异常
var m1 map[int][string]

// 变量 m2 声明且初始化了
m2 := map[int][string]{}

// 另一种初始化方式
m1 = make(map[int][string])
```
因此在使用 map 之前，必须先对其进行初始化。

## 引用类型
Go 语言的基本数据类型中，map、slice、channel 这些都是引用类型。

引用类型的特点是，赋值时，不是值传递而是引用传递。

下面通过一个示例来理解：
```go
func main() {
	m1 := make(map[int]int)
	m2 := make(map[int]int)
	m2 = m1

	m1[1] = 99
	for _, v := range m2 {
		fmt.Println(v)
	}

	var i1 int
	var i2 int

	i2 = i1
	i1 = 99
	fmt.Println(i2)
}
```

运行上面的示例代码，输出如下：
```go
99
0
```

符合预期。

## 错误处理
首先明确一个前提：错误不是异常。

常见的错误处理方法是返回 `error`，由调用者决定后续如何处理。但是如果是无法恢复的错误，通常会触发 `panic`，程序会因此而无法运行，而这个就是异常。
  
```go
func main() {
	err := M()
	if err != nil {
		// do somethings
		fmt.Println(err.Error())
	}
}

func M() error {
	// ... ...
	return errors.New("some error occurred")
}
```
`error` 是一个接口，它是 Go 原生内置的类型。

`errors` 是一个包文件，通常用到它的 `errors.New` 方法用构造一个错误值，赋值给接口。因此通常把 `error` 作为返回值类型，`errors.New()` 作为构造 error 类型的返回值的方式之一，`fmt.Errorf()` 也可以构造。

## 定义一个新类型
`type T1 ＝T` 和 `type T1 T` 两个语法本质上有什么区别呢？

前者是基于类型别名（Type Alias）定义新类型，后者是通过类型声明（Type define）给原类型起别名。

```go
// type alias
type T = string  // 类型 T1 是基础类型 string 的别名

func main() {
   var s string = "hello" 
   var t T = s // ok
   fmt.Printf("%T\n", t) // string
}
```
类型 T 与 string 完全等价。完全等价的意思就是，类型别名并没有定义出新类型，类 T 与 string 实际上就是同一种类型，因此通常会称 **类型 T 是基础类型 string 的别名**。

```go
// type define
type T1 int      // 类型 T1 是基础类型 int 的新类型
type T2 T1       // 类型 T1 是自定义类型 T1 的新类型
type T3 string   // 类型 T3 是基础类型 string 的新类型

func main() {
    var n1 T1
    var n2 T2 = 5
    n1 = T1(n2)  // ok
    
    var s T3 = "hello"
    n1 = T1(s) // 错误：cannot convert s (type T3) to type T1
}
```
虽然 T1 和 T2 是不同类型，但因为它们的**底层类型**都是类型 int，所以它们在本质上是相同的。而本质上相同的两个类型，它们的变量可以通过显式转型进行相互赋值，相反，如果本质上是不同的两个类型，它们的变量间连显式转型都不可能，更不要说相互赋值了。

底层类型这个概念在 Go 语言中有重要作用，通常被用来判断两个类型本质上是否相同（Identical）。

## 接口类型不能使用指针

下面这样的代码很常见，通常使用第三方依赖时，经常会写出这样的代码。可是，有没有想过一个问题？为什么 `http.ResponseWriter` 是没有带指针的，而 `*http.Request` 又带了指针？
```go
type Context struct {
	Writer http.ResponseWriter
	Req *http.Request 
}

func indexHanlder(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(w, "URL.Path = %q \n", req.URL.Path)
}
```

记住一句话：**使用指针可以节省内存，但接口类型不能使用指针**。因此定义结构体、作为函数参数时接口类型都是没有指针的，而结构体类型则需要结合实际情况考虑加不加指针。

## goroutine 共享变量

下面这段代码，作为 goroutine 运行的闭包会发生什么？
```go
func main() {
	values := []string{"a", "b", "c"}
	for _, v := range values {
		go func() {
			fmt.Println(v)
		}()
	}

	time.Sleep(1 * time.Second)
}
```

初次使用 goroutine 时，肯定都遇到过这个问题，怎么全部输出的是 c、c、c？

这是因为变量 v 是一个共享变量，存在竞争状态。有两种解决方案：将变量作为参数传递给闭包和使用中间变量。

将变量作为参数传递：
```go
for _, v := range values {
  go func(u string) {
      fmt.Println(u)
  }(v)
}
```

使用中间变量：
```go
for _, v := range values {
  v := v
  go func() {
      fmt.Println(v)
  }()
}
```

## 如何调试 Go
如果使用 [Goland](https://www.jetbrains.com/go/) 作为开发工具，那么 Go 程序的调试非常简单，不需要额外安装插件，开箱即用。

只需要在对应文件位置，打上断点即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221117225101.png)

如果使用 VsCode 作为开发工具，则需要通过额外安装插件并配置达到调试目的，所以建议直接使用 IDE。

---

不过需要注意的是，操作系统需要与对应版本对应上，否则是调试不了的。

例如，M1 芯片 不支持amd64，如果刚好安装的是 `darwin/amd64` 版本，则会提示：

> Debugging programs compiled with go version go1.18.1 darwin/amd64 is not supported.
> 

解决方案很简单，直接[下载](https://go.dev/dl/)安装 arm64 版本即可。

## 如何卸载 Go

卸载已经按照好的 Go 非常简单，通常只需要三步：

1. 找到Go 二进制文件的位置，通常是 `/usr/local/go`
```bash
$ which go
/usr/local/go/bin/go
```

2. 删除Golang二进制文件
```bash
sudo rm -rvf /usr/local/go/
```

3. 从 PATH 环境变量中删除 Go 二进制文件所在目录

如果是 macOS，则需要多一步删除 `/etc/paths.d/go` 文件：
```bash
$ sudo rm -rvf /etc/paths.d/go
```

详情可以查看[官方文档](https://go.dev/doc/manage-install)