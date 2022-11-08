---
title: Go 语言学习笔记——结构体
date: 2022-10-27 23:59:51
tags: ["Go"]
categories: ["Go"]
---

Go 语言并不是一门面向对象的编程语言，它没有面向对象所特有的 class，在 Go 语言中，对事物进行抽象使用结构体类型 struct。

<!-- more -->

不过，在学习如何定义一个结构体类型之前，首先要来看看如何在 Go 中自定义一个新类型。

## 自定义新类型
在 Go 中，自定义一个新类型一般有两种方法，下面一一介绍。

### type

使用类型声明语法（Type 关键字），这也是最常用的方式：
```go
type T S // 基于类型 S 定义一个新类型T
```
Go 语言中，凡通过类型声明语法声明的类型都被称为 defined 类型。

在这里，S 可以是任何一个已定义的类型，包括 **Go 原生类型**，或者是**其他已定义的自定义类型**。

```go
type T1 int
type T2 T1
```

在上面的示例代码中，新类型 T1 是基于 Go 原生类型 int 定义的新自定义类型，而新类型 T2 则是基于刚刚定义的类型 T1，定义的新类型。

这里引入一个概念：**底层类型（Underlying Type）**—— 如果一个新类型是基于某个 Go 原生类型或者其他已定义的自定义类型定义的，那么就可以说 Go 原生类型/其他已定义的自定义类型是新类型的底层类型。

底层类型在 Go 语言中有重要作用，**它被用来判断两个类型本质上是否相同（Identical）。**

在上面例子中，虽然 T1 和 T2 是不同类型，但因为它们的底层类型都是类型 int，所以它们在本质上是相同的。

而本质上相同的两个类型，它们的变量可以通过显式转型进行相互赋值，相反，如果本质上是不同的两个类型，它们的变量间连显式转型都不可能，更不要说相互赋值了。

```go
type T1 int
type T2 T1
type T3 string
func main() {
    var n1 T1
    var n2 T2 = 5
    n1 = T1(n2)  // ok
    
    var s T3 = "hello"
    n1 = T1(s) // 错误：cannot convert s (type T3) to type T1
}
```

除了基于已有类型定义新类型之外，还可以基于**类型字面值**来定义新类型，这种方式多用于自定义一个新的复合类型：
```go
type M map[int]string     // 定义一个 [int]string 类型的 map
type S []string           // 定义一个 切片类型的新类型

// 也可以写成这样
type (
	M map[int]string
	S []int
)
```

### type alias

第二种方式是使用类型别名（Type Alias）：
```go
type T = S // type alias
```

与前面的第一种自定义新类型的方式相比，类型别名在形式上多出了一个等号，其次就是新类型 T 和原类型 S 是完全等价的，完全等价的意思就是，类型别名并没有定义出新类型，类 T 与 S 实际上就是**同一种类型**。

通过下面这段示例代码来验证：
```go
type T = string 
  
var s string = "hello" 
var t T = s // ok
fmt.Printf("%T\n", t) // string
```

类型 T 是通过类型别名的方式定义的，T 与 string 实际上是一个类型，所以这里，使用 string 类型变量 s 给 T 类型变量 t 赋值的动作，实质上就是**同类型赋值**。最后输出的 string 也是符合预期的。

## 结构体

复合类型的定义一般都是通过**类型字面值**的方式来进行的，作为复合类型之一的结构体类型也不例外：
```go
// 定义一个名称为 T 的结构体类型
type T struct {
    Field1 T1
    Field2 T2
    // ... ...
    FieldN Tn
}
```
类型字面值由若干个字段（field）聚合而成，每个字段有自己的名称与类型，且每个字段的名称是唯一的。

另外，这个名称为 T 的结构体，因为首字母是大写的关系，它是带有导出标识符的，所以在其他包中也可以被访问到，反之，如果是小写，则只能在当前包中使用。结构体中的字段也遵循这个规则。

除了上面这种典型的定义方式，还有几种特殊的情况。

### 空结构体

可以定义一个空结构体，也就是没有包含任何字段的结构体类型：
```go
type Empty struct{} // Empty是一个不包含任何字段的空结构体类型

var e Empty
// 查看变量内存占用大小
println(unsafe.Sizeof(e)) // 0
```

因为空结构体类型变量的内存占用为 0，基于空结构体类型内存零开销这样的特性，可以作为“事件”信息进行 Goroutine 之间的通信：
```go
var c = make(chan Empty) // 声明一个元素类型为Empty的channel
c<-Empty{}               // 向channel写入一个“事件”
```

这种以空结构体为元素类建立的 channel，是目前能实现的、内存占用最小的 Goroutine 间通信方式。

### 类型嵌入

类型嵌入指的就是在一个类型的定义中嵌入了其他类型，Go 语言支持两种类型嵌入，**接口类型的类型嵌入**和**结构体类型的类型嵌入**。

这里先只介绍结构体类型的类型嵌入，后面的笔记中会详细介绍它俩。
```go
type Person struct {
    Name string
    Phone string
    Addr string
}

type Book1 struct {
    Title string
    Author Person
}

// 还可以使用嵌入字段 省略字段名称
type Book2 struct {
    Title string
    Person
}
```

访问 Book 结构体字段 Author 中的 Phone 字段，下面两种方式是等价的：
```go
var book1 Book1
var book2 Book2

// 正常通过结构体字段一层一层访问
println(book1.Author.Phone)

// 直接访问嵌入字段所属类型中字段
println(book2.Phone)
```

## 结构体的声明与初始化

和其他所有变量的声明一样，也可以使用标准变量声明语句，或者是短变量声明语句声明一个结构体类型的变量：
```go
type Book struct {
     // ...
}

// 这三种方式都是等价的
var book Book
var book = Book{}
book := Book{}       // 推荐使用复合字面值的形式
```

### 零值初始化
零值初始化说的是使用结构体的零值作为它的初始值。

结构体类型的零值变量，通常不具有或者很难具有合理的意义，比如通过下面代码得到的零值 book 变量就是这样：
```go
var book Book    // book为零值结构体变量
```

因为一本书既没有书名，也没有作者、页数、索引等信息，那么通过 Book 类型对这本书的抽象就失去了实际价值。所以对于像 Book 这样的结构体类型，使用零值初始化并不是正确的选择。

但是这并不是意味着零值初始化就完全没有意义了，相反，如果一种类型采用零值初始化得到的零值变量，是有意义的，而且是直接可用的。

可以说，定义零值可用类型是简化代码、改善开发者使用体验的一种重要的手段。

Go 标准库中的 `bytes.Buffer` 结构体类型，就是一个典型的例子：
```go
var b bytes.Buffer            
b.Write([]byte("Hello, Go"))
fmt.Println(b.String())        // 输出：Hello, Go
```

可以看到不需要对 `bytes.Buffer` 类型的变量 b 进行任何显式初始化，就可以直接通过处于零值状态的变量 b，调用它的方法进行写入和读取操作。

### 复合字面值

最简单的对结构体变量进行显式初始化的方式，就是按顺序依次给每个结构体字段进行赋值：
```go
type Book struct {
    Title string              // 书名
    Pages int                 // 书的页数
    Indexes map[string]int    // 书的索引
}

var book = Book{"The Go Programming Language", 700, make(map[string]int)}
println("Book Name：", book.Title)       // Book Name：The Go Programming Language
println("Book Pages：", book.Pages)      // Book Pages：700
```

这种方式虽然是最简单的，但是却不是最优的，因为存在很多问题：
* 当结构体类型定义中的字段顺序发生变化，或者字段出现增删操作时，就需要手动调整该结构体类型变量的显式初始化代码
* 当一个结构体的字段较多时，这种逐一字段赋值的方式实施起来就会比较困难，增加开发者的心智负担
* 一旦结构体中包含非导出字段，那么这种逐一字段赋值的方式就不再被支持了，编译器会报错

Go 语言推荐我们用 **field:value** 形式的复合字面值，对结构体类型变量进行显式初始化：
```go
var book = Book {
  Title: "The Go Programming Language",
  Pages: 700,
  Indexes: make(map[string]int),
}
```

使用这种方式，不用担心结构体字段的顺序。未显式出现在字面值中的结构体字段将采用它对应类型的零值。

## 总结
* Go 语言不是一门面向对象范式的编程语言，它没有 C++ 或 Java 中的那种 class 类型
* Go 语言通过结构体，提供抽象能力
* 结构体的定义不支持递归
* 结构体的初始化有几种方式：零值初始化、复合字面值初始化，依据实际场景选择使用

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)