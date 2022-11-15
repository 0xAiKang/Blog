---
title: Go 语言学习笔记——接口
date: 2022-11-02 23:26:20
tags: ["Go"]
categories: ["Go"]
---

在前面的笔记中，已经很多次用到了接口，但是还没有真正介绍它，是因为它和并发原语（Goroutine、channel、select）一样重要，更考验理解力，所以放在后面一些。

下面就正式进入接口的学习了。

<!-- more -->

## 接口类型
**接口类型是由 type 和 interface 关键字定义的一组方法集合**，其中，**方法集合**唯一确定了这个接口类型所表示的接口。

下面是一个典型的接口类型定义：
```go
type MyInterface interface {
  M1(int) error
  M2(io.Writer, ...string)
}
```

通过这个定义，可以看到，接口类型 MyInterface 所表示的接口的方法集合，包含两个方法 M1 和 M2。**之所以称 M1 和 M2 为“方法”，更多是从这个接口的实现者的角度考虑的**。

接口类型的方法集合中声明的方法，它的参数列表不需要写出形参名字，返回值列表也是如此。

Go 语言要求接口类型声明中的方法必须是具名的，并且方法名字在这个接口类型的方法集合中是唯一的。

Go 接口类型允许嵌入的不同接口类型的方法集合存在交集，但前提是交集中的方法不仅名字要一样，它的函数签名部分也要保持一致，也就是参数列表与返回值列表也要相同，否则 Go 编译器照样会报错。

```go
type Interface1 interface {
    M1()
}
type Interface2 interface {
    M1(string) 
    M2()
}
type Interface3 interface{
    Interface1
    Interface2 // 编译器报错：duplicate method M1
    M3()
}
```

接口类型定义中也可以声明首字母小写的非导出方法，不过，在日常的编码过程中，较少使用这种非导出方法的接口类型。

### 空接口类型

如果一个接口类型定义中没有一个方法，那么它的方法集合就为空：
```go
type EmptyInterface interface {

}
```

这个方法集合为空的接口类型就被称为**空接口类型**。

但是通常不需要自己显示定义这类空接口类型，可以直接使用 `interface{}` 这个类型字面值作为所有空接口类型的代表就可以了。

接口类型一旦被定义后，它就和其他 Go 类型一样可以用于声明变量，比如：

```go
var err error   // err是一个error接口类型的实例变量
var r io.Reader // r是一个io.Reader接口类型的实例变量
```

这些类型为接口类型的变量被称为**接口类型变量**，如果没有被显式赋予初值，接口类型变量的默认值为 nil。\
如果要为接口类型变量显式赋予初值，我们就要为接口类型变量选择合法的右值。

如果一个变量的类型是空接口类型，由于空接口类型的方法集合为空，这就意味着任何类型都实现了空接口的方法集合，所以可以将任何类型的值作为右值，赋值给空接口类型的变量

```go
var i interface{} = 15   // ok
fmt.Printf("%T \n", i)   // int

i = "hello, golang"      // ok
fmt.Printf("%T \n", i)   // string

type T struct{}
var t T
i = t  // ok
i = &t // ok
```

空接口类型的这一可接受任意类型变量值作为右值的特性，让他成为 Go 加入泛型语法之前唯一一种具有“泛型”能力的语法元素，包括 Go 标准库在内的一些通用数据结构与算法的实现，都使用了空类型interface{}作为数据元素的类型，这样我们就无需为每种支持的元素类型单独做一份代码拷贝了。

Go 语言还支持接口类型变量赋值的“逆操作”，也就是通过接口类型变量“还原”它的右值的类型与值信息，这个过程被称为“**类型断言（Type Assertion）**”。类型断言通常使用下面的语法形式：
```go
v, ok := i.(T)
```

如果接口类型变量 i 之前被赋予的值确为 T 类型的值，那么这个语句执行后，左侧“comma, ok”语句中的变量 ok 的值将为 true，变量 v 的类型为 T，它值会是之前变量 i 的右值。\
如果 i 之前被赋予的值不是 T 类型的值，那么这个语句执行后，变量 ok 的值为 false，变量 v 的类型还是那个要还原的类型，但它的值是类型 T 的零值。

类型断言也支持下面这种语法形式：

```go
v := i.(T)
```
但是在这种语法形式下，如果接口变量i 之前被赋予的值不是 T 类型的值，那么这个语句将抛出 panic。\
因为可能会出现 panic，所以并不推荐使用这种语法形式。

下面用一段示例代码来加深一下理解：
```go
var a int64 = 13
var i interface{} = a
v1, ok := i.(int64) 
// 断言成功
fmt.Printf("v1=%d, the type of v1 is %T, ok=%t\n", v1, v1, ok)
v2, ok := i.(string)
// 断言失败，变量 i 的 int64 与 string 类型不一致
fmt.Printf("v2=%s, the type of v2 is %T, ok=%t\n", v2, v2, ok)
v3 := i.(int64) 
// 断言成功
fmt.Printf("v3=%d, the type of v3 is %T, ok =%t\n", v3, v3, ok) 
v4 := i.([]int)
fmt.Printf("the type of v4 is %T\n", v4) 
```

运行示例代码，输出如下：
```go
v1=13, the type of v1 is int64, ok=true
v2=, the type of v2 is string, ok=false
v3=13, the type of v3 is int64, ok =false
panic: interface conversion: interface {} is int64, not []int
```

在这段代码中，如果 `v, ok := i.(T)` 中的 T 是一个接口类型，那么类型断言的语义就会变成：断言 i 的值实现了接口类型 T。如果断言成功，变量 v 的类型为 i 的值的类型，而并非接口类型 T。如果断言失败，v 的类型信息为接口类型 T，它的值为 nil，下面再来看一个 T 为接口类型的示例：

```go
type MyInterface interface {
    M1()
}
type T int
               
func (T) M1() {
    println("T's M1")
}              
               
func main() {  
    var t T    
    var i interface{} = t
    v1, ok := i.(MyInterface)
    if !ok {   
        panic("the value of i is not MyInterface")
    }          
    v1.M1()    
    fmt.Printf("the type of v1 is %T\n", v1) // the type of v1 is main.T
               
    i = int64(13)
    v2, ok := i.(MyInterface)
    fmt.Printf("the type of v2 is %T\n", v2) // the type of v2 is <nil>
    // v2 = 13 //  cannot use 1 (type int) as type MyInterface in assignment: int does not implement MyInterface (missing M1   method) 
}
```

可以看到，通过`the type of v2 is <nil>`，其实是看不出断言失败后的变量 v2 的类型的，但通过最后一行代码的编译器错误提示，我们能清晰地看到 v2 的类型信息为 MyInterface。

## 尽量定义小接口

而 Go 选择了去繁就简的形式，这主要体现在以下两点上：
### 隐式契约
Go 语言中接口类型与它的实现者之间的关系是隐式的，不需要像其他语言（比如 Java）那样要求实现者显式放置“implements”进行修饰，实现者只需要实现接口方法集合中的全部方法便算是遵守了契约，并立即生效了。

### 更倾向于“小契约”

如果契约太繁杂了就会束缚了手脚，缺少了灵活性，抑制了表现力。所以 Go 选择了使用“小契约”，表现在代码上就是**尽量定义小接口，即方法个数在 1~3 个之间的接口**。Go 语言之父 Rob Pike 曾说过的“接口越大，抽象程度越弱”，这也是 Go 社区倾向定义小接口的另外一种表述。

## 小接口有哪些优势

### 接口越小，抽象程度越高

### 小接口易于实现和测试

这是一个显而易见的优点。小接口拥有比较少的方法，一般情况下只有一个方法。所以要想满足这一接口，只需要实现一个方法或者少数几个方法就可以了，这显然要比实现拥有较多方法的接口要容易得多。

尤其是在单元测试环节，构建类型去实现只有少量方法的接口要比实现拥有较多方法的接口付出的劳动要少许多。

### 小接口表示的“契约”职责单一，易于复用组合

Go 推崇通过组合的方式构建程序。Go 开发人员一般会尝试通过嵌入其他已有接口类型的方式来构建新接口类型，就像通过嵌入 io.Reader 和 io.Writer 构建 io.ReadWriter 那样。

那构建时，如果有众多候选接口类型供选择，该怎么选择呢？\
选择那些新接口类型需要的契约职责，同时也要求不要引入我们不需要的契约职责。在这样的情况下，拥有单一或少数方法的小接口便更有可能成为我们的目标，而那些拥有较多方法的大接口，可能会因引入了诸多不需要的契约职责而被放弃。

由此可见，小接口更契合 Go 的组合思想，也更容易发挥出组合的威力。

## 总结
* 接口类型定义中嵌入的不同接口类型的方法集合若存在交集，交集中的方法不仅名字要一样，函数签名也要相同，否则会编译失败
* 对接口类型和非接口类型进行类型断言的语义是不完全相同的
* Go 惯例上推荐尽量定义小接口，即方法个数在 1～3 个之间
* 小接口有诸多优点，比如，抽象程度高、易于测试与实现、与组合的设计思想一脉相承
* 接口本质是契约，具有天然的降低耦合的作用

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)