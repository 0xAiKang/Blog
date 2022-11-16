---
title: Go 语言学习笔记——类型嵌入
date: 2022-11-01 23:29:51
tags: ["Go"]
categories: ["Go"]
---

前两篇笔记主要了解了 Go 方法的声明、本质，以及 receiver 类型选择的三个原则，这篇笔记主要来了解 Go 语言的组合思想——类型嵌入。

<!-- more -->

## 类型嵌入
什么是类型嵌入？\
类型嵌入指的是在一个类型的定义中嵌入了其他类型。Go 语言支持两种类型嵌入，分别是接口类型的类型嵌入和结构体类型的类型嵌入。

### 接口类型嵌入

接口类型声明了由一个方法集合代表的接口，比如下面接口类型 E：
```go
type E interface {
  M1()
  M2()
}
```

这个接口类型 E 的方法集合，包含两个方法，分别是 M1 和 M2，它们组成了 E 这个接口类型所代表的接口。\
如果某个类型实现了方法 M1 和 M2，就可以说这个类型实现了 E 所代表的接口。

此时，再定义另外一个接口类型 I，它的方法集合中包含了三个方法M1、M2、和 M3：
```go
type I interface {
  M1()
  M2()
  M3()
}
```

接口类型 I 的方法集合中定义的 M1、M2 和接口类型 E 的方法集合中的方法完全相同。\
这种情况下，可以直接使用接口类型 E 替代上面接口类型 I 定义的 M1 和 M2：
```go
type I interface {
  E
  M3()
}
```

像这种在一个接口类型（I）定义中，嵌入另外一个接口类型（E）的方式，就是**接口类型的类型嵌入**。

而且，这个带有类型嵌入的接口类型 I的定义与上面那个包含 M1、M2、M3 的接口类型 I 的定义，是等价的。\
因此可以得出一个结论：**接口类型嵌入的语义就是新接口类型（I）将嵌入接口类型（E）的方法集合，并入到自己的方法集合中**。

到这里你可能会问，既然都是等价的，那么直接在接口类型定义中平铺方法列表就好了，为啥要使用类型嵌入方式定义接口类型呢？其实这也是 **Go 组合设计哲学的一种体现**。

按 Go 语言惯例，Go 中的接口类型中只包含少量方法，并且常常只是一个方法。通过在接口类型中嵌入其他接口类型可以实现接口的组合，这也是 **Go 语言中基于已有接口类型构建新接口类型的惯用法**。

### 结构体类型嵌入 

其实在前面的结构体笔记中，有简单用到过结构体类型嵌入，但是没有深入了解，那么接下来了解一下。

```go
type T1 int
type t2 struct{
    n int
    m int
}
type I interface {
    M1()
}
type S1 struct {
    T1
    *t2
    I            
    a int
    b string
}

// 需要注意 🚧，如果定义成这样，就不是嵌入字段了，因此不会继承接口的方法集合，也就不能直接调用 S1.M1() 了
/*type S1 struct {
    T1 T1
    t2 *t2
    I I            
    a int
    b string
}*/
```
上面的示例代码是一个带有**嵌入字段（Embedded Field）的结构体定义**。

可以看到，结构体 S1 定义中有三个“非常规形式” 的标识符，分别是 T1、`*t2`、和 I，像这种“非常规形式” 的标识符既代表字段的名字，也代表字段的类型：
* T1：字段名为 T1，类型为自定义类型 T1
* `*t2`：字段名为 t2，类型为自定义结构体类型 t2 的指针类型
* I：字段名为 I，类型为接口类型 I

这种以某个类型名、类型的指针类型名或接口类型名，直接作为结构体字段的方式就叫做**结构体的类型嵌入**，这些字段也被叫做**嵌入字段（Embedded Field）**。

## “继承”原理

嵌入字段具体有什么用呢？它跟普通结构体字段又有什么不同？下面结合一段示例代码来具体说明：
```go
type MyInt int
func (n *MyInt) Add(m int) {
    *n = *n + MyInt(m)
}
type t struct {
    a int
    b int
}
type S struct {
    *MyInt
    t
    io.Reader
    s string
    n int
}
func main() {
    m := MyInt(17)
    r := strings.NewReader("hello, go")
    s := S{
        MyInt: &m,
        t: t{
            a: 1,
            b: 2,
        },
        Reader: r,
        s:      "demo",
    }
    var sl = make([]byte, len("hello, go"))
    s.Reader.Read(sl)
    fmt.Println(string(sl)) // hello, go
    s.MyInt.Add(5)
    fmt.Println(*(s.MyInt)) // 22
}
```

在这个示例中，结构体类型 S 使用了类型嵌入方式进行定义，嵌入了三个字段Myint、t、以及 Reader。

第三个嵌入字段的名字为 Reader 而不是 io.Reader 的原因是，Go 语言规定如果结构体使用从其他包导入的类型作为嵌入字段，比如 pkg.T，那么这个嵌入字段的字段名就是 T，代表的类型为 pkg.T。

运行上面的示例代码，输出如下：
```go
hello, go
22
```

这样看起来，使用嵌入字段和普通字段似乎并没有什么差别，输出都是一样的。

将 main 函数中，部分代码替换成下面这部分：
```go
var sl = make([]byte, len("hello, go"))
s.Read(sl) 
fmt.Println(string(sl))
s.Add(5) 
fmt.Println(*(s.MyInt))
```

这里可能会有疑问，类型 S 又没有定义 Read 方法和 Add 方法，这样写不会编译失败吗？

再次运行示例代码，会发现不但没有编译失败，程序还正常输出了。

之所以没有编译失败，是因为这两个方法就是来自于结构体类型 S 的两个嵌入字段 Reader 和 MyInt。\
结构体类型 S“继承”了 Reader 字段的方法 Read 的实现，也“继承”了 `*MyInt` 的 Add 方法的实现。

这里的”继承“打了引号，并不是真正意义上的继承，只是使用了这一语义。

其原理是通过结构体类型 S 的实例 s 调用 Read 方法时，Go 发现结构体类型 S 自身并没有定义 Read 方法，于是 Go 会查看 S 的嵌入字段对应的类型是否定义了 Read 方法。这个时候，Reader 字段就被找了出来，之后 `s.Read` 的调用就被转换为 `s.Reader.Read` 调用。

这种将调用“委派”给该结构体内部嵌入类型的实例去执行，叫做委派模式。

当外界调用新类型的方法时，Go 编译器会首先查找新类型是否实现了这个方法，如果没有，就会将调用委派给其内部实现了这个方法的嵌入类型的实例去执行

Add 方法的调用原理同上。

因此，现在就清楚了嵌入字段的作用，它可以用来**实现方法的“继承”**。

## 类型嵌入与方法集合
在前面讲解接口类型的类型嵌入时，我们提到过接口类型的类型嵌入的本质，就是**嵌入类型的方法集合并入到新接口类型的方法集合中**，并且，**接口类型只能嵌入接口类型**。而**结构体类型对嵌入类型的要求就比较宽泛了**，**可以是任意自定义类型或接口类型**。

下面就分别来看看，在这两种情况下，结构体类型的方法集合会有怎样的变化。\
这里借助前面笔记中的 dumpMethodSet 工具函数来输出各个类型的方法集合。

### 结构体类型中嵌入接口类型

```go
type I interface {
    M1()
    M2()
}
type T struct {
    I
}
func (T) M3() {}
func main() {
    var t T
    var p *T
    dumpMethodSet(t)
    dumpMethodSet(p)
}
```

运行上面的示例代码，输出如下：
```
main.T's method set:
- M1
- M2
- M3
*main.T's method set:
- M1
- M2
- M3
```

可以看到，原本结构体类型 T 只带有一个方法 M3，但在嵌入接口类型 I 后，结构体类型 T 的方法集合中又并入了接口类型 I 的方法集合。

所以，结论就是：**结构体类型的方法集合，包含嵌入的接口类型的方法集合**。

不过这里需要注意：和前面接口类型中嵌入接口类型，不同的是，结构体类型嵌入接口类型不允许方法集合存在交集。

```go
type E1 interface {
    M1()
    M2()
    M3()
}
type E2 interface {
   M1()
   M2()
   M4()
}
type T struct {
   E1
   E2
}
func main() {
   t := T{}
   t.M1()
   t.M2()
}
```

运行上面的示例代码，会发现编译失败：
```go
main.go:22:3: ambiguous selector t.M1
main.go:23:3: ambiguous selector t.M2
```

这是因为两个接口类型中都存在 M1 与 M2 方法，在结构体没有实现这两个方法的情况下，编译器无法自己做出选择。

解决方案也很简单：
1. 消除接口类型中重复定义的方法
2. 为结构体增加 M1、M2 方法的实现

### 结构体类型中嵌入结构体类型

前面已经了解了，在结构体类型中嵌入结构体类型，
可以作为实现”继承“的手段。

外部的结构体类型 T 可以“继承”嵌入的结构体类型的所有方法的实现。并且，无论是 `T` 类型的变量实例还是 `*T` 类型变量实例，都可以调用所有“继承”的方法。

但这种情况下，**带有嵌入类型的新类型究竟“继承”了哪些方法**，通过下面的示例来看一下：
```go
type T1 struct{}

func (T1) T1M1()   { println("T1's M1") }
func (*T1) PT1M2() { println("PT1's M2") }

type T2 struct{}

func (T2) T2M1()   { println("T2's M1") }
func (*T2) PT2M2() { println("PT2's M2") }

type T struct {
    T1
    *T2
}
func main() {
    t := T{
        T1: T1{},
        T2: &T2{},
    }
    dumpMethodSet(t)
    dumpMethodSet(&t)
}
```

上面的示例代码中，各实例的方法集合是不同的：
* `T1` 的方法集合包含：T1M1
* `*T1` 的方法集合包含：T1M1、PT1M2
* `T2` 的方法集合包含：T2M1
* `*T2` 的方法集合包含：T2M1、PT2M2

运行示例代码，输出如下：
```go
main.T's method set:
- PT2M2
- T1M1
- T2M1
*main.T's method set:
- PT1M2
- PT2M2
- T1M1
- T2M1
```

通过输出结果，我们看到了 `T` 和 `*T` 类型的方法集合果然有差别的：
* 类型 `T` 的方法集合 = `T1` 的方法集合 + `*T2` 的方法集合
* 类型 `*T` 的方法集合 = `*T1` 的方法集合 + `*T2` 的方法集合

这里需要注意的是，`*T` 类型的方法集合，它包含的可不是 `T1` 类型的方法集合，而是 `*T1` 类型的方法集合，而 `*T1` 方法集合又包含 T1M1、PT1M2，（T2同理），所以`*T` 类型的方法集合包含了PT1M2、PT2M2、T1M1、T2M2。

## 总结
* 接口类型嵌入就是在一个接口类型中，嵌入另外一个接口类型，允许方法集合并入
* 结构体类型嵌入就是以某个类型名、类型的指针类型名或接口类型名，直接作为结构体字段
* 接口类型只能嵌入接口类型，结构体类型可以嵌入任意自定义类型或接口类型
* 在 Go 语言中可以借助结构体类型嵌入实现“继承”
* 结构体类型中嵌入接口类型时，包含嵌入的接口类型的方法集合（但是接口类型的方法集合不能存在交集）
* 结构体类型中嵌入结构体类型时，`T` 和 `*T` 的方法集合不一样
* 无论原类型是接口类型还是非接口类型，类型别名都与原类型拥有完全相同的方法集合

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)