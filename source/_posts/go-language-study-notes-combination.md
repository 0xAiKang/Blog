---
title: Go 语言学习笔记——组合
date: 2022-11-04 22:51:46
tags: ["Go"]
categories: ["Go"]
---

前面两篇有关接口的笔记介绍了Go 接口的基本知识、接口类型定义的惯例以及接口在运行时的表示。

<!-- more -->

这篇笔记来学习一下，如何使用接口，不过，这里的“如何使用”，指的是学习如何利用接口进行应用设计，换句话说就是**Go 接口的应用模式或惯例**。

**在实际真正需要的时候才对程序进行抽象。不要为了抽象而抽象**。

## 一切皆组合
组合是 Go 语言的重要设计哲学之一，而正交性则为组合哲学的落地提供了更为方便的条件。

正交（Orthogonality）是从几何学中借用的术语，说的是如果两条线以直角相交，那么这两条线就是正交的。

编程语言的语法元素间和语言特性也存在着正交的情况，并且通过将这些正交的特性组合起来，可以实现更为高级的特性。

在语言设计层面，Go 语言就为广大 Gopher 提供了诸多正交的语法元素供后续组合使用，包括：
* Go 语言无类型体系，没有父子类的概念，类型定义是正交独立的
* 方法和类型是正交的，每种类型都可以拥有自己的方法集合，方法本质上只是一个将 receiver 参数作为第一个参数的函数而已
* 接口与它的实现者之间无“显式关联”，也就说接口与 Go 语言其他部分也是正交的

在这些正交语法元素中，接口作为 Go 语言提供的具有天然正交性的语法元素，在 Go 程序的静态结构搭建与耦合设计中扮演着至关重要的角色。而要想知道接口究竟扮演什么角色，我们就先要了解组合的方式。

构建 Go 应用程序的静态骨架结构有两种主要的组合方式，如下图所示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221109152526.png)

下面分别介绍垂直组合和水平组合。

## 垂直组合

垂直组合更多用在**将多个类型，通过类型嵌入的方式实现新类型的定义**。

传统面向对象变成语言大多是通过继承的方式构建出自己的类型体系，但 Go 语言并没有类型体系的概念。

Go 语言通过**类型的组合而不是继承让单一类型承载更多的功能**。

因为不是继承，也就没有了面向对象中的“父子关系”的概念了，也没有向上、向下转型（Type Casting），被嵌入的类型也不知道将其嵌入的外部类型的存在。**调用方法时，方法的匹配取决于方法名字，而不是类型**。

### 嵌入接口构建接口类型

在接口中嵌入接口，实现接口行为聚合，组成大接口。这种方式在标准库中非常常见，也是 Go 接口类型定义的惯例。

比如标准库中的 ReadWriter 接口类型的定义：
```go
// $GOROOT/src/io/io.go
type ReadWriter interface {
    Reader
    Writer
}
```

### 嵌入接口构建结构体类型

在结构体类型中嵌入接口：
```go
type MyReader struct {
  io.Reader // underlying reader
  N int64   // max bytes remaining
}
```

在结构体中嵌入接口，会包含嵌入的接口类型的方法集合，可以用于快速构建满足某一个接口的结构体类型。

### 嵌入结构体构建结构体类型

在结构体中嵌入接口类型名和在结构体中嵌入其他结构体，都是“委派模式（delegate）”的一种应用。
```go
type MyInt int
func (n *MyInt) Add(m int) {
    *n = *n + MyInt(m)
}
type S struct {
    *MyInt
}
func main() {
    m := MyInt(17)
    r := strings.NewReader("hello, go")
    s := S{
        MyInt: &m,
    }
    var sl = make([]byte, len("hello, go"))
    s.Add(5)
    fmt.Println(*(s.MyInt)) // 22
}
```
结构体实例 s 本身没有定义 Add 方法，于是会查看 S 的嵌入字段对应的类型是否定义了 Read 方法，找到之后，`s.Add` 的调用就被转换为 `s.MyInt.Add` 调用。

## 水平组合

水平组合就是通过接口将各个垂直组合出的类型“耦合”在一起。

通过接口进行水平组合的基本模式就是：**使用接受接口类型参数的函数或方法**。

在这个基本模式基础上，还有几种“衍生品”，下面一一介绍。

### 基本模式

接受接口类型参数的函数或方法是水平组合的基本语法，形式是这样的：
```go
func YourFuncName(param YourInterfaceType)
```

套用骨架关节的概念，用这幅图来表示上面基本模式语法的运用方法：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221109160229.png)

函数 / 方法参数中的接口类型作为“连接点”，支持将位于多个包中的多个类型与 YourFuncName 函数连接到一起，共同实现某一新特性。

### 创建模式

Go 社区流传一个经验法则：“接受接口，返回结构体（Accept interfaces, return structs）”，这其实就是一种把接口作为“连接点”的应用模式。

下面是 Go 标准库中，运用创建模式创建结构体实例的例子：
```go
// $GOROOT/src/sync/cond.go
type Cond struct {
    ... ...
    L Locker
}
func NewCond(l Locker) *Cond {
    return &Cond{L: l}
}
// $GOROOT/src/log/log.go
type Logger struct {
    mu     sync.Mutex 
    prefix string     
    flag   int        
    out    io.Writer  
    buf    []byte    
}
func New(out io.Writer, prefix string, flag int) *Logger {
    return &Logger{out: out, prefix: prefix, flag: flag}
}
```

以上面 log 包的 New 函数为例，这个函数用于实例化一个 log.Logger 实例，它接受一个 io.Writer 接口类型的参数，返回 *log.Logger。从 New 的实现上来看，传入的 out 参数被作为初值赋值给了 log.Logger 结构体字段 out。

创建模式通过接口，在 NewXXX 函数所在包与接口的实现者所在包之间建立了一个连接。

大多数包含接口类型字段的结构体的实例化，都可以使用创建模式实现。

### 包装器模式

### 适配器模式

### 中间件

## 尽量避免使用空接口作为函数参数类型

```go
// $GOROOT/src/io/io.go
type Reader interface {
  Read(p []byte) (n int, err error)
}
```

Go 编译器通过解析这个接口定义，得到接口的名字信息以及它的方法信息，在为这个接口类型参数赋值时，编译器就会根据这些信息对实参进行检查。

可是，如果函数或方法的参数类型为空接口`interface{}`，编译器无法得知实参的任何信息，因此只有到运行时才能发现错误。

## 总结
* Go 语言通过类型嵌入（Type Embedding）实现垂直组合
* 水平组合就是通过接口将各个垂直组合出的类型“耦合”在一起，有多种模式
* 尽量不要使用可以“逃过”编译器类型安全检查的空接口类型 `interface{}`

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)