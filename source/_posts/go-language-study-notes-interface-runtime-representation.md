---
title: Go 语言学习笔记——接口运行时表示
date: 2022-11-03 22:55:28
tags: ["Go"]
categories: ["Go"]
---

接口在 Go 中的地位非常高，这是因为接口是 Go 这门静态语言中唯一“动静兼容”的语法特性。

<!-- more -->

## 静态特性与动态特性

接口的静态特性体现在**接口类型变量具有静态类型**。\

拥有静态类型意味着**编译器会在编译阶段对所有接口类型变量的赋值操作进行类型检查，编译器会检查右值的类型是否实现了该接口方法集合的所有方法**，如果没有实现，则会编译失败。而不是等到运行时才会检查。

```go
var err error = 1 // cannot use 1 (type int) as type error in assignment: int does not implement error (missing Error method)
```

接口的动态体现在接口类型变量在运行时还**存储了右值的真实类型信息，这个右值的真实类型信息被称为接口类型变量的动态类型**。

```go
var err error
err = error.New("error1")
fmt.Printf("%T \n", err)   // *errors.errorString
```

从上面的示例代码中可以看到，err 接口类型变量是 `errors.New` 构造的一个错误值，借助 `fmt.Printf` 函数输出了接口类型变量的动态类型是 `*errors.errorString`。

## 动静兼容的特性有什么好处
接口类型变量在程序运行时，可以被赋值为不同的动态类型变量，每次赋值后，接口类型变量中存储的动态类型信息都会发生变化，这让 Go 语言可以像动态语言（Python）那样拥有鸭子类型（Duck Typing）的灵活性。

> 什么是鸭子类型？
> 
就是指某类型所表现出的特性（比如是否可以作为某接口类型的右值），不是由其基因（比如 C++ 中的父类）决定的，而是由类型所表现出来的行为（比如类型拥有的方法）决定的。

比如下面的例子：
```go
type QuackableAnimal interface {
    Quack()
}
type Duck struct{}
func (Duck) Quack() {
    println("duck quack!")
}
type Dog struct{}
func (Dog) Quack() {
    println("dog quack!")
}
type Bird struct{}
func (Bird) Quack() {
    println("bird quack!")
}                         
                          
func AnimalQuackInForest(a QuackableAnimal) {
    a.Quack()             
}                         
                          
func main() {             
    animals := []QuackableAnimal{new(Duck), new(Dog), new(Bird)}
    for _, animal := range animals {
        AnimalQuackInForest(animal)
    }  
}
```

在这个示例中，使用接口类型 `QuackableAnimal` 来代表具有“会叫”（`Quack()`方法）这一特征的动物，而 Duck、Bird 和 Dog 类型各自都具有这样的特征，


这里的 Duck、Bird、Dod 都是“鸭子类型”，它们之间并没有什么联系，之所以能作为右值赋值给 QuackableAnimal 类型变量，只是因为他们表现出了 QuackableAnimal 所要求的特征罢了，也就是拥有 `Quack()` 方法，而不需要严格的继承体系。

与动态语言不同的是，Go 接口还可以保证“动态特性”使用时的安全性。比如，编译器在编译期就可以捕捉到将 int 类型变量传给 QuackableAnimal 接口类型变量这样的明显错误，决不会让这样的错误遗漏到运行时才被发现。

## 一个问题

接下来通过一个问题来更深入认识一下动静特性。

```go
type MyError struct {
    error
}
var ErrBad = MyError{
    error: errors.New("bad things happened"),
}
func bad() bool {
    return false
}
func returnsError() error {
    var p *MyError = nil
    if bad() {
        p = &ErrBad
    }
    return p
}
func main() {
    err := returnsError()
    if err != nil {
        fmt.Printf("error occur: %+v\n", err)
        return
    }
    fmt.Println("ok")
}
```

在这个示例中，程序的运行逻辑很清晰，调用 returnsError 函数返回指针变量 p，值为 nil，然后比较 err 变量是否等于 nil，最后输出结果。

运行一下示例代码，看看结果是否和预期一致：
```go
error occur: <nil>
```

可以看到，并没有输出预期的 ok，这是怎么回事呢？要搞清楚这个问题，需要进一步了解接口类型变量的内部表示。

### 接口类型变量的内部表示
接口类型“动静兼备”的特性也决定了它的变量的内部表示绝不像一个静态类型变量（如 int、float64）那样简单。

在Go 的源码中可以找到接口类型变量在运行时的表示：
```go
// $GOROOT/src/runtime/runtime2.go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}
type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```

可以看到，在运行时层面，接口类型变量有两种内部表示：`iface` 和 `eface`，这两种表示分别用于不同的接口类型变量：

* eface 用于表示没有方法的空接口（empty interface）类型变量，也就是 `interface{}`类型的变量
* iface 用于表示其余拥有方法的接口 interface 类型变量

它们的共同点是都拥有两个指针字段，并且功能相同，都是指向**当前赋值给该接口类型变量的动态类型变量的值**。

不同点在于，eface 表示空接口类型，并没有方法列表，
因此它的第一个指针字段指向一个 `_type` 类型结构，这个接口为该接口类型变量的动态类型信息，定义是这样的：
```go
// $GOROOT/src/runtime/type.go
type _type struct {
    size       uintptr
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32
    tflag      tflag
    align      uint8
    fieldAlign uint8
    kind       uint8
    // function for comparing objects of this type
    // (ptr to object A, ptr to object B) -> ==?
    equal func(unsafe.Pointer, unsafe.Pointer) bool
    // gcdata stores the GC type data for the garbage collector.
    // If the KindGCProg bit is set in kind, gcdata is a GC program.
    // Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
    gcdata    *byte
    str       nameOff
    ptrToThis typeOff
}
```

而 iface 除了要存储动态类型信息之外，还有存储接口本身的信息（接口的类型信息、方法列表信息等）以及动态类型所实现的方法的信息，因此 iface 的第一个字段指向一个itab类型结构。itab 结构的定义如下：

```go
// $GOROOT/src/runtime/runtime2.go
type itab struct {
    inter *interfacetype
    _type *_type
    hash  uint32 // copy of _type.hash. Used for type switches.
    _     [4]byte
    fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}
```

核心字段如下：
* inter：存储着这个接口类型自身的信息
* _type：存储着这个接口类型变量的动态类型的信息
* func：动态类型已实现的接口方法的调用地址数组

其中 interfacetype 结构的定义如下：
```go
// $GOROOT/src/runtime/type.go
type interfacetype struct {
    typ     _type       // interfacetype 结构由类型信息
    pkgpath name        // 包路径名
    mhdr    []imethod   // 接口方法集合切片（mhdr）
}
```

为了更好地理解 eface 与 iface 在内存的表示，下面分别

## 空接口类型内存中表示

```go
type T struct {
	name string
	age  int
}

func main() {
	var t = T{
		age:  23,
		name: "yumi",
	}

	var ei interface{} = t   // Go运行时使用eface结构表示ei
 
	println("ei = ", ei)      // ei =  (0x1097d60,0xc00000c030)
	fmt.Println("ei = ", ei)  // ei =  {yumi 23}
	fmt.Printf("%T \n", ei)   // main.T 
}
```

该示例代码中的空接口类型变量 ei 在内存中的表示 如下图所示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/空接口类型在内存中的表示.jpg)

可以看到空接口类型的表示较为简单：
* _type 字段指向它的动态类型 T 的类型信息
* data 字段指向一个 T 类型的实例值

### 非空接口类型内存中表示

```go
type T struct {
	name string
	age  int
}

func (t T) M1() {}
func (t T) M2() {}

type NonEmptyInterface interface {
	M1()
	M2()
}

func main() {
	var t = T{
		name: "yumi",
		age:  23,
	}

	var i NonEmptyInterface = t
  
	println("i = ", i)        // i =  (0x10c2248,0xc0000a4018)
	fmt.Println("i = ", i)    // i =  {yumi 23}
	fmt.Printf("%T \n", i)    // main.T
}
```

和 eface 比起来，iface 的表示稍微复杂些，下图是 接口类型变量i 在内存中的表示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/非空接口类型在内存中的表示.jpg)

虽然 eface 和 iface 的第一个字段有所差别，但 tab 和 _type 可以统一看作是动态类型信息。Go 语言中每种类型都会有唯一的 _type 信息，无论是内置原生类型，还是自定义类型都有。Go 运行时会为程序内的全部类型建立只读的共享 _type 信息表，因此**拥有相同动态类型的同类接口类型变量的 _type/tab 信息是相同的**。

接口类型变量的 data 部分则是指向一个动态分配的内存空间，这个内存空间存储的是赋值给接口类型变量的动态类型变量的值。\
未显式初始化的接口类型变量的值为 nil，也就是这个变量的 _type/tab 和 data 都为 nil。

也就是说，判断两个接口类型变量是否相同，**只需要判断 _type/tab 是否相同，以及 data 指针指向的内存空间所存储的数据值是否相同**就可以了。
注意 🚧，这里不是 data 指针的值相同。

## 比较接口变量

### nil 接口变量
未赋初值的接口类型变量的值为 nil，这类变量也就是 nil 接口变量，下面来看一下内存中表示输出的例子：
```go
func printNilInterface() {
	// nil接口变量
	var i interface{} // 空接口类型
	var err error     // 非空接口类型
	println(i)
	println(err)
	println("i = nil:", i == nil)
	println("err = nil:", err == nil)
	println("i = err:", i == err)
}
```

运行上面的示例代码，输出如下：
```go
(0x0,0x0)
(0x0,0x0)
i = nil: true
err = nil: true
i = err: true
```

可以看到，无论是空接口类型变量还是非空接口类型变量，一旦变量值为 nil，那么它们内部表示均为`(0x0,0x0)`，也就是类型信息、数据值信息均为空。因此上面的变量 i 和 err 等值判断为 true。

### 空接口类型变量

下面是空接口类型变量的内部表示输出的例子：
```go
func printEmptyInterface() {
    var eif1 interface{} // 空接口类型
    var eif2 interface{} // 空接口类型
    var n, m int = 17, 18

    eif1 = n
    eif2 = m
    println("eif1:", eif1)
    println("eif2:", eif2)
    println("eif1 = eif2:", eif1 == eif2) // false

    eif2 = 17
    println("eif1:", eif1)
    println("eif2:", eif2)
    println("eif1 = eif2:", eif1 == eif2) // true

    eif2 = int64(17)
    println("eif1:", eif1)
    println("eif2:", eif2)
    println("eif1 = eif2:", eif1 == eif2) // false
}
```

运行上面的示例代码，输出如下：
```go
eif1: (0x10ac580,0xc00007ef48)
eif2: (0x10ac580,0xc00007ef40)
eif1 = eif2: false
eif1: (0x10ac580,0xc00007ef48)
eif2: (0x10ac580,0x10eb3d0)
eif1 = eif2: true
eif1: (0x10ac580,0xc00007ef48)
eif2: (0x10ac640,0x10eb3d8)
eif1 = eif2: false
```

示例代码的逻辑很清晰：
* 第一次打印：动态类型的类型信息是相同的（都是 int），所以 _type 都是0x10ac580，但是 data 指针指向内存中存储的值不一样，因此 eif1 不等于 eif2
* 第二次打印：动态类型的类型信息是相同的，data 指针指向内存中存储的值也相同，因此 eif1 等于 eif2
* 第三次打印：动态类型的类型信息不同（一个是 int，一个是 int64），即便 data 指针指向的内存块中存储值是相同的，最终 eif1 也不等于 eif2

结论：**对于空接口类型变量，只有 _type 和 data 所指数据内容一致的情况下，两个空接口类型变量之间才能划等号**。

### 非空接口类型变量

```go
type T int
func (t T) Error() string { 
    return "bad error"
}
func printNonEmptyInterface() { 
    var err1 error // 非空接口类型
    var err2 error // 非空接口类型
    err1 = (*T)(nil)
    println("err1:", err1)
    println("err1 = nil:", err1 == nil)
    err1 = T(5)
    err2 = T(6)
    println("err1:", err1)
    println("err2:", err2)
    println("err1 = err2:", err1 == err2)
    err2 = fmt.Errorf("%d\n", 5)
    println("err1:", err1)
    println("err2:", err2)
    println("err1 = err2:", err1 == err2)
}
```

运行上面的示例代码，输出如下：
```go
eif1: (0x10ac580,0xc00007ef48)
eif2: (0x10ac580,0xc00007ef40)
eif1 = eif2: false
eif1: (0x10ac580,0xc00007ef48)
eif2: (0x10ac580,0x10eb3d0)
eif1 = eif2: true
eif1: (0x10ac580,0xc00007ef48)
eif2: (0x10ac640,0x10eb3d8)
eif1 = eif2: false
```

看到上面示例中每一轮通过 println 输出的 err1 和 err2 的 tab 和 data 值，要么 data 值不同，要么 tab 与 data 值都不同。

和空接口类型变量一样，只有 tab 和 data 指的数据内容一致的情况下，两个非空接口类型变量之间才能划等号。

这里我们要注意 err1 下面的赋值情况：
```
err1 = (*T)(nil)
```

针对这种赋值，println 输出的 err1 是（0x10ed120, 0x0），也就是非空接口类型变量的类型信息并不为空，数据指针为空，因此它与 nil（0x0,0x0）之间不能划等号。

现在我们再回到我们开头的那个问题，你是不是已经豁然开朗了呢？开头的问题中，从 returnsError 返回的 error 接口类型变量 err 的数据指针虽然为空，但它的类型信息（iface.tab）并不为空，而是 *MyError 对应的类型信息，这样 err 与 nil（0x0,0x0）相比自然不相等，这就是我们开头那个问题的答案解析，现在你明白了吗？

现在再回头看上面那个问题，是不是清晰很多了。\
因为 returnsError 返回的 error 接口类型变量 err 的数据指针虽然为空，但它的类型信息（iface.data) 并不为空，而是 `*MyError` 对应的类型信息，因此 err 与 nil 并不相等。

### 空接口类型变量与非空接口类型变量的等值比较

```go
func printEmptyInterfaceAndNonEmptyInterface() {
  var eif interface{} = T(5)
  var err error = T(5)
  println("eif:", eif)
  println("err:", err)
  println("eif = err:", eif == err)
  err = T(6)
  println("eif:", eif)
  println("err:", err)
  println("eif = err:", eif == err)
}
```

运行上面的示例代码，输出如下：
```go
eif: (0x1093500,0x10c0808)
err: (0x10c0d58,0x10c0808)
eif = err: true
eif: (0x1093500,0x10c0808)
err: (0x10c0d58,0x10c0810)
eif = err: false
```
可以看到，虽然空接口类型变量（eface(_type, data)）和非空接口类型变量（iface(tab, data)）内部表示的结构不一样，但Go 在进行等值比较时，类型比较用的是 eface._type 和 eface.tab._type，因此在这个例子中，eif 和 err 都是T(5) 时，两者是相等的。

## 总结
* 动态特性让 Go 拥有与动态语言相近的灵活性，而静态特性又在编译阶段保证了这种灵活性的安全
* 判断两个接口类型变量是否相同，不仅需要 _type/tab 相同，还需要 data 指针指向的内存空间的值相同

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)